## Linux内核的整体架构

#### 1. 前言

a) 内核版本为Linux 3.10.29
b) 鉴于嵌入式系统大多使用ARM处理器，因此涉及到体系结构部分的内容，都以ARM为分析对象

#### 2. Linux内核的核心功能

如下图所示，Linux内核只是Linux操作系统一部分。对下，它管理系统的所有硬件设备；对上，它通过系统调用，向Library Routine（例如C库）或者其它应用程序提供接口。

![image](http://www.wowotech.net/content/uploadfile/201402/942a949e1c39a71d129c5836a5a0ccbf20140221042332.png)



因此，其核心功能就是：**管理硬件设备，供应用程序使用**。而现代计算机（无论是PC还是嵌入式系统）的标准组成，就是CPU、Memory（内存和外存）、输入输出设备、网络设备和其它的外围设备。所以为了管理这些设备，Linux内核提出了如下的架构。



#### 3. Linux内核的整体架构

##### 3.1 整体架构和子系统划分

![image](http://www.wowotech.net/content/uploadfile/201402/fc14b80aa29aafc174432f4167535e9e20140221043037.png)

上图说明了Linux内核的整体架构。根据内核的核心功能，Linux内核提出了5个子系统，分别负责如下的功能：

1. Process Scheduler，也称作进程管理、进程调度。负责管理CPU资源，以便让各个进程可以以尽量公平的方式访问CPU。

2. Memory Manager，内存管理。负责管理Memory（内存）资源，以便让各个进程可以安全地共享机器的内存资源。另外，内存管理会提供虚拟内存的机制，该机制可以让进程使用多于系统可用Memory的内存，不用的内存会通过文件系统保存在外部非易失存储器中，需要使用的时候，再取回到内存中。

3. VFS（Virtual File System），虚拟文件系统。Linux内核将不同功能的外部设备，例如Disk设备（硬盘、磁盘、NAND Flash、Nor Flash等）、输入输出设备、显示设备等等，抽象为可以通过统一的文件操作接口（open、close、read、write等）来访问。这就是Linux系统“一切皆是文件”的体现（其实Linux做的并不彻底，因为CPU、内存、网络等还不是文件，如果真的需要一切皆是文件，还得看贝尔实验室正在开发的"[Plan 9](http://plan9.bell-labs.com/plan9/)”的）。

4. Network，网络子系统。负责管理系统的网络设备，并实现多种多样的网络标准。

5. IPC（Inter-Process Communication），进程间通信。IPC不管理任何的硬件，它主要负责Linux系统中进程之间的通信。

   

##### 3.2 进程调度（Process Scheduler)

