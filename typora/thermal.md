# Design Overview

### Software Architecture

**Thermal zone device** (**TZ**): report  current temperature

**Cooling device** (**TC**): turn on / off a specific temperature in milli-degree C.

**Linux Thermal Framework（LTF**）: monitor each thermal zone periodically and bases on trips points to activate coolers. 是zone 和 cooler的抽象

**mtk_thermal_monitor**: 基本功能：log the status of each thermal zone devices and colling devices periodically.扩展功能：(1)  SMA：滤除噪声   (2) cooler exit point ,冷却器会在达到冷却退出点之后再退出

（3).条件冷却

**trip**: temperature threshold, 每个thermal zone最多有10个trip，最少一个,每个trip只能绑定一个TC

**thermald** : a small native daemon to handle signal sent form **mtk_cooler_shutdown.c** 

if receives signal ,it starts  **ShutdownAlerTDialogActivity** via **ActivityAmanager**.

**ShutdownAlerTDialogActivity** : broadcasts **Intent** with 	**ACTION_REQUEST_SHUTDOWN**  and pops up Alert Dialog  that phone is hot

**ACTION_REQUEST_SHUTDOWN**  :by broadcasts  **ACTION_REQUEST_SHUTDOWN**  Intent, **Thermal management**  can power off the whole system .

**Thermal management**: communicates thermal kernel modules via **sysfs** and **procfs**.

its main function is parse a thermal policy of the .conf format  .configures thermal zone device drivers and cooling device drivers

thermal zone device drivers and cooling device drivers register not in LTF directly but in **mtk_thermal_monitor** to utilize the logging and extended services.

![image-20221029162149230](F:\typora\image\image-20221029162149230.png)

### Linux thermal framework and mtk Extension   

Linux thermal framework source paths:

![image-20221029162611459](F:\typora\image\image-20221029162611459.png)

mtk_thermal_monitor AOSP source paths:

![image-20221029162735630](F:\typora\image\image-20221029162735630.png)

#### the main functions of mtk_thermal_monitor

**simple moving average**  **(SMA)**

is used to filter errors of temperature values of a thermal zone.   it works as a simple low pass filter. 

**cooler exit point** 

![image-20221029172220652](F:\typora\image\image-20221029172220652.png)

keep a cooler active for a period of time to make sure a certain amount of temperature 

drop.

**conditional cooling**

adds more criteria on the decision to activate a cooling device.,thus a cooling device  can be activated  in  more precise way to avoid over-killing system performance.

 **log the status**

 log the status of each thermal zone devices and colling devices periodically.



#### Api functions for a thermal zone device driver

**mtk_thermal_zone_device_register_wrapper**

to replace LTF's thermal_zone_device_register

**mtk_thermal_zone_device_unregister_wrapper**

**mtk_thermal_zone_bind_cooling_device_wrapper**

to replace LTF's thermal_zone_bind_cooling_device

**mtk_thermal_zone_bind_trigger_trip**

if thermal zone driver controls a hw thermal sensor with interrupt on threshold capability. the thermal zone implementation can invoke this function in interrupt callback functions to save the overhead of polling ADC channel every time.

**mtk_thermal_get_temp**

get the temperature  of all thermal zones registered

# Thermal zones

![image-20221029182430377](F:\typora\image\image-20221029182430377.png)

### thermal zone - mtktscpu

![image-20221029182524584](F:\typora\image\image-20221029182524584.png)

monitors IC temperature. 

a pseudo thermal zone. and it takes the maximal value of TS1,TS2,...,TS5.



### Thermal zone - mtkts1,...5

![image-20221029183124637](F:\typora\image\image-20221029183124637.png)

these five thermal sensors are only for test,the default thermal policy is not periodically poll their temperatures,if needed,set corresponding polling delay to 1000.



### thermal zone - mtktspmic 

![image-20221029183516311](F:\typora\image\image-20221029183516311.png)

monitor  pmic  temperature

### thermal zone - mt6357tsbuck1 and mt6357tsbuck2

![image-20221029185559326](F:\typora\image\image-20221029185559326.png)

 monitor temperature external buck1 and buck2 in MT6358 pmic respectively

### thermal zone - mtkcharger1 and mtkcharger2

![image-20221029185731765](F:\typora\image\image-20221029185731765.png)

monitor the RT9465 (slave charger) and MT6371(main charger) respectively.

### thermal zone - mtktsbattery

![image-20221029190030878](F:\typora\image\image-20221029190030878.png)

monitors temperature near battery connector if the battery carries an NTC, the values are obtained form battery/ charger IC drivers. the data only updated every 10 seconds.

### thermal zone- mtkspa

![image-20221107105607527](F:\typora\image\image-20221107105607527.png)

monitors MT6177 temperature die temperature and nearby PCB temperatures. MT6177  is placed near FDD(频分双工)/TDD（时分双工） pa,this thermal zone only reports real temperature when there is traffic through WCDMA      [CDMA](https://product.pconline.com.cn/itbk/sjtx/sj/1107/2475393.html) (Code Division Multiple Access) 又称码分多址 ，or TD-SCDMA(是英文Time Division-Synchronous Code Division Multiple Access（时分同步码分多址）的简称） modem ，when modem idle，mtkspa reports -127℃。 RF temperatures are queried by thermal daemon every 30s then set to mtk_ts_pa_thput.c.mtk_ts_pa.c wraps(封装) temperature  information into a thermal zone and read temperature form  mtk_ts_pa_thput.c

###  thermal zone- mtktswmt

![image-20221107112124877](F:\typora\image\image-20221107112124877.png)

mtktswmt reports MT6631  combo IC.report real temperature since WIFI . BT/GPS/FM activities will not make mtkswmt report temperature  since wifi is the major heat source. the invalid values of mtktswmt include -64℃ and -127℃.for power saving ,if there is no traffic on wifi,the sensor will update temperature every 300 seconds, if the traffic is reach teh threshold,tempature will update every 3 seconds .

### thermal zone- mtktsAP ,mtktsmdpa.

![image-20221107114108698](F:\typora\image\image-20221107114108698.png)

this called BTS,short for board thermal sensor .In MT676X reference design,there are two BTS,one beside MT676X (mtktsAP),and the  other beside modem power amplifiers (mtktsbtsmdpa).mtktsAP is connected to AUX ADC channel 0 of MT676X. mtktsmdpa  is  connected to AUX ADC channel 1 of MT676x. it is a NTC nearby AP MT6757. with the NTC,we can know if the heat  spread to PCB area or Tskin form AP.

# Coolers

### summary

MT676x provides the many kinds of cooling devices,still only several are used in default thermal policies. See below table. Others provide the capability to customers for optimizing（优化） thermal performance for different MT676x projects. Multiple coolers of the same functions are provided since each cooler instance can only be bound(绑定) to one and only one trip of a thermal zone.

![image-20221107144110017](F:\typora\image\image-20221107144110017.png)

### MT676x Static AP OPP Limit

MT676x thermal management provides 34 static AP coolers named from cpu00,cpu01,...cpu33. The numbering is only for a possible future extension. AP cooling device mapping table is shown as below.

![image-20221107152103471](F:\typora\image\image-20221107152103471.png)



Form MT676x,static AP  coolers are only used for hard limit and mainly for protection. They are not used in temperature control and thermal throttling anymore.

![image-20221107152909400](F:\typora\image\image-20221107152909400.png)

when any static CPU actually taking effects,"set_static_cpu_power_limit"string can be 

found in kernel log.



### MT676x Adaptive AP Coolers

it just provides a sorted index for all available MT676x CPU OPPs for thermal management to use in communication with CPU DVFS.

we don't need to choose proper CPU OPPs for cooling.ATM just chooses
a CPU OPP according to CPU Tj continuously,check how it works in the next iteration,then decides if higher or lower CPU OPP fits better,until CPU Tj reach stable at the target temperature.

The parameters of ATM can only reflect on how many iteration(迭代) it needs to converge(收敛)to a proper OPP. With bad parameters,CPU Tj may fluctuate hugely

hey can only be bound to thermal zone of mtktscpu since they are designed to control AP temperature.Binding one CPU adaptive cooler to a trip then it will control CPU Tj around the temperature of target Tj.Target Tj is a range between Target Tjo and Target Tj2,and the actual value can be derived from Tpcb.

### Battery Charging Current Throttling

![image-20221107161307964](F:\typora\image\image-20221107161307964.png)

![image-20221107161317501](F:\typora\image\image-20221107161317501.png)

### Camera exit

To control high power camera scenarios,such as Sw high resolution video playback,high resolution video recoding,and video shadow depth of field,MT676x provide four camera exit coolers:

mtk-cl-cam00
mtk-cl-cam01
mtk-cl-cam02
mtk-cl-cam-urgent

![image-20221107164559334](F:\typora\image\image-20221107164559334.png)

MT676x provides two types of camera exit coolers:
Type 1:with a notification dialog.
Type 2:Without a notification dialog
mtk-cl-cam00,mtk-cl-cam01,and mtk-cl-cam02 are belonging to type 1
mtk-cl-cam-urgent is belonging to type 2

### LTE Modem Uplink Data Rate Throttling

To control thermal problems from LTE modem and corresponding power amplifiers,LTE modem uplink throughput throttling is added.There six coolers designed for this:
mtk-cl-mutto0
mtk-cl-mutto1
mtk-cl-mutt02
mtk-cl-mutt03
mtk-cl-noIMS
mtk-cl-mdoff
mtk-cl-adp-mutt

![image-20221107165623584](F:\typora\image\image-20221107165623584.png)

### System Shut Down

Mtk-cl-shutdown00
Mtk-cl-shutdown01
Mtk-cl-shutdown02

![image-20221107171521781](F:\typora\image\image-20221107171521781.png)

mtk-cl-kshutdowno0
mtk-cl-kshutdown01
mtk-cl-kshutdown02
mtk-cl-kshutdown03
mtk-cl-kshutdown04

![image-20221107171615059](F:\typora\image\image-20221107171615059.png)

### Backlight Limit

MT676x backlight cooling devices:
Mtk-cl-backlighto1
Mtk-cl-backlight02
Mtk-cl-backlight03

![image-20221107172858171](F:\typora\image\image-20221107172858171.png)

MT676x provides three backlight cooling devices.when none of three are activated, thermal management does not limit backlight LED .每一个设备打开，亮度下降30%.

If thermal managemrnt limits brightness to below 70% while current brightness only set to 40%,nothing will happen.

Brightness changes can be easily observed by users,so mtk  does not adopt these coolers in any thermal policy by default.

## Thermal Configurations 

thermal policy defines the following:

1. how thermal SW monitors each thermal zone

   a. polling delay of each thermal zone

   polling delay  can be set below 200ms on MT676X in the thermal zones mtktscpu 

   for more accurate temperature sensing. Definition of this value should  take the temperature curve of a thermal zone into consideration.

    b. how many points to calculated SMA value 

   The more SMA points provides better noise filtering but delays thermal  zone responsiveness more.

2. how many trips on a thermal zone

   ​	a. limitation: only allow max 10 trips for a single thermal zone.

3. which cooler is bound to a thermal trip

   ​	a. trip to cooler is one to one mapping. each cooler can be bound to only one trip. each trip can be bound to only one cooler.

4. cooler exit point

5. condition cooling

MT676x provides three default thermal policies 

1. DP default thermal policy thermal.conf  
2. TP  thermal protection      thermal.off.conf
3. HT120 high temperature 120℃   ht120.mtc

### 

​	

































































































































































































































































































































































