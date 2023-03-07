# 1.软件架构及API汇整

http://www.wowotech.net/pm_subsystem/psy_class_overview.html

所有的class,思路都是:抽象共性、统一接口、屏蔽细节。

power supply class为编写供电设备（power supply，后面简称PSY）的驱动提供了统一的框架，功能包括：

1）抽象PSY设备的共性，向用户空间提供统一的API。

2）为底层PSY驱动的编写，提供简单、统一的方式。同时封装并实现公共逻辑，驱动工程师只需把精力集中在和硬件相关的部分即可。



# 13.Driver的电源管理

[Linux电源管理(13)_Driver的电源管理 (wowotech.net)](http://www.wowotech.net/?post=131)

## WakeUp Count

wakeup count的目的是为了解决同步问题（suspend中 enable_irq与中断处理函数中disbale_irq的执行顺序），也即是为了解决”在睡眠的过程中，接收到事件而想退出睡眠“的问题以及判断“在用户层发起睡眠请求时，当前系统是否符合睡眠条件”

## 唤醒的两种情况：

### 在事件处理完之前主动阻止suspend

有点类似于锁，关于这点的应用就是之后要提到的对内核层的wake lock；可以把wake_lock()当做一种锁来用，其内部使用了__pm_stay_awake()来使系统保持唤醒状态。别忘记，这还是一种开发者主观上的使用，即开发者希望wake_lock() ---> wake_unlock()期间系统不休眠，类似一种特殊的临界区。

```c
int xxx(struct device *dev)  //此函数执行过程中能阻止系统休眠
{
	pm_stay_awake(dev); //event_count++
	....
	pm_relax(dev);//event_count--
	...
}
int xxx_probe(struct platform_device *pdev)
{
	struct device *dev = pdev->dev;
	...
	device_init_wakeup(dev, true); //设置设备唤醒系统的能力
	...
}
//pm_wakeup_pending()主要是判断event_count变量在suspend的过程中是否改变,若有改变，终止suspend
//pm_wakeup_pending（）在函数 s2idle_loop中被一个死循环反复调用
```

### 当系统suspend完成后,使用中断唤醒系统

Suspend的最终形态是WFI，wait for interrupt。要具备硬件唤醒功能，得是一个能到达CPU Core的中断才行，而这中间经过各级中断管理器，并不说只要是一个中断就能唤醒系统.

#### IRQF_NO_SUSPEND

在suspend过程中dpm_suspend_noirq()->suspend_devices_irq()时保留IRQF_NO_SUSPEND类型的中断响应，IRQF_NO_SUSPEND 仅仅是STR 的时候，不关闭中断。这并不代表具有唤醒系统的能力。

#### irq_enable_wake(irq); 

后者直接跟irq_chip打交道，把唤醒功能的设置交由irq_chip driver处理

在Atmel的BSP中，这一个过程只是对一个名为wakeups的变量做了位标记，以便其随后在plat_form_ops->enter()时重新使能该中断。

# 101.系统休眠（System Suspend）和设备中断处理

[系统休眠（System Suspend）和设备中断处理 (wowotech.net)](http://www.wowotech.net/pm_subsystem/suspend-irq.html)

## 设备IRQ的suspend和resume

在系统suspend过程的后期，各个设备的IRQ (interrupt request line)会被disable掉。具体的时间点是在各个设备的late suspend阶段之后。

```c
static int suspend_enter(suspend_state_t state, bool *wakeup)
{	……

    error = dpm_suspend_late(PMSG_SUSPEND);//－－－－－late suspend阶段

    error = platform_suspend_prepare_late(state);

    下面的代码中会disable各个设备的irq

    error = dpm_suspend_noirq(PMSG_SUSPEND);//－－－－进入noirq的阶段
 //

    error = platform_suspend_prepare_noirq(state);

    ……

}
```

在dpm_suspend_noirq函数中

1.会调用suspend_device_irqs函数来disable所有设备的irq  

2.会针对系统中的每一个device，依次调用device_suspend_noirq来执行该设备noirq情况下的suspend callback函数.

系统resume过程中，在各个设备的early resume过程之前，各个设备的IRQ会被重新打开，具体代码如下:

```c
static int suspend_enter(suspend_state_t state, bool *wakeup)

{	……

    platform_resume_noirq(state);//－－－－首先执行noirq阶段的resume

    dpm_resume_noirq(PMSG_RESUME);//－－－－－－在这里会恢复irq，然后进入early resume阶段

    platform_resume_early(state);

    dpm_resume_early(PMSG_RESUME);

	……
}
```

## 关于IRQF_NO_SUSPEND Flag

IRQF_NO_SUSPEND 置位后 suspend_device_irqs并不会disable该IRQ，从而让该中断在随后的suspend和resume过程中，保持中断开启。当然，这并不能保证该中断可以将系统从suspend中唤醒。如果想要达到唤醒的目的，请调用enable_irq_wake。

需要注意的是：IRQF_NO_SUSPEND flag影响使用该IRQ的所有外设（一个IRQ可以被多个外设共享，不过ARM中不会这么用）。正因为如此，我们应该尽可能的避免同时使用IRQF_NO_SUSPEND 和IRQF_SHARED这两个flag。

## 系统中断唤醒接口：enable_irq_wake() disable_irq_wake()

enable_irq_wake()函数用来打开该外设中断线通往系统电源管理模块（也就是上面的suspend-resume模块）之路.

![irq-suspend](http://www.wowotech.net/content/uploadfile/201704/d77ad10fce03b9d181a2be9351c6113420170421040223.gif)

调用了enable_irq_wake会影响系统suspend过程中的suspend_device_irqs处理

```c
static bool suspend_device_irq(struct irq_desc *desc)

{
    if (irqd_is_wakeup_set(&desc->irq_data)) 
    {
        irqd_set(&desc->irq_data, IRQD_WAKEUP_ARMED); //desc->irq_data在                enable_irq_wake中被设置，在此情况下中断不会被disable，只是被标记一个IRQD_WAKEUP_ARMED的标记
        return 0;
    }
    省略Disable 中断的代码
}
```

在系统suspend的过程中，唯一有机会执行的interrupt handler是那些标记IRQF_NO_SUSPEND flag的IRQ，

而使用了enable_irq_wake的irq会走系统唤醒路径

## Interrupts and Suspend-to-Idle

```c
static int suspend_enter(suspend_state_t state, bool *wakeup)

{

    ……

    各个设备的late suspend阶段

    各个设备的noirq suspend阶段

    if (state == PM_SUSPEND_FREEZE)
    {

        freeze_enter();

        goto Platform_wake;

    }

    ……

}
```

不一样的地方在noirq suspend完成之后，freeze不会disable那些non-BSP的处理器和syscore suspend阶段，而是调用freeze_enter函数，把所有的处理器推送到idle状态。这时候，任何的enable的中断都可以将系统唤醒,包括IRQF_NO_SUSPEND置位的irq。

wakeup interrupt由于其中断没有被mask掉，因此也可以将系统从suspend-to-idle状态中唤醒。整个过程和将系统从suspend状态中唤醒一样，唯一不同的是：将系统从freeze状态唤醒走的中断处理路径，而将系统从suspend状态唤醒走的唤醒处理路径，需要电源管理HW BLOCK中特别的中断处理逻辑的参与。

# IRQF_NO_SUSPEND 标志和enable_irq_wake函数不能同时使用

1.如果IRQ没有共享，使用IRQF_NO_SUSPEND flag说明你想要在整个系统的suspend-resume过程中（包括suspend_device_irqs之后的阶段）保持中断打开以便正常的调用其interrupt handler。而调用enable_irq_wake函数则说明你想要将该设备的irq信号设定为中断源，因此并不期望调用其interrupt handler。

2.IRQF_NO_SUSPEND 标志和enable_irq_wake函数都不是针对一个interrupt handler的，而是针对该IRQ上的所有注册的handler的。在一个IRQ上共享唤醒源以及no suspend中断源是比较荒谬的。