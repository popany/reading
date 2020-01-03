# UNIX Network Programming

## Chapter 1. Introduction

### 1.1 Introduction

Client and server is used by most network-aware applications. Deciding that the client always initiates requests tends to simplify the protocol as well as the programs themselves. Of course, some of the more complex network applications also require **asynchronous callback communication**, where the server initiates a message to the client. But it is far more common for applications to stick to the **basic client/server model**.

### 1.2 A Simple Daytime Client

#### Figure 1.5 TCP daytime client

intro/daytimetcpcli.c

    #include  "unp.h"

    int
    main(int argc, char **argv)
    {
        int     sockfd, n;
        char    recvline[MAXLINE + 1];
        struct sockaddr_in servaddr;

        if (argc != 2)
            err_quit("usage: a.out <IPaddress>");

       if ( (sockfd = socket(AF_INET, SOCK_STREAM, 0)) < 0)
            err_sys("socket error");

        bzero(&servaddr, sizeof(servaddr));
        servaddr.sin_family = AF_INET;
        servaddr.sin_port = htons(13);  /* daytime server */
        if (inet_pton(AF_INET, argv[1], &servaddr.sin_addr) <= 0)
            err_quit("inet_pton error for %s", argv[1]);

        if (connect(sockfd, (SA *) &servaddr, sizeof(servaddr)) < 0)
            err_sys("connect error");

        while ( (n = read(sockfd, recvline, MAXLINE)) > 0){
            recvline[n] = 0;        /* null terminate */
            if (fputs(recvline, stdout) == EOF)
                err_sys("fputs error");
        }
        if (n < 0)
            err_sys("read error");

        exit(0);
    }

The `socket` function creates an Internet (`AF_INET`) **stream** (`SOCK_STREAM`) socket, which is a fancy name for a **TCP socket**. The function returns a small integer descriptor that we can use to identify the socket in all future function calls (e.g., the calls to `connect` and `read` that follow).

TCP socket is synonymous with a **TCP endpoint**.

We must be careful when using TCP because it is a **byte-stream protocol** with no record boundaries.

With a byte-stream protocol, these 26 bytes can be returned in numerous ways: a single TCP segment containing all 26 bytes of data, in 26 TCP segments each containing 1 byte of data, or any other combination that totals to 26 bytes. Normally, a single segment containing all 26 bytes of data is returned, but with larger data sizes, we cannot assume that the server's reply will be returned by a single read. Therefore, when reading from a TCP socket, we always need to code the **read in a loop** and terminate the loop when either read returns `0` (i.e., the other end **closed** the connection) or a value less than `0` (an **error**).

In this example, the **end of the record** is being denoted by the server closing the connection. This technique is also used by version 1.0 of the Hypertext Transfer Protocol (HTTP). Other techniques are available. For example, the Simple Mail Transfer Protocol (SMTP) marks the end of a record with the two-byte sequence of an ASCII carriage return followed by an ASCII linefeed. Sun Remote Procedure Call (RPC) and the Domain Name System (DNS) place a binary count containing the record length in front of each record that is sent when using TCP. The important concept here is that **TCP itself provides no record markers**: If an application wants to delineate the ends of records, it must do so itself and there are a few common ways to accomplish this.

### 1.3 Protocol Independence

It is better to make a program protocol-independent. ***Figure 11.11*** will show a version of this client that is protocol-independent by using the getaddrinfo function (which is called by tcp_connect).

In **Chapter 11**, we will discuss the functions that convert between hostnames and IP addresses, and between service names and ports.

### 1.4 Error Handling: Wrapper Functions

#### Unix `errno` Value

The value of `errno` is set by a function only if an error occurs. Its value is undefined if the function does not return an error. All of the positive error values are constants with all-uppercase names beginning with "`E`," and are normally defined in the `<sys/errno.h>` header. No error has a value of `0`.

Storing `errno` in a global variable does not work with multiple threads that share all global variables. We will talk about solutions to this problem in **Chapter 26**.

### 1.5 A Simple Daytime Server

We can write a simple version of a TCP daytime server, which will work with the client from ***Section 1.2***. We use the wrapper functions that we described in the previous section and show this server in ***Figure 1.9***.

#### Figure 1.9 TCP daytime server

`intro/daytimetcpsrv.c`

    #include     "unp.h".
    #include     <time.h>
    
    int
    main(int argc, char **argv)
    {
        int     listenfd, connfd;
        struct sockaddr_in servaddr;
        char    buff[MAXLINE];
        time_t ticks;
    
        listenfd = Socket(AF_INET, SOCK_STREAM, 0);
    
        bzeros(&servaddr, sizeof(servaddr));
        servaddr.sin_family = AF_INET;
        servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
        servaddr.sin_port = htons(13); /* daytime server */
    
        Bind(listenfd, (SA *) &servaddr, sizeof(servaddr));
    
        Listen(listenfd, LISTENQ);
    
        for ( ; ; ) {
            connfd = Accept(listenfd, (SA *) NULL, NULL);
    
            ticks = time(NULL);
            snprintf(buff, sizeof(buff), "%.24s\r\n", ctime(&ticks));
            Write(connfd, buff, strlen(buff));
    
            Close(connfd);
        }
    }

##### Create a TCP socket

The creation of the TCP socket is identical to the client code.

##### Bind server's well-known port to socket

The server's well-known port (`13` for the daytime service) is bound to the socket by filling in an Internet socket address structure and calling `bind`. We specify the IP address as `INADDR_ANY`, which allows the server to accept a client connection on **any interface**, in case the server host has multiple interfaces. Later we will see how we can restrict the server to accepting a client connection on just a **single interface**.

##### Convert socket to listening socket

By calling `listen`, the socket is converted into a listening socket, on which incoming connections from clients will be accepted by the kernel. These **three steps**, `socket`, `bind`, and `listen`, are the normal steps for any TCP server to **prepare** what we call the *listening descriptor* (listenfd in this example).

The constant `LISTENQ` is from our `unp.h` header. It specifies the **maximum number** of **client connections** that the kernel will queue for this listening descriptor. We say much more about this queueing in ***Section 4.5***.

##### Accept client connection, send reply

Normally, the server process is put to **sleep** in the call to `accept`, waiting for a client connection to arrive and be accepted. A TCP connection uses what is called a **three-way handshake** to establish a connection. When this **handshake completes**, **`accept` returns**, and the return value from the function is a **new descriptor** (`connfd`) that is called the **connected descriptor**. This new descriptor is used for communication with the new client. A new descriptor is returned by `accept` for each client that connects to our server.

##### Terminate connection

The server closes its connection with the client by calling `close`. This initiates the normal TCP connection termination sequence: a **FIN** is **sent** in each direction and each FIN is **acknowledged** by the other end. We will say much more about TCP's **three-way handshake** and the **four TCP packets** used to terminate a TCP connection in ***Section 2.6***.

As with the client in the previous section, we have only examined this server briefly, saving all the details for later in the book. Note the following points:

* As with the client, the server is protocol-dependent on `IPv4`. We will show a protocol-independent version that uses the `getaddrinfo` function in ***Figure 11.13***.

* Our server handles only one client at a time. If multiple client connections arrive at about the same time, the kernel **queues** them, up to some **limit**, and returns them to `accept` one at a time.

* The server that we show in ***Figure 1.9*** is called an **iterative server** because it iterates through each client, one at a time. There are numerous techniques for writing a **concurrent server**, one that handles multiple clients at the same time. The simplest technique for a concurrent server is to call the Unix `fork` function (***Section 4.7***), creating one child process for each client. Other techniques are to use threads instead of `fork` (***Section 26.4***), or to pre-`fork` a fixed number of children when the server starts (***Section 30.6***).

* If we start a server like this from a shell command line, we might want the server to run for a long time, since servers often run for as long as the system is up. This requires that we add code to the server to run correctly as a Unix **daemon**: a process that can run in the background, unattached to a terminal. We will cover this in ***Section 13.4***.

### 1.6 Roadmap to Client/Server Examples in the Text

### 1.7 OSI Model

A common way to describe the layers in a network is to use the International Organization for Standardization (ISO) open systems interconnection (OSI) model for computer communications.

#### Figure 1.14. Layers in OSI model and Internet protocol suite

We consider the bottom two layers of the OSI model as the **device driver** and **networking hardware** that are supplied with the system. Normally, we need not concern ourselves with these layers other than being aware of some properties of the datalink, such as the 1500-byte Ethernet maximum transfer unit (**MTU**), which we describe in ***Section 2.11***.

The network layer is handled by the IPv4 and IPv6 protocols, both of which we will describe in ***Appendix A***. The transport layers that we can choose from are `TCP` and `UDP`, and we will describe these in ***Chapter 2***. We show a **gap** between TCP and UDP in Figure 1.14 to indicate that it is possible for an application to **bypass** the transport layer and use IPv4 or IPv6 directly. This is called a **raw socket**, and we will talk about this in ***Chapter 28***.

The upper three layers of the OSI model are combined into a single layer called the **application**. This is the Web client (browser), Telnet client, Web server, FTP server, or whatever application we are using. With the Internet protocols, there is rarely any distinction between the upper three layers of the OSI model.

The sockets programming interfaces described in this book are interfaces from the **upper three layers** (the "application") into the **transport layer**. This is the focus of this book: how to write applications using sockets that use either `TCP` or `UDP`. We already mentioned raw sockets, and in ***Chapter 29*** we will see that we can even **bypass** the **IP layer** completely to read and write our own **datalink-layer** frames.

Why do sockets provide the interface from the **upper three layers** of the OSI model into the **transport layer**? There are two reasons for this design, which we note on the right side of ***Figure 1.14***. First, the upper three layers handle all the details of the **application** (FTP, Telnet, or HTTP, for example) and know little about the **communication details**. The lower four layers know little about the application, but handle all the communication details: **sending** data, **waiting** for acknowledgments, **sequencing** data that arrives out of order, calculating and verifying **checksums**, and so on. The second reason is that the upper three layers often form what is called a **user process** while the lower four layers are normally provided as part of the operating system (OS) **kernel**. Unix provides this separation between the user process and the kernel, as do many other contemporary operating systems. Therefore, the interface between layers 4 and 5 is the natural place to build the **API**.

### 1.8 BSD Networking History

The **sockets API** originated with the 4.2BSD system, released in 1983. ***Figure 1.15*** shows the development of the various BSD releases, noting the major TCP/IP developments. A few changes to the sockets API also took place in 1990 with the 4.3BSD Reno release, when the OSI protocols went into the BSD kernel.

#### Figure 1.15. History of various BSD releases

### 1.9 Test Networks and Hosts

#### Figure 1.16. Networks and hosts used for most examples in the text

#### Discovering Network Topology

We show the network topology in ***Figure 1.16*** for the hosts used for the examples throughout this text, but you may need to know your own network topology to run the examples and exercises on your own network. Although there are no current Unix standards with regard to network configuration and administration, two basic commands are provided by most Unix systems and can be used to discover some details of a network: `netstat` and `ifconfig`.

1. `netstat -i` provides information on the interfaces. We also specify the `-n` flag to print numeric addresses, instead of trying to find names for the networks. This shows us the interfaces and their names.

2. `netstat -r` shows the routing table, which is another way to determine the interfaces. We normally specify the `-n` flag to print numeric addresses. This also shows the IP address of the default router.

3. Given the interface names, we execute `ifconfig` to obtain the details for each interface.

4. One way to find the IP address of many hosts on the local network is to `ping` the broadcast address (which we found in the previous step).

### 1.10 Unix Standards

At the time of this writing, the most interesting Unix standardization activity was being done by The Austin Common Standards Revision Group (CSRG). Their efforts have produced roughly 4,000 pages of specifications covering over 1,700 programming interfaces [Josey 2002]. These specifications carry both the IEEE POSIX designation as well as The Open Group's Technical Standard designation. The net result is that you'll likely encounter references to the same standard by various names: ISO/IEC 9945:2002, IEEE Std 1003.1-2001, and the Single Unix Specification Version 3, for example. In this text, we will refer to this standard as simply The **POSIX Specification**, except in sections like this one where we are discussing specifics of various older standards.

The easiest way to acquire a copy of this consolidated standard is to either order it on CD-ROM or access it via the Web (free of charge). The starting point for either of these methods is

[http://www.UNIX.org/version3](http://www.UNIX.org/version3)

#### Background on POSIX

#### Background on The Open Group

#### Unification of Standards

#### Internet Engineering Task Force (IETF)

### 1.11 64-Bit Architectures

During the mid to late 1990s, the trend began toward 64-bit architectures and 64-bit software. One reason is for larger addressing within a process (i.e., 64-bit pointers), which can address large amounts of memory (more than 232 bytes). The common programming model for existing 32-bit Unix systems is called the **ILP32 model**, denoting that integers (I), long integers (L), and pointers (P) occupy 32 bits. The model that is becoming most prevalent for 64-bit Unix systems is called the **LP64 model**, meaning only long integers (L) and pointers (P) require 64 bits. ***Figure 1.17*** compares these two models.

#### Figure 1.17. Comparison of number of bits to hold various datatypes for the ILP32 and LP64 models

ANSI C invented the `size_t` datatype, which is used, for example, as the argument to `malloc` (the number of bytes to allocate), and as the third argument to `read` and `write` (the number of bytes to read or write). On a 32-bit system, `size_t` is a 32-bit value, but on a 64-bit system, it must be a 64-bit value, to take advantage of the larger addressing model. This means a 64-bit system will probably contain a typedef of `size_t` to be an `unsigned long`. The networking API **problem** is that some drafts of POSIX.1g specified that function arguments containing the size of a socket address structures have the `size_t` datatype (e.g., the third argument to `bind` and `connect`). Some XTI structures also had members with a datatype of `long` (e.g., the    `t_info` and `t_opthdr` structures). If these had been left as is, both would change from 32-bit values to 64-bit values when a Unix system changes from the ILP32 to the LP64 model. In both instances, there is **no need** for a 64-bit datatype: The length of a socket address structure is a few hundred bytes at most, and the use of long for the XTI structure members was a **mistake**.

The solution is to use datatypes designed specifically to handle these scenarios. The sockets API uses the `socklen_t` datatype for lengths of socket address structures, and XTI uses the `t_scalar_t` and `t_uscalar_t` datatypes. The reason for not changing these values from 32 bits to 64 bits is to make it easier to provide binary compatibility on the new 64-bit systems for applications compiled under 32-bit systems.

### 1.12 Summary

***Figure 1.5*** shows a complete, albeit simple, TCP client that fetches the current time and date from a specified server, and ***Figure 1.9*** shows a complete version of the server. These two examples introduce many of the terms and concepts that are expanded on throughout the rest of the book.

Our client was protocol-dependent on IPv4 and we modified it to use IPv6 instead. But this just gave us another protocol-dependent program. In ***Chapter 11***, we will develop some functions to let us write protocol-independent code, which will be important as the Internet starts using IPv6.

Throughout the text, we will use the wrapper functions developed in ***Section 1.4*** to reduce the size of our code, yet still check every function call for an error return. Our wrapper functions all begin with a capital letter.

The Single Unix Specification Version 3, known by several other names and called simply The POSIX Specification by us, is the confluence of two long-running standards efforts, finally drawn together by The Austin Group.

Readers interested in the history of Unix networking should consult [Salus 1994] for a description of Unix history, and [Salus 1995] for the history of TCP/IP and the Internet.

## Chapter 2. The Transport Layer: TCP, UDP, and SCTP 

### 2.1 Introduction
