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

This chapter focuses on the **transport layer**: TCP, UDP, and Stream Control Transmission Protocol (SCTP). Most client/server applications use either TCP or UDP. SCTP is a newer protocol, originally designed for transport of telephony signaling across the Internet. These transport protocols use the **network-layer** protocol IP, either IPv4 or IPv6. While it is possible to use IPv4 or IPv6 directly, bypassing the transport layer, this technique, often called raw sockets, is used much less frequently. Therefore, we have a more detailed description of IPv4 and IPv6, along with ICMPv4 and ICMPv6, in ***Appendix A***.

**UDP** is a simple, unreliable **datagram** protocol, while **TCP** is a sophisticated, reliable **byte-stream** protocol. SCTP is similar to TCP as a reliable transport protocol, but it also provides **message boundaries**, transport-level support for multihoming, and a way to minimize head-of-line blocking. We need to understand the services provided by these transport protocols to the application, so that we know what is handled by the protocol and what we must handle in the application.

There are features of TCP that, when understood, make it easier for us to write robust clients and servers. Also, when we understand these features, it becomes easier to debug our clients and servers using commonly provided tools such as netstat. We cover various topics in this chapter that fall into this category: TCP's three-way handshake, TCP's connection termination sequence, and TCP's `TIME_WAIT` state; SCTP's four-way handshake and SCTP's connection termination; plus SCTP, TCP, and UDP buffering by the socket layer, and so on.

### 2.2 The Big Picture

Although the protocol suite is called "TCP/IP," there are more members of this family than just TCP and IP. ***Figure 2.1*** shows an overview of these protocols.

#### Figure 2.1. Overview of TCP/IP protocols

### 2.3 User Datagram Protocol (UDP)

UDP is a simple transport-layer protocol. It is described in RFC 768 [Postel 1980]. The application writes a **message** to a UDP socket, which is then encapsulated in a **UDP datagram**, which is then further encapsulated as an **IP datagram**, which is then sent to its destination. There is **no guarantee** that a UDP datagram will ever **reach** its final destination, that **order** will be preserved across the network, or that datagrams arrive only **once**.

The problem that we encounter with network programming using UDP is its **lack of reliability**. If a datagram reaches its final destination but the **checksum** detects an error, or if the datagram is **dropped** in the network, it is not delivered to the UDP socket and is not automatically **retransmitted**. If we want to be certain that a datagram reaches its destination, we can build lots of features into our application: **acknowledgments** from the other end, **timeouts**, **retransmissions**, and the like.

Each UDP datagram has a length. The **length** of a datagram is passed to the receiving application along with the data. We have already mentioned that TCP is a byte-stream protocol, without any record boundaries at all (***Section 1.2***), which differs from UDP.

We also say that UDP provides a **connectionless** service, as there need not be any **long-term** relationship between a UDP client and server. For example, a UDP client can create a socket and send a datagram to a given server and then immediately send another datagram on the **same socket** to a **different server**. Similarly, a UDP server can receive several datagrams on a **single UDP socket**, each from a **different client**.

### 2.4 Transmission Control Protocol (TCP)

The service provided by TCP to an application is different from the service provided by UDP. TCP is described in RFC 793 [Postel 1981c], and updated by RFC 1323 [Jacobson, Braden, and Borman 1992], RFC 2581 [Allman, Paxson, and Stevens 1999], RFC 2988 [Paxson and Allman 2000], and RFC 3390 [Allman, Floyd, and Partridge 2002]. First, TCP provides **connections** between clients and servers. A TCP client establishes a connection with a given server, exchanges data with that server across the connection, and then terminates the connection.

TCP also provides **reliability**. When TCP sends data to the other end, it requires an **acknowledgment** in return. If an acknowledgment is not received, TCP automatically **retransmits** the data and waits a **longer** amount of time. After some number of retransmissions, TCP will give up, with the total amount of time spent trying to send data typically between **4 and 10 minutes** (depending on the implementation).

Note that TCP does not guarantee that the data will be received by the other endpoint, as this is impossible. It delivers data to the other endpoint if possible, and **notifies the user** (by giving up on retransmissions and breaking the connection) if it is not possible. Therefore, TCP cannot be described as a 100% reliable protocol; it provides **reliable delivery** of data or **reliable notification** of failure.

TCP contains algorithms to estimate the **round-trip time** (RTT) between a client and server dynamically so that it knows how long to wait for an acknowledgment. For example, the RTT on a LAN can be milliseconds while across a WAN, it can be seconds. Furthermore, TCP continuously estimates the RTT of a given connection, because the RTT is affected by variations in the network traffic.

TCP also **sequences** the data by associating a sequence number with every byte that it sends. For example, assume an application writes 2,048 bytes to a TCP socket, causing TCP to send two segments, the first containing the data with sequence numbers 1–1,024 and the second containing the data with sequence numbers 1,025–2,048. (A **segment** is the unit of data that TCP passes to IP.) If the segments arrive out of order, the receiving TCP will reorder the two segments based on their sequence numbers before passing the data to the receiving application. If TCP receives duplicate data from its peer (say the peer thought a segment was lost and retransmitted it, when it wasn't really lost, the network was just overloaded), it can detect that the data has been duplicated (from the sequence numbers), and discard the **duplicate** data.

There is no reliability provided by **UDP**. UDP itself does not provide anything like **acknowledgments**, **sequence** numbers, **RTT** estimation, **timeouts**, or **retransmissions**. If a UDP datagram is duplicated in the network, two copies can be delivered to the receiving host. Also, if a UDP client sends two datagrams to the same destination, they can be reordered by the network and arrive out of order. UDP applications must handle all these cases, as we will show in ***Section 22.5***.

TCP provides **flow control**. TCP always tells its peer exactly how many **bytes of data** it is **willing to accept** from the peer at any one time. This is called the **advertised window**. At any time, the window is the amount of room currently available in the **receive buffer**, guaranteeing that the sender cannot overflow the receive buffer. The window **changes dynamically** over time: As data is received from the sender, the window size **decreases**, but as the receiving application reads data from the buffer, the window size **increases**. It is possible for the window to **reach 0**: when TCP's receive buffer for a socket is full and it must wait for the application to read data from the buffer before it can take any more data from the peer.

UDP provides no flow control. It is easy for a **fast UDP sender** to transmit datagrams at a rate that the UDP receiver cannot keep up with, as we will show in ***Section 8.13***.

Finally, a TCP connection is **full-duplex**. This means that an application can **send** and **receive** data in **both directions** on a given connection at any time. This means that TCP must **keep track of state information** such as **sequence numbers** and **window sizes** for each direction of data flow: sending and receiving. After a full-duplex connection is established, it can be turned into a **simplex connection** if desired (see ***Section 6.6***).

**UDP can be full-duplex**.

### 2.5 Stream Control Transmission Protocol (SCTP) 

SCTP provides services similar to those offered by UDP and TCP. SCTP is described in RFC 2960 [Stewart et al. 2000], and updated by RFC 3309 [Stone, Stewart, and Otis 2002]. An introduction to SCTP is available in RFC 3286 [Ong and Yoakum 2002]. **SCTP** provides associations between clients and servers. SCTP also provides applications with **reliability**, **sequencing**, **flow control**, and **full-duplex** data transfer, like TCP. The word "association" is used in SCTP instead of "connection" to avoid the connotation that a connection involves communication between only two IP addresses. An **association** refers to a communication between **two systems**, which may involve **more than two addresses** due to **multihoming**.

Unlike TCP, SCTP is **message-oriented**. It provides sequenced delivery of individual records. Like UDP, the **length of a record** written by the sender is passed to the receiving application.

SCTP can provide **multiple streams** between connection endpoints, each with its **own reliable sequenced** delivery of messages. A lost message in one of these streams does **not block delivery** of messages in any of the other streams. This approach is in contrast to TCP, where a loss at any point in the single stream of bytes **blocks delivery** of all future data on the connection until the loss is repaired.

SCTP also provides a **multihoming** feature, which allows a **single SCTP endpoint** to support **multiple IP addresses**. This feature can provide increased robustness against network failure. An endpoint can have multiple **redundant** network connections, where each of these networks has a different connection to the Internet infrastructure. SCTP can work around a failure of one network or path across the Internet by switching to another address already associated with the SCTP association.

Similar robustness can be obtained from **TCP** with help from **routing protocols**. For example, BGP connections within a domain (iBGP) often use addresses that are assigned to a virtual interface within the router as the endpoints of the TCP connection. The domain's routing protocol ensures that if there is a route between two routers, it can be used, which would not be possible if the addresses used belonged to an interface that went down, for example. SCTP's multihoming feature allows hosts to multihome, not just routers, and allows this multihoming to occur across different service providers, which the routing-based TCP method cannot allow.

### 2.6 TCP Connection Establishment and Termination

To aid in our understanding of the `connect`, `accept`, and `close` functions and to help us debug TCP applications using the `netstat` program, we must understand how TCP connections are established and terminated, and TCP's **state transition diagram**.

#### Three-Way Handshake

The following scenario occurs when a TCP connection is established:

1. The server must be prepared to accept an incoming connection. This is normally done by calling `socket`, `bind`, and `listen` and is called a **passive open**.

2. The client issues an active open by calling `connect`. This causes the client TCP to send a **"synchronize" (SYN) segment**, which tells the server the client's **initial sequence number** for the data that the client will send on the connection. Normally, there is **no data sent with the SYN**; it just contains an **IP header**, a **TCP header**, and possible **TCP options** (which we will talk about shortly).

3. The server must **acknowledge (ACK)** the client's SYN and the server must also send its own **SYN** containing the **initial sequence number** for the data that the server will send on the connection. The server sends its **SYN** and the **ACK** of the client's **SYN** in a **single segment**.

4. The client must **acknowledge** the server's SYN.

The **minimum number** of packets required for this exchange is three; hence, this is called **TCP's three-way handshake**. We show the **three segments** in ***Figure 2.2***.

#### Figure 2.2. TCP three-way handshake

We show the client's initial sequence number as *J* and the server's initial sequence number as *K*. The **acknowledgment number** in an ACK is the **next expected sequence number** for the end sending the ACK. Since a SYN occupies one byte of the sequence number space, the acknowledgment number in the ACK of each SYN is the initial sequence number plus one. Similarly, the ACK of each FIN is the sequence number of the FIN plus one.

An everyday analogy for establishing a TCP connection is the telephone system [Nemeth 1997]. The `socket` function is the equivalent of having a **telephone** to use. `bind` is telling other people your **telephone number** so that they can call you. `listen` is **turning on the ringer** so that you will hear when an incoming call arrives. `connect` requires that we know the other person's phone number and dial it. `accept` is when the person being called **answers the phone**. Having the client's identity returned by `accept` (where the identify is the client's IP address and port number) is similar to having the **caller ID** feature show the caller's phone number. One difference, however, is that `accept` returns the client's identity only after the connection has been established, whereas the caller ID feature shows the caller's phone number before we choose whether to answer the phone or not. If the **DNS** is used (***Chapter 11***), it provides a service analogous to a **telephone book**. `getaddrinfo` is similar to looking up a person's phone number in the phone book. `getnameinfo` would be the equivalent of having a phone book sorted by telephone numbers that we could search, instead of a book sorted by name.

#### TCP Options

Each SYN can contain **TCP options**. Commonly used options include the following:

* **MSS** option. With this option, the TCP sending the SYN announces its **maximum segment size**, the maximum amount of data that it is willing to accept in each TCP segment, on this connection. The sending TCP uses the receiver's MSS value as the maximum size of a segment that it sends. We will see how to fetch and set this TCP option with the `TCP_MAXSEG` socket option (***Section 7.9***).

* **Window scale** option. The maximum window that either TCP can advertise to the other TCP is 65,535, because the corresponding field in the TCP header occupies 16 bits. But, high-speed connections, common in today's Internet (45 Mbits/sec and faster, as described in RFC 1323 [Jacobson, Braden, and Borman 1992]), or long delay paths (satellite links) require a larger window to obtain the maximum throughput possible. This newer option specifies that the advertised window in the TCP header must be scaled (left-shifted) by 0–14 bits, providing a maximum window of almost one gigabyte (65,535 x 2^14). Both end-systems must support this option for the window scale to be used on a connection. We will see how to affect this option with the `SO_RCVBUF` socket option (***Section 7.5***).

To provide interoperability with **older implementations** that do not support this option, the following rules apply. TCP can send the option with its SYN as part of an active open. But, it can scale its windows only if the other end also sends the option with its SYN. Similarly, the server's TCP can send this option only if it receives the option with the client's SYN. This logic assumes that implementations **ignore options** that they do not understand, which is required and common, but unfortunately, not guaranteed with all implementations.

* Timestamp option. This option is needed for high-speed connections to prevent possible data corruption caused by old, delayed, or duplicated segments. Since it is a newer option, it is negotiated similarly to the window scale option. As network programmers there is nothing we need to worry about with this option.

These common options are supported by most implementations. The latter two are sometimes called the "RFC 1323 options," as that RFC [Jacobson, Braden, and Borman 1992] specifies the options. They are also called the "long fat pipe options," since a network with either a high bandwidth or a long delay is called a long fat pipe. ***Chapter 24*** of TCPv1 contains more details on these options.

#### TCP Connection Termination

While it takes three segments to establish a connection, it takes **four** to terminate a connection.

1 One application calls `close` first, and we say that this end performs the **active close**. This end's TCP sends a **FIN segment**, which means it is **finished sending data**.

2 The other end that receives the FIN performs the **passive close**. The received FIN is **acknowledged** by TCP. The receipt of the FIN is also passed to the application as an **end-of-file** (after any data that may have already been queued for the application to receive), since the receipt of the FIN means the application will **not receive** any additional data on the connection.

3 Sometime later, the application that received the end-of-file will `close` its socket. This causes its TCP to **send a FIN**.

4 The TCP on the system that receives this **final FIN** (the end that did the active close) **acknowledges** the FIN.

Since a FIN and an ACK are required in each direction, **four segments** are normally required. We use the qualifier "normally" because in some scenarios, the FIN in **Step 1** is sent with data. Also, the segments in **Steps 2 and 3** are both from the end performing the passive close and could be **combined** into one segment. We show these packets in ***Figure 2.3***.

#### Figure 2.3. Packets exchanged when a TCP connection is closed

A FIN occupies one byte of sequence number space just like a SYN. Therefore, the ACK of each FIN is the sequence number of the FIN **plus one**.

Between *Steps 2 and 3* it is possible for data to flow from the end doing the passive close to the end doing the active close. This is called a **half-close** and we will talk about this in detail with the `shutdown` function in ***Section 6.6***.

The sending of each FIN occurs when a socket is closed. We indicated that the application calls close for this to happen, but realize that when a **Unix process terminates**, either voluntarily (calling exit or having the main function return) or involuntarily (receiving a signal that terminates the process), **all open descriptors are closed**, which will also cause a FIN to be sent on any TCP connection that is still open.

Although we show the client in ***Figure 2.3*** performing the active close, **either end**—the client or the server—**can perform** the active close. Often the client performs the active close, but with some protocols (notably HTTP/1.0), the server performs the active close.

#### TCP State Transition Diagram

The operation of TCP with regard to connection establishment and connection termination can be specified with a **state transition diagram**. We show this in ***Figure 2.4***.

#### Figure 2.4. TCP state transition diagram

There are **11** different states defined for a connection and the rules of TCP dictate the transitions from one state to another, based on the **current state** and the **segment received** in that state. For example, if an application performs an active open in the **CLOSED** state, TCP sends a SYN and the new state is **SYN_SENT**. If TCP next receives a SYN with an ACK, it sends an ACK and the new state is **ESTABLISHED**. This final state is where most data transfer occurs.

The two arrows leading from the ESTABLISHED state deal with the termination of a connection. If an application calls close before receiving a FIN (an active close), the transition is to the **FIN_WAIT_1** state. But if an application receives a FIN while in the ESTABLISHED state (a passive close), the transition is to the **CLOSE_WAIT** state.

We denote the normal client transitions with a darker solid line and the normal server transitions with a darker dashed line. We also note that there are two transitions that we have not talked about: a **simultaneous open** (when both ends send SYNs at about the same time and the SYNs cross in the network) and a **simultaneous close** (when both ends send FINs at the same time). ***Chapter 18*** of TCPv1 contains examples and a discussion of both scenarios, which are possible but rare.

One reason for showing the state transition diagram is to show the 11 TCP states with their names. These states are displayed by `netstat`, which is a useful tool when debugging client/server applications. We will use `netstat` to monitor state changes in ***Chapter 5***.

#### Watching the Packets

***Figure 2.5*** shows the actual packet exchange that takes place for a complete TCP connection: the connection establishment, data transfer, and connection termination. We also show the TCP states through which each endpoint passes.

#### Figure 2.5. Packet exchange for TCP connection

The client in this example announces an MSS of 536 (indicating that it implements only the minimum reassembly buffer size) and the server announces an MSS of 1,460 (typical for IPv4 on an Ethernet). It is okay for the **MSS** to be **different** in each direction.

Once a connection is established, the client forms a request and sends it to the server. We assume this request fits into a single TCP segment (i.e., less than 1,460 bytes given the server's announced MSS). The server processes the request and sends a reply, and we assume that the reply fits in a single segment (less than 536 in this example). We show both data segments as bolder arrows. Notice that the acknowledgment of the client's request is sent with the server's reply. This is called **piggybacking** and will normally happen when the time it takes the server to process the request and generate the reply is less than around 200 ms. If the server takes longer, say one second, we would see the acknowledgment followed later by the reply. (The dynamics of TCP data flow are covered in detail in ***Chapters 19 and 20*** of TCPv1.)

We then show the four segments that terminate the connection. Notice that the end that performs the active close (the client in this scenario) enters the **TIME_WAIT** state. We will discuss this in the next section.

It is important to notice in ***Figure 2.5*** that if the entire purpose of this connection was to send a one-segment request and receive a one-segment reply, there would be eight segments of overhead involved when using TCP. If UDP was used instead, only two packets would be exchanged: the request and the reply. But switching from TCP to UDP removes all the reliability that TCP provides to the applizcation, pushing lots of these details from the transport layer (TCP) to the UDP application. Another important feature provided by TCP is congestion control, which must then be handled by the UDP application. Nevertheless, it is important to understand that many applications are built using UDP because the application exchanges small amounts of data and UDP avoids the overhead of TCP connection establishment and connection termination.

### 2.7 TIME_WAIT State

Undoubtedly, one of the most misunderstood aspects of TCP with regard to network programming is its **TIME_WAIT** state. We can see in ***Figure 2.4*** that the end that performs the active close goes through this state. The duration that this endpoint remains in this state is twice the **maximum segment lifetime** (**MSL**), sometimes called **2MSL**.

Every implementation of TCP must choose a value for the **MSL**. The recommended value in RFC 1122 [Braden 1989] is 2 minutes, although Berkeley-derived implementations have traditionally used a value of 30 seconds instead. This means the duration of the **TIME_WAIT** state is **between 1 and 4 minutes**. The MSL is the maximum amount of time that any given IP datagram can live in a network. We know this time is bounded because every datagram contains an 8-bit hop limit (the IPv4 TTL field in ***Figure A.1*** and the IPv6 hop limit field in ***Figure A.2***) with a maximum value of 255. Although this is a hop limit and not a true time limit, the assumption is made that a packet with the maximum hop limit of 255 cannot exist in a network for more than MSL seconds.

The way in which a packet gets "lost" in a network is usually the result of **routing anomalies**. A router crashes or a link between two routers goes down and it takes the routing protocols seconds or minutes to stabilize and find an alternate path. During that time period, **routing loops** can occur (router A sends packets to router B, and B sends them back to A) and packets can get caught in these loops. In the meantime, assuming the lost packet is a TCP segment, the sending TCP times out and retransmits the packet, and the retransmitted packet gets to the final destination by some alternate path. But sometime later (up to MSL seconds after the lost packet started on its journey), the routing loop is corrected and the packet that was lost in the loop is sent to the final destination. This original packet is called a **lost duplicate** or a wandering duplicate. TCP must handle these duplicates.

There are two reasons for the TIME_WAIT state:

1. To implement TCP's full-duplex connection termination reliably

2. To allow old duplicate segments to expire in the network

The first reason can be explained by looking at ***Figure 2.5*** and assuming that the final ACK is lost. The server will resend its final FIN, so the client must maintain state information, allowing it to resend the final ACK. If it did not maintain this information, it would respond with an RST (a different type of TCP segment), which would be interpreted by the server as an error. If TCP is performing all the work necessary to terminate both directions of data flow cleanly for a connection (its full-duplex close), then it must correctly handle the loss of any of these four segments. This example also shows why the end that performs the **active close** is the end that remains in the **TIME_WAIT** state: because that end is the one that might have to retransmit the final ACK.

To understand the second reason for the TIME_WAIT state, assume we have a TCP connection between 12.106.32.254 port 1500 and 206.168.112.219 port 21. This connection is closed and then sometime later, we establish another connection between the same IP addresses and ports: 12.106.32.254 port 1500 and 206.168.112.219 port 21. This latter connection is called an **incarnation** of the previous connection since the IP addresses and ports are the same. TCP must prevent **old duplicates** from a connection from reappearing at some later time and being misinterpreted as belonging to a new incarnation of the same connection. To do this, TCP will **not initiate** a new incarnation of a connection that is currently in the TIME_WAIT state. Since the duration of the **TIME_WAIT** state is **twice the MSL**, this allows MSL seconds for a packet in one direction to be lost, and another MSL seconds for the reply to be lost. By enforcing this rule, we are guaranteed that when we successfully establish a TCP connection, all old duplicates from previous incarnations of the connection have expired in the network.

There is an exception to this rule. Berkeley-derived implementations will initiate a new incarnation of a connection that is currently in the TIME_WAIT state if the arriving SYN has a sequence number that is "**greater than**" the **ending sequence** number from the previous incarnation. Pages 958–959 of TCPv2 talk about this in more detail. This requires the server to perform the active close, since the TIME_WAIT state must exist on the end that receives the next SYN. This capability is used by the rsh command. RFC 1185 [Jacobson, Braden, and Zhang 1990] talks about some pitfalls in doing this.

### 2.8 SCTP Association Establishment and Termination

SCTP is connection-oriented like TCP, so it also has association establishment and termination handshakes. However, SCTP's handshakes are different than TCP's, so we describe them here.

#### Four-Way Handshake

The following scenario, similar to TCP, occurs when an SCTP association is established:

1. The server must be prepared to accept an incoming association. This preparation is normally done by calling `socket`, `bind`, and `listen` and is called a **passive open**.

2. The client issues an active open by calling **`connect`** or by **sending a message**, which implicitly opens the association. This causes the client SCTP to send an **INIT message** (which stands for "initialization") to tell the server the client's **list of IP addresses**, **initial sequence number**, **initiation tag** to identify all packets in this association, **number of outbound streams** the client is requesting, and number of inbound streams the client can support.

3. The server acknowledges the client's INIT message with an **INIT-ACK message**, which contains the server's **list of IP addresses**, **initial sequence number**, **initiation tag**, **number of outbound streams** the server is requesting, number of inbound streams the server can support, and a **state cookie**. The state cookie contains all of the state that the server needs to ensure that the association is valid, and is digitally signed to ensure its validity.

4. The client echos the server's state cookie with a **COOKIE-ECHO message**. This message may also contain user data bundled within the same packet.

5. The server **acknowledges** that the cookie was correct and that the association was established with a **COOKIE-ACK message**. This message may also contain user data bundled within the same packet.

The minimum number of packets required for this exchange is four; hence, this process is called SCTP's **four-way handshake**. We show a picture of the four segments in ***Figure 2.6***.

#### Figure 2.6. SCTP four-way handshake

The SCTP four-way handshake is similar in many ways to TCP's three-way handshake, except for the **cookie generation**, which is an integral part. The INIT carries with it (along with its many parameters) a verification tag, Ta, and an initial sequence number, *J*. The tag Ta must be present in every packet sent by the peer for the life of the association. The initial sequence number *J* is used as the starting sequence number for DATA messages termed DATA chunks. The peer also chooses a verification tag, *Tz*, which must be present in each of its packets for the life of the association. Along with the verification tag and initial sequence number, *K*, the receiver of the INIT also sends a cookie, *C*. The cookie contains all the state needed to set up the SCTP association, so that the server's SCTP stack does not need to keep information about the associating client. Further details on SCTP's association setup can be found in ***Chapter 4*** of [Stewart and Xie 2001].

At the conclusion of the four-way handshake, each side chooses a primary destination address. The primary destination address is used as the default destination to which data will be sent in the absence of network failure.

The four-way handshake is used in SCTP to avoid a form of denial-of-service attack we will discuss in ***Section 4.5***.

SCTP's four-way handshake using Cookies formalizes a method of protection against this attack. Many TCP implementations use a similar method; the big difference is that in TCP, the cookie state must be encoded into the initial sequence number, which is only 32 bits. SCTP provides an arbitrary-length field, and requires cryptographic security to prevent attacks.

#### SCTP State Transition Diagram

The operation of SCTP with regard to association establishment and termination can be specified with a **state transition diagram**. We show this in ***Figure 2.8***.

#### Figure 2.8. SCTP state transition diagram

As in ***Figure 2.4***, the transitions from one state to another in the state machine are dictated by the rules of SCTP, based on the current state and the chunk received in that state. For example, if an application performs an active open in the CLOSED state, SCTP sends an INIT and the new state is COOKIE-WAIT. If SCTP next receives an INIT ACK, it sends a COOKIE ECHO and the new state is COOKIE-ECHOED. If SCTP then receives a COOKIE ACK, it moves to the ESTABLISHED state. This final state is where most data transfer occurs, although DATA chunks can be piggybacked on COOKIE ECHO and COOKIE ACK chunks.

The two arrows leading from the ESTABLISHED state deal with the termination of an association. If an application calls close before receiving a SHUTDOWN (an active close), the transition is to the SHUTDOWN-PENDING state. However, if an application receives a SHUTDOWN while in the ESTABLISHED state (a passive close), the transition is to the SHUTDOWN-RECEIVED state.

#### Watching the Packets

***Figure 2.9*** shows the actual packet exchange that takes place for a sample SCTP association: the association establishment, data transfer, and association termination. We also show the SCTP states through which each endpoint passes.

#### Figure 2.9. Packet exchange for SCTP association

In this example, the client piggybacks its first data chunk on the COOKIE ECHO, and the server replies with data on the COOKIE ACK. In general, the COOKIE ECHO will often have one or more DATA chunks bundled with it when the application is using the one-to-many interface style (we will discuss the one-to-one and one-to-many interface styles in ***Section 9.2***).

The unit of information within an SCTP packet is a "chunk." A "chunk" is self-descriptive and contains a chunk type, chunk flags, and a chunk length. This approach facilitates the bundling of chunks simply by combining multiple chunks into an SCTP outbound packet (details on chunk bundling and normal data transmission procedures can be found in ***Chapter 5*** of [Stewart and Xie 2001]).

#### SCTP Options

SCTP uses parameters and chunks to facilitate optional features. New features are defined by adding either of these two items, and allowing normal SCTP processing rules to report unknown parameters and unknown chunks. The upper two bits of both the parameter space and the chunk space dictate what an SCTP receiver should do with an unknown parameter or chunk (further details can be found in ***Section 3.1*** of [Stewart and Xie 2001]).

Currently, two extensions for SCTP are under development:

1. The dynamic address extension, which allows cooperating SCTP endpoints to dynamically add and remove IP addresses from an existing association.

2. The partial reliability extension, which allows cooperating SCTP endpoints, under application direction, to limit the retransmission of data. When a message becomes too old to send (according to the application's direction), the message will be skipped and thus no longer sent to the peer. This means that not all data is assured of arrival at the other end of the association.

### 2.9 Port Numbers

At any given time, multiple processes can be using any given transport: UDP, SCTP, or TCP. All three transport layers use 16-bit integer port numbers to differentiate between these processes.

When a client wants to contact a server, the client must identify the server with which it wants to communicate. TCP, UDP, and SCTP define a group of well-known ports to identify well-known services. For example, every TCP/IP implementation that supports FTP assigns the well-known port of 21 (decimal) to the FTP server. Trivial File Transfer Protocol (TFTP) servers are assigned the UDP port of 69.

Clients, on the other hand, normally use **ephemeral ports**, that is, **short-lived ports**. These port numbers are normally assigned automatically by the transport protocol to the client. Clients normally do not care about the value of the ephemeral port; the client just needs to be certain that the ephemeral port is unique on the client host. The transport protocol code guarantees this **uniqueness**.

The Internet Assigned Numbers Authority (IANA) maintains a list of port number assignments. Assignments were once published as RFCs; RFC 1700 [Reynolds and Postel 1994] is the last in this series. RFC 3232 [Reynolds 2002] gives the location of the online database that replaced RFC 1700: [http://www.iana.org/](http://www.iana.org/). The port numbers are divided into three ranges:

1. The well-known ports: 0 through 1023. These port numbers are controlled and assigned by the IANA. When possible, the same port is assigned to a given service for TCP, UDP, and SCTP. For example, port 80 is assigned for a Web server, for both TCP and UDP, even though all implementations currently use only TCP.  
At the time that port 80 was assigned, SCTP did not yet exist. New port assignments are made for all three protocols, and RFC 2960 states that all existing TCP port numbers should be valid for the same service using SCTP.

2. The registered ports: 1024 through 49151. These are not controlled by the IANA, but the IANA registers and lists the uses of these ports as a convenience to the community. When possible, the same port is assigned to a given service for both TCP and UDP. For example, ports 6000 through 6063 are assigned for an X Window server for both protocols, even though all implementations currently use only TCP. The upper limit of 49151 for these ports was introduced to allow a range for ephemeral ports; RFC 1700 [Reynolds and Postel 1994] lists the upper range as 65535.

3. The dynamic or private ports, 49152 through 65535. The IANA says nothing about these ports. These are what we call ephemeral ports. (The magic number 49152 is three-fourths of 65536.)

***Figure 2.10*** shows this division, along with the common allocation of the port numbers.

#### Figure 2.10. Allocation of port numbers

We note the following points from this figure:

* Unix systems have the concept of a **reserved port**, which is any port less than 1024. These ports can only be assigned to a socket by an appropriately privileged process. All the IANA well-known ports are reserved ports; hence, the server allocating this port (such as the FTP server) must have superuser privileges when it starts.

* Historically, Berkeley-derived implementations (starting with 4.3BSD) have allocated ephemeral ports in the range 1024–5000. This was fine in the early 1980s, but it is easy today to find a host that can support more than 3977 connections at any given time. Therefore, many newer systems allocate ephemeral ports differently to provide more ephemeral ports, either using the IANA-defined ephemeral range or a larger range (e.g., Solaris as we show in ***Figure 2.10***).  
As it turns out, the upper limit of 5000 for the ephemeral ports, which many older systems implement, was a typo [Borman 1997a]. The limit should have been 50,000.

* There are a few clients (not servers) that require a reserved port as part of the client/server authentication: the rlogin and rsh clients are common examples. These clients call the library function rresvport to create a TCP socket and assign an unused port in the range 513–1023 to the socket. This function normally tries to bind port 1023, and if that fails, it tries to bind 1022, and so on, until it either succeeds or fails on port 513.  
Notice that the BSD reserved ports and the rresvport function both overlap with the upper half of the IANA well-known ports. This is because the IANA well-known ports used to stop at 255. RFC 1340 (a previous "Assigned Numbers" RFC) in 1992 started assigning well-known ports between 256 and 1023. The previous "Assigned Numbers" document, RFC 1060 in 1990, called ports 256–1023 the Unix Standard Services. There are numerous Berkeley-derived servers that picked their well-known ports in the 1980s starting at 512 (leaving 256–511 untouched). The rresvport function chose to start at the top of the 512–1023 range and work down.

#### Socket Pair

The socket pair for a TCP connection is the four-tuple that defines the two endpoints of the connection: the local IP address, local port, foreign IP address, and foreign port. A socket pair uniquely identifies every TCP connection on a network. For SCTP, an association is identified by a set of local IP addresses, a local port, a set of foreign IP addresses, and a foreign port. In its simplest form, where neither endpoint is multihomed, this results in the same four-tuple socket pair used with TCP. However, when either of the endpoints of an association are multihomed, then multiple four-tuple sets (with different IP addresses but the same port numbers) may identify the same association.

The two values that identify each endpoint, an IP address and a port number, are often called a **socket**.

We can extend the concept of a socket pair to UDP, even though UDP is connectionless. When we describe the socket functions (bind, connect, getpeername, etc.), we will note which functions specify which values in the socket pair. For example, bind lets the application specify the local IP address and local port for TCP, UDP, and SCTP sockets.

### 2.10 TCP Port Numbers and Concurrent Servers
