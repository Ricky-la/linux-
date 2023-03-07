# Network Program

##  	A Client-Server Transaction

​				Most network applications are based on the client-server model:            ![](F:\typora\image\image-20220914094208092.png)

##   Hardware Organization of a Network  Host 

​    	Network looks to your computer like an IO device,and in fact the UNIX api for dealing with networks 		,Makes it looks like a file.

##   Computer Networks

​		A network is a hierarchical system of boxes and wires organized by geographical proximity

​				SAN (system area network) spans cluster or machine room 

​				LAN (local area network) spans a building or campus 

​				WAN (wide area network) spans country or world

### 		Lowest Level: Ethernet segment

​				![image-20220914105206593](F:\typora\image\image-20220914105206593.png)



​				Ethernet segment consists of a collection of hosts connected by wires to a hub

​				Each Ethernet adapter has a unique 48 bits address (MAC address) 

​				Hosts send bits to any other host in chunks called frames

​				Sharing a single communication channel,every hosts can see every bit

### 		Next level : Bridged Ethernet segment

​	     		![image-20220914105358551](F:\typora\image\image-20220914105358551.png)				

### 		Next level: Internet 

![image-20220914105732479](F:\typora\image\image-20220914105732479.png)

​                   Multiple incompatible LANs can be physically connected by specialized computers called routers. 

​					

## 	The  notion of an internet  protocol

​				**Provides a naming scheme**

​						defines a uniforms format for host address

​						Each host and router is assigned at least one of these internet addresses   

​				**Provides a delivery mechanism**

​						defines a standard transfer unit (packet)

​						packet consist of header and payload

​							header: consists info as packet size,source and destination address.

​							payload: contains data bits sent from source host

​			             <img src="F:\typora\image\image-20220914162321870.png" alt="image-20220914162321870" style="zoom:80%;" />

### 		Global IP Internet 

####                   Based on TCP/IP protocol family

​							**IP**: provides basic naming scheme and unreliable delivery capability of packets from host to 									host.

​							**UDP**: Uses IP to provide unreliable datagram delivery from process to process 

​							**TCP**: uses IP  to provide reliable byte streams from process to process over connections

### 	 Domain naming system (DNS)

​			The internet maintains a mapping between IP address and domain names in huge worldwide                 			distributed database called DNS.

## 	Internet Connections(TCP)

​			Each connection is point to point. Full-duplex and reliable

​			A socket is an endpoint of a connection. socket address is an IP address: port pair.

​			A port is a 16 bits integer that identifies a process.

​						**Ephemeral  port:** assigned automatically by client kernel when client makes a connection 			     		request.

​						**well-known port:** associated with some service provided common port (80 port for web           						services)

## Sockets

### 	Socket Address Structures

```c
struct sockaddr{
  uint16_t sa_family;   /*protocol family (tcp udp ipv6)*/  
    char sa_data[14];   /*address data*/
};

strcut sockaddr_in{  /*for ipv4*/
    uint16_t sin_family;  /*protocol family(always AF_INET)*/ 
    uint16_t sin_port;    /*prot num*/
    struct in_addr  sin_addr; /*ip addr*/
    unsigned char sin_zero[8];  /*8 byte 0*/
};

 struct in_addr 
 {
     in_addr_t s_addr;
 };

typedef in_addr_t unsigned int

/*server*/
int socket(int domain,int type,int protocol); /*create a socket*/
/*Example:
	int clientfd=socket(AF_INET,SOCK_STREAM,0);
	AF_INET:indicate that the 32 bit IPV4 addresses.
	SOCK_STREAM：indicates that the socket will be the end point of a connection
    			(connections only)
*/

int  bind(int sockfd ,sockaddr *addr,socklen_t addrlen) /*associate the server's socket address with a socket desceiptor*/  
    
int listen(int sockfd ,int backlog); 
/*convert sockfd from an active socket to a listening socket that can accept connection requests from client*/

int accept(int listenfd, sockaddr *addr,int addrlen)
/*avtively accept the arrived conncetion*/
/*returns a connected descriptor that can be used to communicate with the client via Unix I/O routines */ 

/*client*/
int socket(int domain,int type,int protocol); /*create a socket*/

int connect(int clientfd,sockaddr *addr,socklen_t addrlen)
```

![image-20220915102157910](F:\typora\image\image-20220915102157910.png)		

​    Finally,the client will use clientfd (return by socket) to communicate with connfd (return by               	accept),the free listenfd will to listen the new connection



​	     ![image-20220915102732011](F:\typora\image\image-20220915102732011.png) 

  

## Host and Server Conversion: getaddrinfo

```c
/*getaddrinfo family*/
int getaddrinfo(const char* host,    /*hostname or address*/
                const char* service,   /*prot or service name*/
                const struct addrinfo *hints, /*input linked list*/
                struct addrinfo **result); /*output linked list head*/

void freeaddrinfo(struct addrinfo *result);  /*Free linked list*/

const char *gai_strerror(int errcode);   /*Return error msg*/

struct addrinfo{
    int ai_flag;        /*hint argumet flags */
    int ai_family;        /*first arg to socket function*/
    int ai_socktype;  /*second arg to socket function*/
    int ai_protocol;   /*third arg tp socket function*/
    char *ai_canonname;   /*canonical host name*/
    size_t ai_addrlen;       /*size of ai_addr struct*/
    struct sockaddr *ai_addr;  
    struct addrinfo *ai_next;
}

/*getnameinfo*/
int  getnameinfo(const sockaddr *sa,socklen_t salen,  /*in: socket addr*/
                 char *host.size_t hostlen,    /*out:host*/
                 char *serv,size_t servlen,    /*out:service*/
                 int flags);                 /*optional,flags*/

```

​		![image-20220915105236678](F:\typora\image\image-20220915105236678.png)\

```c
int main(int argc,cgar **argv)
{
    struct addrinfp *p ,*listp ,hints;
    char buf[MAXLINE];
    int rc,flags;
    
    /*get a list of addrinfo records*/
    memset(&hints.0,sizeof(struct addrinfo));
    hints.ai_family=AF_INET;  //ipv4
    hints.ai_socktype=SOCK_STREAM; //tcp
    if((rc=getaddrinfo(argv[1],NULL,&hints,&listp))!=0){
        fprintf(strerr,"getaddrinfo error:%s\n",gai_strerror(rc));
        exit(1);
    }
   /*walk the list and display each ip address*/
    flags=NI_NUMERICHOST; /*display address instead of name*/
    for(p=listp;p;p=p->ai_next){
        getnameinfo(p->ai_addr,p->ai_addrlen,buf,MAXLINE,NULL,0,flags); 
        printf("%s\n,buf);
    }
    /*clean up*/
    freeaddrinfo(listp);
   	exit(0);
}
               
./hostinfo  localhost
127.0.0.1
```

## Sockets Helper

### 	Establish a connection with a server

```c
int open_clientfd(char *hostname,char *port)
{
    int clientfd;
    struct addrinfo hints,*listp,*p;
    /*get a list of potential server address*/
    memset(&hints,0,sizeof(struct addrinfo));
    hints.ai_socktype=SOCK_STREAM; /*open aconnection*/
    hints.ai_flags=AI_NUMERICSERV; /*using numeric port arg*/
    hints.ai_flags|=AI_ADDRCONFIG; 
    getaddrinfo(hostname,port,&hints,&listp);
    /*walk the list for one that we can successfully connect to*/
    for(p=listp;p;p=p->ai_next)
    {
        if((clientfd=socket(p->ai_family,p->ai_socktype,p->ai_protocol))<0)
            continue; /*socket failed,try next*/
    	if(connect(clientfd,p->ai_addr,p->ai_addrlen)!=-1)
            break; /*success*/
        close(clientfd);  /*connect failed*/
    }
    /*clean up*/
    freeaddrinfo(listp);
    if(!p)  /*all connectionss failed*/
        return -1;
    else 
        return clientfd;
}
```

### 	Create a listening descriptor 

```c
int open_listenfd(char *port)
{
    struct addrinfo hints,*listp,*p;
    int listenfd ,optval=1;
    memset(&hints,0,sizeof(struct addrinfo));
    hints.ai_socktype=SOCK_STREAM; /*open aconnection*/
    hints.ai_flags=AI_PASSIVE; /*on any IP addr*/
    hints.ai_flags|=AI_ADDRCONFIG; 
	getaddrinfo(NULL,port,&hints,&listp);
    
       /*walk the list for one that we can successfully connect to*/
    for(p=listp;p;p=p->ai_next)
    {
        /*Create a socket descriptor*/
        if((listenfd=socket(p->ai_family,p->ai_socktype,p->ai_protocol))<0)
            continue; /*socket failed,try next*/
    	/*Eliminates "Address already in use"error from bind*/
        setsockopt(listenfd,SOL_SOCKET,SO_REUSERADDR,
                   (const void*)&optval,sizeof(int));
        /*Bind the descriptor to the address*/
        if(bind(listenfd,p->ai_addr,p->addrlen)==0)
            break; /*success*/
        close(listenfd);  /*connect failed*/
    }
    /*clean up*/
    freeaddrinfo(listp);
    if(!p)  /*No address worked*/
        return -1;
    /*Make it a listening socket ready to accept conn. requests */
    if(listen(listenfd,LISTENQ)<0)
        close
} 
```

​     