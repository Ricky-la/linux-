# Exceptional Control Flow 

The sequence of instructions is called the control flow

## Altering the Control Flow:

program state: 

​	Jumps and branches

​	Call and return

system state  :

​	Data arrives from disk or a network adapter

​	Instruction divides by zero 

​	User hits Ctrl-C at the keyboard

​	System timer expires and so on     

## Exists at all levels of a computer system 

Low level mechanisms

​	Exceptions

Higher level mechanisms

​	Process context switch 

​	Signals 

​	Nonlocal jumps



# Exceptions

## Exception Tables

![image-20220831095429631](F:\typora\image\image-20220831095429631.png)

##   Asynchronous Exceptions(Interrupts)

 	Caused by events external to the processor ,by setting the interrupt pin to notify

CPU.

### 		Examples:

​					Timer interrupt 

​							Every few ms,an external timer chip triggers an interrupt

​							Used by the kernel to take back control from user programs

#### 					  I/O interrupt from external 

​							Hitting Ctrl-C at the keyboard 

​							Arrival of a packet from a network

​							Arrival of data from a disk

##  Synchronous Exceptions

###  	Traps 

​					Intentional 

​					Examples: system calls , breakpoint traps ,special instructions

​					Returns control to  "next" instruction  

#### 			   				System calls example

​								Open file by syscall #2

​                                ![image-20220831100425254](F:\typora\image\image-20220831100425254.png)				  

###       Faults 

​					Unintentional but possibly recoverable

​					Examples: pages faults(recoverable),protection faults(unrecoverable), floating point .

​					Either re-executes faulting ("current") instruction or aborts

#### 				Fault  Example: Page Fault

```c
        int a[1000];
        main()
     	{
     		a[500]=13;
     	}
        80483b7 : c7 05 10 9d 04 08 0d   mov1   $0xd,0x8049d10
        
        // copy page from disk to memory ,return and reexecute movl 
```

#### 				Fault Example : Invalid Memory Reference

```c
        int a[100];
        main()
     	{
     		a[5000]=13;
     	}
        80483b7 : c7 05 10 9d 04 08 0d   mov1   $0xd,0x804e360
        //send SIGSEDV signal to user process,user process exits with "segmentation fault"
```

​                  ![image-20220831101743593](F:\typora\image\image-20220831101743593.png)

###  	Aborts

​					Unintentional and unrecoverable

​					Examples: illegal instruction,parity error ,machine check

​					Aborts current program

# Process  Switch

​	A process is an instance of a running program

## 	Two key abstractions:

###  		Logical control flow :

​						seems to have exclusive use of the CPU and registers

​						provide by kernel mechanism called context switch

### 	 	Private address space	

​						seems to have exclusive use of the memory 

​						provide by kernel mechanism called virtual memory

## 	Concurrent 

​				Two process run concurrently if their overlap in time

## 	Context Switching

​				The kernel is not a separate process, but rather runs as part of some existing process. Control 				flow passes from one process to another via a context switch.

##     System Call Error Handling

​           	On error, Linux system-level functions typically return -1 and set global variable **errno ** to 			   indicate cause ,must check status of system - level function 

​               Only exception is the handful of functions that return void (example **exit**, **free**)

### 		example:

```c
 	if((pid==fork())<0)
    {
        fprintf(stderr,"fork error: %s\n",strerror(errno));
        exit(0);
    }
```

## 	Error- handling Wrappers

```c
	pid_t Fork(void)
    {
        pid_t pid;
        if((pid=fork())<0)
        unix_error("Fork error");
        return pid;  
    }

    void unix_error(char *msg)
    {
        fprintf(stderr,"%s: %s\n",msg,strerror(errno));
        exit(0);
    }
```

### 	Obtaining Process IDs

```c
    pid_t getpid(void);    //returns PID of current proccess
    pid_t getppid(void);   //returns PID of parent process
```

# Creating and Terminating   	Process

#### 			Process status:

​				  **Running** :  execute or wait to be execute

​			 	 **Stopped**:   suspended

​				  **Terminated**: stopped permanently

#### 			Terminating Process

​		Process becomes terminated for one of three reasons:

​								Receiving a signal whose default action is to terminate

​							    Returning from the main routine

​								Calling the exit function

#### 			Creating Process

```c
        int fork(void);
            //returns both child process and parent process
            //return 0 in child process
            //return child pid in parent process
```

###    	Process Graph

   ```c
        int main()
        {
            pid_t pid;
            int x=1;
            pid=Fork();
            if(pid==0)
                printf("child :x=%d\n",++x);
            printf("parent: x=%d\n",--x);
            exit(0);
        }
   ```

![image-20220831170130864](F:\typora\image\image-20220831170130864.png)



### 	Reaping Child Processes

####        	 wait function and waitpid function

```c
        int wait(int *child_status);
                //suspends current process until one of its children terminates
                //return vaule is the child process that terminated
                //if child_status !=null,then the integer it points to will be set to a 
                //value that indicates reason the child terminated and the exit status.
        int waitpid(pid_t pid, int &status,int options);
                //suspend current process until specific process terminates
                //various options
```

 					if do not reap the child process,if parent process terminated firstly ,the child process will         					 become a orphan process,finally will be adopted by init process (pid=1). if child process      					terminated  firstly , it will be a zombie process .

# Execve: Loading and Running Programs

```c
			int execve(char *filename,char *argv[],char *envp[]);
            //Eecutable file filename,can be object file or script file beginning with             //#!interpreter (example #!/bin/bash)
			//With argument list argv  dafault:argv[0]=filename(if it inherits by main             //function ,aslo you can change it)
			//With environment varible list envp  "name-value"strings (e.g.,USER=droh)			
            //Overwrite code,data and stack
			//Called once and never returns except if there is an error

			#include <stdio.h>
            #include<stdlib.h>
            #include <sys/types.h>
            #include <unistd.h>
            #include <sys/wait.h>
            int main(int argc,char **args) {
                    int pid;
                    printf("%s",args[0]);
                    char *argv[]={"ls","/", NULL};
                    char *envp[]={0,NULL}; 
                    if((pid=fork())==0)
                    {
                            execve("/bin/ls",argv,envp);
                            exit(1);
                    }
                    wait(NULL);
                    printf("parent over\n");
                    return 0;  
            }
```

# Signal 

##  		Shell

​						A shell is an application program that runs programs on behalf of the user.

```c
            int main()
            {
                char cmdline[MAXLINE];
                while(1)
                {
                    /*read*/
                    printf("> ");
                    Fgets(cmd,MAXSIZE,STDIN);
                    if(feof(stdin))
                        exit(0);
                    /*evaluate*/
                    eval(cmdline);
                }
            }

            void eval(char *cmdline)
            {
                char *argv[MAXSIZE];  /*Argument list execve()*/
                char buf[MAXSIZE];    /*Holds modified command line*/
                int bg;               /*Should the job run in bg or fg*/
                pid_t pid;            /*Process id*/

                strcpy(buf,cmdline);
                if(argv[0]==NULL)
                    return;           /*Ignore empty */

                if(!builtin_command(argv))
                {
                    if((pid=Fork())==0)   /*Child runs user job*/
                    {
                        if(execve(argv[0],argv,environ)<0)
                        {
                            printf("%s":Command not found.\n,argv[0]);
                            exit(0);
                        }
                    }
                    /*Parent waits for foreground job to terminate*/
                    if(!bg)
                    {
                        int status;
                        if(waitpid(pid,&status,0)<0)
                            unix_error("waitfg:waitpid error");
                    }
                    else
                        printf(“%d %s”,pid,cmdline);
                }
                return;
            }
```

​							**problems**: bg process will not be executed wait function,memory  leak 

​							**Solution** : Exceptional control flow

​											The kernel will interrupt regular processing to alert us when a background process completes by signal

## 		Signal

​					A signal is a small message that notifies a process that an event of some type has occurred in the system.	

### 			Sending a Signal

​						kernel sends a signal to a destination process by updating some state in the context of the destination process. 

```c
              /*Send by kill file*/
			  /bin/kill -9 5    //SIGINT to pid =5
			  /bin/kill -9 -5   //SIGINT to evrey member in group that 										    //have a pid =5 process  
```

```c

          /*send signals from the keyboard*/
          ctrl-c //terminate: send a SIGINT  to every job in the foreground
          ctrl-z //suspend: send a SIGTSTP to every job in the foreground 

```

```c
			/*send signals with kill Function*/
            void example()
            {
                pid_t pid[N];
                int i;
                int child_status;

                for(int i=0;i<N;i++)
                    if((pid[i]=fork())==0)
                    {
                        /*Child : Infinite Loop*/
                        while(1);
                    }
                for(int i=0;i<N;i++)
                {
                    printf("Killing process %d\n",pid[i]);
                    kill(pid[i],SIGINT);
                }
                for(int i=0;i<N;i++)
                {
                    pid_t wpid=wait(&child_status);
                    if(WIFEXITED(child_status))
                        printf("Child %d terminated with exit status %d\n",
                                    wpid,WEXITSTATUS(child_status));
                    else
                        printf("Child %d terminated abnormally\n",wpid);
                }
            }
```



### 			Receiving a Signal

​						A destination process receives a signal when it is forced by the kernel to react in 						some way to the delivery of the signal

​						When encounter process context switch,the signals will be receipt,the                     						destination process set its pnb vector =pending & ~blocking

####  				 		Some possible ways to react:

​							Ignore

​							Terminate the process

​							catch the signal by executing a user-level function called signal handler

#### 		Default Actions 

​					Each signal type has a predefined default action, which is one of:

​							The process terminates

​							The process stops until restarted by a SIGCONT signal.

​							The process ignores the signal.

#### 		Installing Signal Handlers

​		       The signal function modifies the default action associated with the receipt of signal   				signum:  

```c
handler_t *signal(int signum,handler_t *handler);
//different values for argument handler 
// SIG_IGN :ignore signals of type signum
//SIG_DFL :revert to the default action
//Otherwise: handler is the address of a user-level signal handler 
```

#####               Signal  Handling Example :

```c
void sigint_handler(int sig) /*	SIGINT handler*/
{
    printf("So you think you can stop the bomb with ctrl-c,do you?\n");
    exit(0);
}
int main()
{
    /*Install the Sigint handle*/
    if(signal(SIGINT,sigint_handler)==SIG_ERR)
        unix_error("signal error");
    /*Wait for the receipt of a signal*/
    pause();
    return 0;
}
```





​				

### 			Pending and Blocked signals

​							A signal is pending if sent but not yet received

​										There can be at most one pending signal of any particular type

​										 Important : signal are not queued , if a signal is pending,the some type

​										will be discarded.

​							A process can block the receipt of certain signals.	

​										The signal can be sent but can not to be receipt

### 			Pending and Blocked Bits

​							Kernel maintains pending and blocked bit vectors in the context of each process

​								pending: 

​										kernel sets bit k in pending when a signal of type k is delivered.

​										Kernel clears bit k in pending when a signal of type k is received.

​								 blocked: 

​										can be set and cleared by using the sigprocmask function 

```c
//sigemptyset function - Create empty set
//sigfillset function -Add every signal number to set
//sigaddset function -Add signal number to set
//sigdelset function -Delete signal number from set

sigset_t mask ,perv_mask;
sigempytset(&mask);
/*Block SIGINT and save previous blocked set*/
/* 
	Code region that will not be interrput by Sigint
*/
/*Restore previous blocked set,unblocking SIGINT*/
sigprocmask(SIG_SETMASK,&prev_mask,NULL);


```



### 			Process Groups 

​									every process belongs exactly one process group ,signal can be sent to a 									process group.   

#### 				Process Groups  Functions:

```c
				getpgrp() // returns process group of current process
                setpgid() //change process group of a process
```

### Guidelines for Writing Safe Handlers

​		**Keep handlers as simple as possible**

​		**Call only async-signal-safe functions in handlers**

​			printf,sprintf,malloc and exit are not safe 

​		**Save and restore errno on entry and exit**

​		**Protect accesses to shared data structures by temporarily blocking all signals**

```c
        sigprocmask(SIG_BLOCK,&mask_all,&prev_all);
        addjob(pid);
        sigprocmask(SIG_SETMASK,&prev_all,NULL);
```

​		**Declare global variables as volatile**

​		**Declare global flags as volatile sig_atmoic_t** 

### Synchronizing Flows to Avoid Races

​		A subtle synchronization error because it assumes parent runs before child

```c
//Problem: if a child process executed early than added job queue,and when the //handler executed,but the job is not in job queue but will be deleted,if the job
//is added in the future ,if will not be deleted.
int main(int argc,int **agrv)
{
    int pid;
    sigset_t mask_all,prev_all;
   	signfillset(&maksall);
    siganl(SIGCHLD,handle);
    initjobs(); /*Initialize the job list*/ 
    while(1)
    {
        if((pid=Fork())==0)/*child*/
        { 
            Execve("/bin/data",argv,NULL);
        }  
    sigprocmask(SIG_BLOCK,&mask_all,&prev_all);  /*parent*/
    addjob(pid);
    sigprocmask(SIG_SETMASK,&prev_all,NULL);
    }
    exit(0);
}
void handler(int sig)
{
    int olderrno=errno;
    sigset_t mask_all,prev_all;
    pid_t pid;
    
    sigfillset();
    while((pid=waitpid(-1,NULL,0))>0)   /*reap child*/
    {
        sigprocmask(SIG_BLOCK,&mask_all,&prev_all);
        deletejob(pid);  /*Delete teh child from he job list*/
        sigprocmask(SIG_SETMASK,&PREV_all,NULL);
    }
    if(errno!=ECHILD)
        Sio_error("waitpid error");
    errno=olderrno;
}
```

```c
//Corrented Program
int main(int argc,char **argv)
{ 
    int pid;
    sigset_t mask_all,prev_one,mask_one;
   	signfillset(&maksall);
    sigemptyset(&mask_one);
    sigaddset(&mask_one,SIGCHLD);
    siganl(SIGCHLD,handle);
    initjobs(); /*Initialize the job list*/ 
    while(1)
    {
        sigprocmask(SIG_BLOCK,&mask_one,&prev_one); /*block before create child*/
        if((pid=Fork())==0)/*child*/
        { 
            sigprocmask(SIG_SETMASK,&prev_one,NULL);
            Execve("/bin/data",argv,NULL);
        }  
    sigprocmask(SIG_BLOCK,&mask_all,&prev_all);  /*parent*/
    addjob(pid);
    sigprocmask(SIG_SETMASK,&prev_all,NULL);
    }
    exit(0);   
}
```

### Waiting function  in signal_handle

   ```c
volatile sig_atomic_t pid;
int main(int argc,char **argv)
{ 
    sigset_t mask,prev;
    sigemptyset(&mask);
    sigaddset(&maks,SIGCHLD);
    signal(SIGCHLD,sigchld_handle);
    signal(SIGCHLD,sigint_handle);
    while(1)
    {
        sigprocmask(SIG_BLOCK,&mask_one,&prev_one); /*block before create child*/
        if(Fork())==0)/*child*/
        { 
			exit(0);
        }  
        pid=0; /*parent*/
        sigprocmask(SIG_SETMASK,&mask,&prev);  
        /*method 1*/  
        while(!pid)   //correct but wasteful 
            ;
        
        /*method 2*/
        while(!pid)  /*Race: After checkout the pid and before,a SIGCHLD signal*/ 
            pause();/* arrived,the pause will never be stopped*/ 
       
        /*method 3*/
        while(!pid)  /*correct but too slow*/
            sleep(1);
        
        /*method 4*/
        while(!pid)  
            sigsuspend(&prev); //unblock SIGCHLD and atomically pause
        
        
        /*Do some work after receiving SIGCHLD*/

    }
    exit(0);   
}
void sigchld_handle(int s)
{
    int olderrno=errno;
    pid=waitpid(-1,NULL,0); /*main is waiting for nonzero pid*/
    errno=olderrno;
}
void sigint_handle(int s)
{   
}
   ```

#### Solution: Waiting for signals with sigsuspend

```c
int sigsuspend(const sigset_t *mask)
/*Equivalent to atomic version to method 2*/
   	sigprocmask(SIG_BLOCK,&mask,&perv);
	pause();
	sigprocmask(SIG_SETMASK.&prev,NULL);
```



​								

​							

