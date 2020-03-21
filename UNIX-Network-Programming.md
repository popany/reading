# UNIX Network Programming

- [UNIX Network Programming](#unix-network-programming)
  - [Chapter 1. Introduction](#chapter-1-introduction)
    - [1.1 Introduction](#11-introduction)
    - [1.2 A Simple Daytime Client](#12-a-simple-daytime-client)
      - [Figure 1.5 TCP daytime client](#figure-15-tcp-daytime-client)
    - [1.3 Protocol Independence](#13-protocol-independence)
    - [1.4 Error Handling: Wrapper Functions](#14-error-handling-wrapper-functions)
      - [Unix `errno` Value](#unix-errno-value)
    - [1.5 A Simple Daytime Server](#15-a-simple-daytime-server)
      - [Figure 1.9 TCP daytime server](#figure-19-tcp-daytime-server)
        - [Create a TCP socket](#create-a-tcp-socket)
        - [Bind server's well-known port to socket](#bind-servers-well-known-port-to-socket)
        - [Convert socket to listening socket](#convert-socket-to-listening-socket)
        - [Accept client connection, send reply](#accept-client-connection-send-reply)
        - [Terminate connection](#terminate-connection)
    - [1.6 Roadmap to Client/Server Examples in the Text](#16-roadmap-to-clientserver-examples-in-the-text)
    - [1.7 OSI Model](#17-osi-model)
      - [Figure 1.14. Layers in OSI model and Internet protocol suite](#figure-114-layers-in-osi-model-and-internet-protocol-suite)
    - [1.8 BSD Networking History](#18-bsd-networking-history)
      - [Figure 1.15. History of various BSD releases](#figure-115-history-of-various-bsd-releases)
    - [1.9 Test Networks and Hosts](#19-test-networks-and-hosts)
      - [Figure 1.16. Networks and hosts used for most examples in the text](#figure-116-networks-and-hosts-used-for-most-examples-in-the-text)
      - [Discovering Network Topology](#discovering-network-topology)
    - [1.10 Unix Standards](#110-unix-standards)
      - [Background on POSIX](#background-on-posix)
      - [Background on The Open Group](#background-on-the-open-group)
      - [Unification of Standards](#unification-of-standards)
      - [Internet Engineering Task Force (IETF)](#internet-engineering-task-force-ietf)
    - [1.11 64-Bit Architectures](#111-64-bit-architectures)
      - [Figure 1.17. Comparison of number of bits to hold various datatypes for the ILP32 and LP64 models](#figure-117-comparison-of-number-of-bits-to-hold-various-datatypes-for-the-ilp32-and-lp64-models)
    - [1.12 Summary](#112-summary)
  - [Chapter 2. The Transport Layer: TCP, UDP, and SCTP](#chapter-2-the-transport-layer-tcp-udp-and-sctp)
    - [2.1 Introduction](#21-introduction)
    - [2.2 The Big Picture](#22-the-big-picture)
      - [Figure 2.1. Overview of TCP/IP protocols](#figure-21-overview-of-tcpip-protocols)
    - [2.3 User Datagram Protocol (UDP)](#23-user-datagram-protocol-udp)
    - [2.4 Transmission Control Protocol (TCP)](#24-transmission-control-protocol-tcp)
    - [2.5 Stream Control Transmission Protocol (SCTP)](#25-stream-control-transmission-protocol-sctp)
    - [2.6 TCP Connection Establishment and Termination](#26-tcp-connection-establishment-and-termination)
      - [Three-Way Handshake](#three-way-handshake)
      - [Figure 2.2. TCP three-way handshake](#figure-22-tcp-three-way-handshake)
      - [TCP Options](#tcp-options)
      - [TCP Connection Termination](#tcp-connection-termination)
      - [Figure 2.3. Packets exchanged when a TCP connection is closed](#figure-23-packets-exchanged-when-a-tcp-connection-is-closed)
      - [TCP State Transition Diagram](#tcp-state-transition-diagram)
      - [Figure 2.4. TCP state transition diagram](#figure-24-tcp-state-transition-diagram)
      - [Watching the Packets](#watching-the-packets)
      - [Figure 2.5. Packet exchange for TCP connection](#figure-25-packet-exchange-for-tcp-connection)
    - [2.7 TIME_WAIT State](#27-timewait-state)
    - [2.8 SCTP Association Establishment and Termination](#28-sctp-association-establishment-and-termination)
      - [Four-Way Handshake](#four-way-handshake)
      - [Figure 2.6. SCTP four-way handshake](#figure-26-sctp-four-way-handshake)
      - [SCTP State Transition Diagram](#sctp-state-transition-diagram)
      - [Figure 2.8. SCTP state transition diagram](#figure-28-sctp-state-transition-diagram)
      - [Watching the Packets](#watching-the-packets-1)
      - [Figure 2.9. Packet exchange for SCTP association](#figure-29-packet-exchange-for-sctp-association)
      - [SCTP Options](#sctp-options)
    - [2.9 Port Numbers](#29-port-numbers)
      - [Figure 2.10. Allocation of port numbers](#figure-210-allocation-of-port-numbers)
      - [Socket Pair](#socket-pair)
    - [2.10 TCP Port Numbers and Concurrent Servers](#210-tcp-port-numbers-and-concurrent-servers)
      - [Figure 2.11. TCP server with a passive open on port 21](#figure-211-tcp-server-with-a-passive-open-on-port-21)
      - [Figure 2.12. Connection request from client to server](#figure-212-connection-request-from-client-to-server)
      - [Figure 2.13. Concurrent server has child handle client](#figure-213-concurrent-server-has-child-handle-client)
      - [Figure 2.14. Second client connection with same server](#figure-214-second-client-connection-with-same-server)
    - [2.11 Buffer Sizes and Limitations](#211-buffer-sizes-and-limitations)
      - [TCP Output](#tcp-output)
        - [Figure 2.15. Steps and buffers involved when an application writes to a TCP socket](#figure-215-steps-and-buffers-involved-when-an-application-writes-to-a-tcp-socket)
      - [UDP Output](#udp-output)
        - [Figure 2.16. Steps and buffers involved when an application writes to a UDP socket](#figure-216-steps-and-buffers-involved-when-an-application-writes-to-a-udp-socket)
      - [SCTP Output](#sctp-output)
        - [Figure 2.17. Steps and buffers involved when an application writes to an SCTP socket](#figure-217-steps-and-buffers-involved-when-an-application-writes-to-an-sctp-socket)
    - [2.12 Standard Internet Services](#212-standard-internet-services)
    - [2.13 Protocol Usage by Common Internet Applications](#213-protocol-usage-by-common-internet-applications)
    - [2.14 Summary](#214-summary)
  - [Chapter 3. Sockets Introduction](#chapter-3-sockets-introduction)
    - [3.1 Introduction](#31-introduction)
    - [3.2 Socket Address Structures](#32-socket-address-structures)
      - [IPv4 Socket Address Structure](#ipv4-socket-address-structure)
      - [Generic Socket Address Structure](#generic-socket-address-structure)
      - [IPv6 Socket Address Structure](#ipv6-socket-address-structure)
      - [Comparison of Socket Address Structures](#comparison-of-socket-address-structures)
    - [3.3 Value-Result Arguments](#33-value-result-arguments)
    - [3.4 Byte Ordering Functions](#34-byte-ordering-functions)
    - [3.5 Byte Manipulation Functions](#35-byte-manipulation-functions)
    - [3.6 inet_aton, inet_addr, and inet_ntoa Functions](#36-inetaton-inetaddr-and-inetntoa-functions)
    - [3.7 inet_pton and inet_ntop Functions](#37-inetpton-and-inetntop-functions)
    - [3.8 sock_ntop and Related Functions](#38-sockntop-and-related-functions)
    - [3.9 readn, writen, and readline Functions](#39-readn-writen-and-readline-functions)
    - [3.10 Summary](#310-summary)
  - [Chapter 4. Elementary TCP Sockets](#chapter-4-elementary-tcp-sockets)
    - [4.1 Introduction](#41-introduction)
    - [4.2 `socket` Function](#42-socket-function)
    - [4.3 `connect` Function](#43-connect-function)
    - [4.4 bind Function](#44-bind-function)
    - [4.5 listen Function](#45-listen-function)
    - [4.6 accept Function](#46-accept-function)
    - [4.7 fork and exec Functions](#47-fork-and-exec-functions)
    - [4.8 Concurrent Servers](#48-concurrent-servers)
    - [4.9 close Function](#49-close-function)
      - [Descriptor Reference Counts](#descriptor-reference-counts)
    - [4.10 getsockname and getpeername Functions](#410-getsockname-and-getpeername-functions)
    - [4.11 Summary](#411-summary)
  - [Chapter 5. TCP Client/Server Example](#chapter-5-tcp-clientserver-example)
    - [5.1 Introduction](#51-introduction)
    - [5.2 TCP Echo Server: main Function](#52-tcp-echo-server-main-function)
      - [Create socket, bind server's well-known port](#create-socket-bind-servers-well-known-port)
      - [Wait for client connection to complete](#wait-for-client-connection-to-complete)
      - [Concurrent server](#concurrent-server)
    - [5.3 TCP Echo Server: `str_echo` Function](#53-tcp-echo-server-strecho-function)
      - [Read a buffer and echo the buffer](#read-a-buffer-and-echo-the-buffer)
    - [5.4 TCP Echo Client: main Function](#54-tcp-echo-client-main-function)
      - [Create socket, fill in Internet socket address structure](#create-socket-fill-in-internet-socket-address-structure)
      - [Connect to server](#connect-to-server)
    - [5.5 TCP Echo Client: `str_cli` Function](#55-tcp-echo-client-strcli-function)
      - [Read a line, write to server](#read-a-line-write-to-server)
      - [Read echoed line from server, write to standard output](#read-echoed-line-from-server-write-to-standard-output)
      - [Return to `main`](#return-to-main)
    - [5.6 Normal Startup](#56-normal-startup)
    - [5.7 Normal Termination](#57-normal-termination)
    - [5.8 POSIX Signal Handling](#58-posix-signal-handling)
    - [POSIX Signal Semantics](#posix-signal-semantics)
    - [5.9 Handling SIGCHLD Signals](#59-handling-sigchld-signals)
      - [Handling Zombies](#handling-zombies)
      - [Handling Interrupted System Calls](#handling-interrupted-system-calls)
    - [5.10 wait and waitpid Functions](#510-wait-and-waitpid-functions)
    - [5.11 Connection Abort before accept Returns](#511-connection-abort-before-accept-returns)
    - [5.12 Termination of Server Process](#512-termination-of-server-process)
    - [5.13 SIGPIPE Signal](#513-sigpipe-signal)
    - [5.14 Crashing of Server Host](#514-crashing-of-server-host)
    - [5.15 Crashing and Rebooting of Server Host](#515-crashing-and-rebooting-of-server-host)
    - [5.16 Shutdown of Server Host](#516-shutdown-of-server-host)
    - [5.17 Summary of TCP Example](#517-summary-of-tcp-example)
    - [5.18 Data Format](#518-data-format)
    - [5.19 Summary](#519-summary)
  - [Chapter 6. I/O Multiplexing: The `select` and `poll` Functions](#chapter-6-io-multiplexing-the-select-and-poll-functions)
    - [6.1 Introduction](#61-introduction)
    - [6.2 I/O Models](#62-io-models)
      - [Blocking I/O Model](#blocking-io-model)
      - [Nonblocking I/O Model](#nonblocking-io-model)
      - [I/O Multiplexing Model](#io-multiplexing-model)
      - [Signal-Driven I/O Model](#signal-driven-io-model)
      - [Asynchronous I/O Model](#asynchronous-io-model)
      - [Comparison of the I/O Models](#comparison-of-the-io-models)
      - [Synchronous I/O versus Asynchronous I/O](#synchronous-io-versus-asynchronous-io)
    - [6.3 `select` Function](#63-select-function)

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

With a concurrent server, where the main server loop spawns a child to handle each new connection, what happens if the child continues to use the well-known port number while servicing a long request? Let's examine a typical sequence. First, the server is started on the host freebsd, which is multihomed with IP addresses 12.106.32.254 and 192.168.42.1, and the server does a passive open using its well-known port number (21, for this example). It is now waiting for a client request, which we show in ***Figure 2.11***.

#### Figure 2.11. TCP server with a passive open on port 21

We use the notation {*:21, *:*} to indicate the server's socket pair. The server is waiting for a connection request on any local interface (the first asterisk) on port 21. The foreign IP address and foreign port are not specified and we denote them as *:*. We also call this a listening socket.

We use a colon to separate the IP address from the port number because that is what HTTP uses and is commonly seen elsewhere. The netstat program uses a period to separate the IP address and port, but this is sometimes confusing because decimal points are used in both domain names (freebsd.unpbook.com.21) and in IPv4 dotted-decimal notation (12.106.32.254.21).

When we specify the local IP address as an asterisk, it is called the wildcard character. If the host on which the server is running is multihomed (as in this example), the server can specify that it wants only to accept incoming connections that arrive destined to one specific local interface. This is a **one-or-any** choice for the server. The server cannot specify a list of multiple addresses. The wildcard local address is the "any" choice. In ***Figure 1.9***, the wildcard address was specified by setting the IP address in the socket address structure to `INADDR_ANY` before calling `bind`.

At some later time, a client starts on the host with IP address 206.168.112.219 and executes an active open to the server's IP address of 12.106.32.254. We assume the ephemeral port chosen by the client TCP is 1500 for this example. This is shown in ***Figure 2.12***. Beneath the client we show its socket pair.

#### Figure 2.12. Connection request from client to server

When the server receives and accepts the client's connection, it forks a copy of itself, letting the child handle the client, as we show in ***Figure 2.13***. (We will describe the `fork` function in ***Section 4.7***.)

#### Figure 2.13. Concurrent server has child handle client

At this point, we must distinguish between the listening socket and the **connected socket** on the server host. Notice that the **connected socket** uses the same local port (21) as the listening socket. Also notice that on the multihomed server, the local address is filled in for the connected socket (12.106.32.254) once the connection is established.

The next step assumes that another client process on the client host requests a connection with the same server. The TCP code on the client host assigns the new client socket an unused ephemeral port number, say 1501. This gives us the scenario shown in ***Figure 2.14***. On the server, the two connections are distinct: the socket pair for the first connection differs from the socket pair for the second connection because the client's TCP chooses an unused port for the second connection (1501).

#### Figure 2.14. Second client connection with same server

Notice from this example that TCP cannot demultiplex incoming segments by looking at just the destination port number. TCP must look at all **four elements** in the socket pair to determine which endpoint receives an arriving segment. In ***Figure 2.14***, we have three sockets with the same local port (21). If a segment arrives from 206.168.112.219 port 1500 destined for 12.106.32.254 port 21, it is delivered to the first child. If a segment arrives from 206.168.112.219 port 1501 destined for 12.106.32.254 port 21, it is delivered to the second child. All other TCP segments destined for port 21 are delivered to the original server with the listening socket.

### 2.11 Buffer Sizes and Limitations

Certain limits affect the size of `IP datagrams`.

* The maximum size of an `IPv4 datagram` is `65,535` bytes, including the `IPv4 header`. This is because of the 16-bit **total length field** in ***Figure A.1***.

* The maximum size of an `IPv6 datagram` is `65,575` bytes, including the 40-byte `IPv6 header`. This is because of the 16-bit payload length field in ***Figure A.2***. Notice that the `IPv6 payload` length field does not include the size of the `IPv6 header`, while the `IPv4` total length field does include the header size.  
`IPv6` has a **jumbo payload** option, which extends the payload length field to `32` bits, but this option is supported only on datalinks with a **maximum transmission unit* (`MTU`) that exceeds `65,535`. (This is intended for host-to-host interconnects, such as `HIPPI`, which often have no inherent `MTU`.)

* Many networks have an `MTU` which can be **dictated by the hardware**. For example, the `Ethernet MTU` is `1,500` bytes. Other datalinks, such as point-to-point links using the Point-to-Point Protocol (`PPP`), have a configurable `MTU`. Older `SLIP` links often used an `MTU` of `1,006` or `296` bytes.  
The **minimum** link `MTU` for `IPv4` is `68` bytes. This permits a **maximum-sized** `IPv4 header` (`20` bytes of **fixed** header, `30` bytes of **options**) and minimum-sized fragment (the fragment offset is in units of 8 bytes). The minimum link `MTU` for `IPv6` is `1,280` bytes. `IPv6` can run over links with a smaller `MTU`, but requires link-specific fragmentation and reassembly to make the link appear to have an `MTU` of at least `1,280` bytes (RFC 2460 [Deering and Hinden 1998]).

* The smallest `MTU` in the path between two hosts is called the **path `MTU`**. Today, the `Ethernet MTU` of `1,500 bytes` is often the path `MTU`. The path `MTU` **need not be the same** in both directions between any two hosts because routing in the Internet is often **asymmetric** [Paxson 1996]. That is, the route from `A` to `B` can differ from the route from `B` to `A`.

* When an `IP` datagram is to be sent out an interface, if the size of the datagram exceeds the link `MTU`, **fragmentation** is performed by both `IPv4` and `IPv6`. The fragments are not normally **reassembled** until they reach the **final destination**. IPv4 hosts perform fragmentation on datagrams that they **generate** and IPv4 routers perform fragmentation on datagrams that they **forward**. But with `IPv6`, only hosts perform fragmentation on datagrams that they **generate**; IPv6 routers do not fragment datagrams that they are forwarding.  
We must be careful with our terminology. A box labeled as an `IPv6` router may indeed perform fragmentation, but only on datagrams that the router itself generates, never on datagrams that it is forwarding. When this box generates `IPv6` datagrams, it is really acting as a host. For example, most routers support the Telnet protocol and this is used for router configuration by administrators. The IP datagrams generated by the router's Telnet server are generated by the router, not forwarded by the router.  
You may notice that fields exist in the `IPv4 header` (***Figure A.1***) to handle `IPv4` fragmentation, but there are no fields in the `IPv6 header` (***Figure A.2***) for fragmentation. Since fragmentation is the exception, rather than the rule, `IPv6` contains an option header with the fragmentation information.  
Certain firewalls, which usually act as routers, may reassemble fragmented packets to allow inspection of the entire packet contents. This allows the prevention of certain attacks at the cost of additional complexity in the firewall device. It also requires the firewall device to be part of the only path to the network, reducing the opportunities for redundancy.

* If the "don't fragment" (**`DF`**) bit is set in the `IPv4` header (**Figure A.1**), it specifies that this datagram must not be fragmented, either by the sending host or by any router. A router that receives an `IPv4` datagram with the `DF` bit set whose size exceeds the outgoing link's `MTU` generates an `ICMPv4` "destination **unreachable**, fragmentation needed but `DF` bit set" error message (***Figure A.15***).  
Since `IPv6` routers do not perform fragmentation, there is an implied `DF` bit with every `IPv6` datagram. When an `IPv6` router receives a datagram whose size exceeds the outgoing link's `MTU`, it generates an `ICMPv6` "packet too big" error message (***Figure A.16***).  
The `IPv4` `DF` bit and its implied `IPv6` counterpart can be used for path `MTU` discovery (RFC 1191 [Mogul and Deering 1990] for `IPv4` and RFC 1981 [McCann, Deering, and Mogul 1996] for IPv6). For example, if `TCP` uses this technique with `IPv4`, then it sends all its datagrams with the `DF` bit set. If some intermediate router returns an `ICMP` "destination unreachable, fragmentation needed but `DF` bit set" error, `TCP` decreases the amount of data it sends per datagram and retransmits. Path `MTU` discovery is optional with `IPv4`, but `IPv6` implementations all either support path `MTU` discovery or always send using the minimum `MTU`.  
Path MTU discovery is problematic in the Internet today; many firewalls drop all `ICMP` messages, including the fragmentation required message, meaning that `TCP` never gets the signal that it needs to decrease the amount of data it is sending. As of this writing, an effort is beginning in the `IETF` to define another method for path `MTU` discovery that does not rely on `ICMP` errors.

* `IPv4` and `IPv6` define a minimum reassembly buffer size, the minimum datagram size that we are guaranteed any implementation must support. For `IPv4`, this is `576` bytes. `IPv6` raises this to `1,500` bytes. With `IPv4`, for example, we have no idea whether a given destination can accept a 577-byte datagram or not. Therefore, many `IPv4` applications that use `UDP` (e.g., DNS, RIP, TFTP, BOOTP, SNMP) prevent applications from generating IP datagrams that exceed this size.

* `TCP` has a maximum segment size (`MSS`) that announces to the peer `TCP` the maximum amount of `TCP` data that the peer can send per segment. We saw the `MSS` option on the `SYN` segments in ***Figure 2.5***. The goal of the `MSS` is to tell the peer the actual value of the reassembly buffer size and to try to avoid fragmentation. The `MSS` is often set to the interface `MTU` minus the fixed sizes of the `IP` and `TCP` headers. On an Ethernet using `IPv4`, this would be `1,460`, and on an Ethernet using `IPv6`, this would be `1,440`. (The `TCP` header is `20` bytes for both, but the `IPv4` header is `20` bytes and the `IPv6` header is 40 bytes.)  
The `MSS` value in the TCP MSS option is a 16-bit field, limiting the value to `65,535`. This is fine for `IPv4`, since the maximum amount of TCP data in an IPv4 datagram is `65,495` (65,535 minus the 20-byte IPv4 header and minus the 20-byte TCP header). But with the IPv6 jumbo payload option, a different technique is used (RFC 2675 [Borman, Deering, and Hinden 1999]). First, the maximum amount of TCP data in an IPv6 datagram without the jumbo payload option is 65,515 (65,535 minus the 20-byte TCP header). Therefore, the MSS value of 65,535 is considered a special case that **designates "infinity".** This value is used only if the jumbo payload option is being used, which requires an MTU that exceeds 65,535. If TCP is using the jumbo payload option and receives an MSS announcement of 65,535 from the peer, the limit on the datagram sizes that it sends is just the **interface MTU**. If this turns out to be too large (i.e., there is a link in the path with a smaller MTU), then path MTU discovery will determine the smaller value.

* `SCTP` keeps a fragmentation point based on the smallest path MTU found to all the peer's addresses. This smallest MTU size is used to split large user messages into smaller pieces that can be sent in one IP datagram. The `SCTP_MAXSEG` socket option can influence this value, allowing the user to request a smaller fragmentation point.

#### TCP Output

Given all these terms and definitions, ***Figure 2.15*** shows what happens when an application writes data to a `TCP` socket.

##### Figure 2.15. Steps and buffers involved when an application writes to a TCP socket

Every TCP socket has a **send buffer** and we can change the size of this buffer with the `SO_SNDBUF` socket option (***Section 7.5***). When an application calls write, the kernel copies all the data from the application buffer into the socket send buffer. If there is insufficient room in the socket buffer for **all** the application's data (either the application buffer is larger than the socket send buffer, or there is already data in the socket send buffer), the process is put to sleep. This assumes the normal default of a blocking socket. (We will talk about nonblocking sockets in ***Chapter 16***.) The kernel will not return from the write until the **final** byte in the application buffer has been copied into the socket send buffer. Therefore, **the successful return from a write to a TCP socket only tells us that we can reuse our application buffer. It does not tell us that either the peer TCP has received the data or that the peer application has received the data**. (We will talk about this more with the `SO_LINGER` socket option in ***Section 7.5***.)

TCP takes the data in the socket send buffer and sends it to the peer TCP based on all the rules of TCP data transmission (***Chapter 19 and 20*** of TCPv1). The peer TCP must acknowledge the data, and as the ACKs arrive from the peer, only then can our TCP discard the acknowledged data from the socket send buffer. **TCP must keep a copy of our data until it is acknowledged by the peer**.

TCP sends the data to IP in **MSS-sized or smaller chunks**, prepending its TCP header to each segment, where the MSS is the value announced by the peer, or 536 if the peer did not send an MSS option. IP prepends its header, searches the routing table for the destination IP address (the matching routing table entry specifies the outgoing interface), and passes the datagram to the appropriate datalink. IP might perform fragmentation before passing the datagram to the datalink, but as we said earlier, one goal of the MSS option is to try to avoid fragmentation and newer implementations also use path MTU discovery. Each datalink has an **output queue**, and if this queue is full, the packet is **discarded** and an error is returned up the protocol stack: from the datalink to IP and then from IP to TCP. TCP will note this error and try sending the segment later. **The application is not told of this transient condition**.

#### UDP Output

***Figure 2.16*** shows what happens when an application writes data to a `UDP` socket.

##### Figure 2.16. Steps and buffers involved when an application writes to a UDP socket

This time, we show the socket send buffer as a dashed box because it doesn't really exist. A UDP socket has a send buffer size (which we can change with the `SO_SNDBUF` socket option, ***Section 7.5***), but this is simply an **upper limit** on the maximum-sized UDP datagram that can be written to the socket. If an application writes a datagram larger than the socket send buffer size, `EMSGSIZE` is returned. Since UDP is unreliable, it does not need to keep a copy of the application's data and **does not need an actual send buffer**. (The application data is normally copied into a **kernel buffer** of some form as it passes down the protocol stack, but this copy is discarded by the datalink layer after the data is transmitted.)

UDP simply prepends its 8-byte header and passes the datagram to IP. IPv4 or IPv6 prepends its header, determines the outgoing interface by performing the routing function, and then either adds the datagram to the datalink **output queue** (if it fits within the MTU) or fragments the datagram and adds each fragment to the datalink output queue. If a UDP application sends large datagrams (say 2,000-byte datagrams), there is a **much higher probability of fragmentation than with TCP**, because TCP breaks the application data into MSS-sized chunks, something that has no counterpart in UDP.

The successful return from a write to a UDP socket tells us that either the datagram or all fragments of the datagram have been added to the datalink output queue. If there is no room on the queue for the datagram or one of its fragments, **`ENOBUFS`** is often returned to the application.

Unfortunately, **some implementations do not return this error**, giving the application no indication that the datagram was discarded without even being transmitted.

#### SCTP Output

***Figure 2.17*** shows what happens when an application writes data to an SCTP socket.

##### Figure 2.17. Steps and buffers involved when an application writes to an SCTP socket

SCTP, since it is a reliable protocol like TCP, has a send buffer. As with TCP, an application can change the size of this buffer with the `SO_SNDBUF` socket option (***Section 7.5***). When the application calls write, the kernel copies all the data from the application buffer into the socket send buffer. If there is insufficient room in the socket buffer for all of the application's data (either the application buffer is larger than the socket send buffer, or there is already data in the socket send buffer), the process is put to sleep. This sleeping assumes the normal default of a blocking socket. (We will talk about nonblocking sockets in ***Chapter 16***.) The kernel will not return from the write until the final byte in the application buffer has been copied into the socket send buffer. Therefore, **the successful return from a write to an SCTP socket only tells the sender that it can reuse the application buffer. It does not tell us that either the peer SCTP has received the data, or that the peer application has received the data**.

SCTP takes the data in the socket send buffer and sends it to the peer SCTP based on all the rules of SCTP data transmission (for details of data transfer, see ***Chapter 5*** of [Stewart and Xie 2001]). The sending SCTP must await a SACK in which the cumulative acknowledgment point passes the sent data before that data can be removed from the socket buffer.

### 2.12 Standard Internet Services

### 2.13 Protocol Usage by Common Internet Applications

### 2.14 Summary

## Chapter 3. Sockets Introduction

### 3.1 Introduction

### 3.2 Socket Address Structures

Most socket functions require a pointer to a socket address structure as an argument. Each supported protocol suite defines its own socket address structure. The names of these structures begin with `sockaddr_` and end with a unique suffix for **each protocol** suite.

#### IPv4 Socket Address Structure

#### Generic Socket Address Structure

#### IPv6 Socket Address Structure

#### Comparison of Socket Address Structures

### 3.3 Value-Result Arguments

### 3.4 Byte Ordering Functions

### 3.5 Byte Manipulation Functions

### 3.6 inet_aton, inet_addr, and inet_ntoa Functions

### 3.7 inet_pton and inet_ntop Functions

### 3.8 sock_ntop and Related Functions

### 3.9 readn, writen, and readline Functions

### 3.10 Summary

## Chapter 4. Elementary TCP Sockets

### 4.1 Introduction

This chapter describes the elementary socket functions required to write a complete TCP client and server.

### 4.2 `socket` Function

To perform network I/O, the first thing a process must do is call the `socket` function, specifying the type of communication protocol desired (TCP using IPv4, UDP using IPv6, Unix domain stream protocol, etc.).

### 4.3 `connect` Function

The `connect` function is **used by a TCP client** to establish a connection with a TCP server.

The client does **not have to call** `bind` (which we will describe in the next section) before calling `connect`: the kernel will choose both an ephemeral port and the source IP address if necessary.

In the case of a TCP `socket`, the `connect` function initiates TCP's **three-way handshake** (***Section 2.6***). The function returns only when the **connection is established** or an **error occurs**. There are several different error returns possible.

1. If the client TCP receives no response to its **`SYN`** segment, `ETIMEDOUT` is returned. 4.4BSD, for example, sends one `SYN` when connect is called, another **`6` seconds** later, and another **`24` seconds** later (p. 828 of TCPv2). If no response is received after a total of **`75` seconds**, the error is returned.

    Some systems provide administrative control over this timeout; see Appendix E of TCPv1.

2. If the server's response to the client's `SYN` is a reset (**`RST`**), this indicates that **no process is waiting** for connections on the server host at the port specified (i.e., the server process is probably not running). This is a **hard error** and the error `ECONNREFUSED` is returned to the client as soon as the `RST` is received.

    An **`RST`** is a type of **TCP segment** that is sent by TCP when something is wrong. Three conditions that generate an `RST` are: when a `SYN` arrives for a port that has **no listening server** (what we just described), when TCP wants to **abort an existing connection**, and when TCP receives a segment for a connection that does **not exist**. (TCPv1 [pp. 246–250] contains additional information.)

3. If the client's `SYN` elicits an `ICMP` "destination unreachable" from some intermediate router, this is considered a **soft error**. The client kernel saves the message but **keeps sending SYNs** with the same **time between each `SYN`** as in the first scenario. If no response is received after some fixed amount of time (**`75` seconds** for 4.4BSD), the saved `ICMP` error is returned to the process as either `EHOSTUNREACH` or `ENETUNREACH`. It is also possible that the remote system is not reachable by any route in the local system's forwarding table, or that the connect call returns without waiting at all.

    Many earlier systems, such as 4.2BSD, incorrectly aborted the connection establishment attempt when the ICMP "destination unreachable" was received. This is wrong because this `ICMP` error can indicate a transient condition. For example, it could be that the condition is caused by a routing problem that will be corrected.

    Notice that `ENETUNREACH` is not listed in ***Figure A.15***, even when the error indicates that the destination network is unreachable. Network unreachables are considered obsolete, and applications should just treat `ENETUNREACH` and `EHOSTUNREACH` as the same error.

In terms of the TCP state transition diagram (***Figure 2.4***), connect moves from the `CLOSED` state (the state in which a socket begins when it is created by the socket function) to the `SYN_SENT` state, and then, on success, to the `ESTABLISHED` state. **If connect fails, the socket is no longer usable and must be closed. We cannot call connect again on the socket**. In ***Figure 11.10***, we will see that when we call connect in a loop, trying each IP address for a given host until one works, **each time connect fails, we must close the socket descriptor and call socket again**.

### 4.4 bind Function

The `bind` function assigns a local protocol address to a socket. With the Internet protocols, the protocol address is the combination of either a 32-bit IPv4 **address** or a 128-bit IPv6 address, along with a 16-bit TCP or UDP **port** number.

* Servers `bind` their well-known port when they start. We saw this in ***Figure 1.9***. If a TCP client or server does not do this, the kernel chooses an **ephemeral port** for the socket when either `connect` or `listen` is called. It is normal for a TCP client to let the kernel choose an ephemeral port, unless the application requires a reserved port (***Figure 2.10***), but it is rare for a TCP server to let the kernel choose an ephemeral port, since servers are known by their well-known port.

    Exceptions to this rule are Remote Procedure Call (**RPC**) servers. They normally let the kernel choose an ephemeral port for their listening socket since this port is then registered with the RPC port mapper. Clients have to contact the port mapper to obtain the ephemeral port before they can connect to the server. This also applies to RPC servers using UDP.

* A process can `bind` a specific IP address to its socket. The IP address must belong to an interface on the host. For a TCP client, this assigns the source IP address that will be used for IP datagrams sent on the socket. For a TCP server, this restricts the socket to receive incoming client connections destined only to that IP address.

    Normally, a TCP client does not `bind` an IP address to its socket. The kernel chooses the source IP address when the socket is connected, based on the outgoing interface that is used, which in turn is based on the route required to reach the server (p. 737 of TCPv2).

    If a TCP server does not `bind` an IP address to its socket, the kernel uses the destination IP address of the client's `SYN` as the server's source IP address (p. 943 of TCPv2).

### 4.5 listen Function

The `listen` function is called only by a **TCP server** and it performs two actions:

1. When a socket is created by the `socket` function, it is assumed to be an active socket, that is, a client socket that will issue a connect. The `listen` function converts an **unconnected socket** into a **passive socket**, indicating that the kernel should `accept` incoming connection requests directed to this socket. In terms of the TCP state transition diagram (***Figure 2.4***), the call to `listen` moves the socket from the `CLOSED` state to the `LISTEN` state.

2. The second argument to this function specifies the maximum number of connections the kernel should **queue** for this socket.

This function is normally called after both the socket and `bind` functions and must be called before calling the `accept` function.

    #include <sys/socket.h>
    int listen (int sockfd, int backlog);

To understand the `backlog` argument, we must realize that for a given listening socket, the kernel maintains two queues:

1. An **incomplete connection queue**, which contains an entry for each `SYN` that has arrived from a client for which the server is awaiting completion of the TCP three-way handshake. These sockets are in the `SYN_RCVD` state (***Figure 2.4***).

2. A **completed connection queue**, which contains an entry for each client with whom the TCP three-way handshake has completed. These sockets are in the `ESTABLISHED` state (***Figure 2.4***).

When an entry is created on the incomplete queue, the parameters from the listen socket are copied over to the newly created connection. The connection creation mechanism is completely automatic; the server process is not involved. ***Figure 4.8*** depicts the packets exchanged during the connection establishment with these two queues.

When a `SYN` arrives from a client, TCP **creates a new entry** on the **incomplete queue** and then **responds** with the second segment of the three-way handshake: the server's `SYN` with an `ACK` of the client's `SYN` (***Section 2.6***). This entry will remain on the incomplete queue until the third segment of the three-way handshake **arrives** (the client's `ACK` of the server's `SYN`), or until the entry **times out**. (Berkeley-derived implementations have a timeout of `75 seconds` for these incomplete entries.) If the three-way handshake completes normally, the entry **moves** from the incomplete queue to the end of the **completed queue**. When the process calls `accept`, which we will describe in the next section, the first entry on the completed queue is **returned to the process**, or if the queue is empty, the process is put to **sleep** until an entry is placed onto the completed queue.

There are several points to consider regarding the handling of these two queues.

* The backlog argument to the `listen` function has historically specified the maximum value for the sum of both queues.

* Berkeley-derived implementations add a fudge factor to the backlog: It is multiplied by 1.5 (p. 257 of TCPv1 and p. 462 of TCPV2). For example, the commonly specified backlog of 5 really allows up to 8 queued entries on these systems, as we show in ***Figure 4.10***.

* **Do not specify a backlog of `0`**, as different implementations interpret this differently (***Figure 4.10***). **If you do not want any clients connecting to your listening socket, close the listening socket**.

* Assuming the three-way handshake completes normally (i.e., no lost segments and no retransmissions), an entry **remains** on the **incomplete connection queue** for **one `RTT`**, whatever that value happens to be between a particular client and server. ***Section 14.4*** of TCPv3 shows that for one Web server, the median `RTT` between many clients and the server was `187` ms. (The median is often used for this statistic, since a few large values can noticeably skew the mean.)

* Historically, sample code always shows a `backlog` of 5, as that was the maximum value supported by 4.2BSD. This was adequate in the 1980s when busy servers would handle only a few hundred connections per day. But with the growth of the World Wide Web (WWW), where busy servers handle millions of connections per day, this small number is completely inadequate (pp. 187–192 of TCPv3). Busy HTTP servers must specify a much larger backlog, and newer kernels must support larger values.

* A problem is: What value should the application specify for the backlog, since 5 is often inadequate? There is no easy answer to this. HTTP servers now specify a larger value, but if the value specified is a constant in the source code, to increase the constant requires recompiling the server. Another method is to assume some default but allow a command-line option or an environment variable to override the default. It is always acceptable to specify a value that is larger than supported by the kernel, as the kernel should silently truncate the value to the maximum value that it supports, without returning an error (p. 456 of TCPv2).

* Manuals and books have historically said that the reason for queuing a fixed number of connections is to handle the case of the server process being busy between successive calls to `accept`. This implies that of the two queues, the completed queue should normally have more entries than the incomplete queue. Again, busy Web servers have shown that this is false. The reason for specifying a large backlog is because the incomplete connection queue can grow as client SYNs arrive, waiting for completion of the three-way handshake.

* If the queues are **full** when a client `SYN` arrives, TCP **ignores** the arriving `SYN` (pp. 930–931 of TCPv2); it does not send an `RST`. This is because the condition is considered temporary, and the client TCP will **retransmit** its `SYN`, hopefully finding room on the queue in the near future. If the server TCP immediately responded with an `RST`, the client's connect would return an error, forcing the application to handle this condition instead of letting TCP's normal retransmission take over. Also, the client could not differentiate between an `RST` in response to a SYN meaning "there is no server at this port" versus "there is a server at this port but its queues are full."

* Data that arrives after the three-way handshake completes, but before the server calls accept, should be queued by the server TCP, up to the size of the connected socket's receive buffer.

### 4.6 accept Function

`accept` is called by a TCP server to return the next completed connection from the front of the **completed connection queue** (***Figure 4.7***). If the completed connection queue is empty, the process is put to **sleep** (assuming the default of a **blocking socket**).

If `accept` is successful, its return value is a **brand-new descriptor** automatically created by the kernel. This new descriptor refers to the TCP connection with the client. When discussing `accept`, we call the first argument to `accept` the **listening socket** (the descriptor created by socket and then used as the first argument to both `bind` and `listen`), and we call the return value from `accept` the **connected socket**. It is important to differentiate between these two sockets. A given server normally creates only one listening socket, which then exists for the lifetime of the server. The kernel creates one connected socket for each client connection that is accepted (i.e., for which the TCP three-way handshake completes). **When the server is finished serving a given client, the connected socket is closed**.

### 4.7 fork and exec Functions

Before describing how to write a concurrent server in the next section, we must describe the Unix `fork` function. This function (including the variants of it provided by some systems) is the only way in Unix to create a new process.

All descriptors open in the parent before the call to `fork` are shared with the child after `fork` returns. We will see this feature used by network servers: The parent calls `accept` and then calls `fork`. The connected socket is then shared between the parent and child. Normally, the child then reads and writes the connected socket and the parent closes the connected socket.

There are two typical uses of `fork`:

1. A process makes a copy of itself so that one copy can handle one operation while the other copy does another task. This is typical for network servers. We will see many examples of this later in the text.

2. A process wants to execute another program. Since the only way to create a new process is by calling `fork`, the process first calls `fork` to make a copy of itself, and then one of the copies (typically the child process) calls `exec` (described next) to replace itself with the new program. This is typical for programs such as shells.

The only way in which an executable program file on disk can be executed by Unix is for an existing process to call one of the **six `exec` functions**. (We will often refer generically to "the exec function" when it does not matter which of the six is called.) `exec` replaces the current **process image** with the new program file, and this new program normally starts at the main function. The **process ID does not change**. We refer to the process that calls exec as the **calling process** and the newly executed program as the **new program**.

These functions return to the caller only if an error occurs. Otherwise, control passes to the start of the new program, normally the `main` function.

### 4.8 Concurrent Servers

The server in ***Figure 4.11*** is an iterative server. For something as simple as a daytime server, this is fine. But when a client request can **take longer to service**, we do not want to tie up a single server with one client; we want to **handle multiple clients at the same time**. The simplest way to write a concurrent server under Unix is to `fork` a child process to handle each client. ***Figure 4.13*** shows the outline for a typical concurrent server.

We said in ***Section 2.6*** that calling `close` on a TCP socket causes a `FIN` to be sent, followed by the normal TCP connection termination sequence. Why doesn't the `close` of `connfd` in ***Figure 4.13*** by the parent terminate its connection with the client? To understand what's happening, we must understand that every file or socket has a **reference count**. The reference count is maintained in the **file table** entry (pp. 57–60 of APUE). This is a count of the number of descriptors that are currently open that refer to this file or socket. In ***Figure 4.13***, after socket returns, the file table entry associated with `listenfd` has a reference count of `1`. After `accept` returns, the file table entry associated with `connfd` has a reference count of `1`. But, after fork returns, both descriptors are shared (i.e., duplicated) between the parent and child, so the file table entries associated with both sockets now have a reference count of `2`. Therefore, when the parent closes `connfd`, it just **decrements the reference count** from `2` to `1` and that is all. The actual **cleanup and de-allocation** of the socket does not happen until the reference count reaches `0`. This will occur at some time later when the child closes `connfd`.

### 4.9 close Function

The normal Unix `close` function is also used to close a socket and terminate a TCP connection.

The default action of `close` with a TCP socket is to **mark the socket as closed** and return to the process immediately. The socket descriptor is no longer usable by the process: It **cannot be used as an argument to `read` or `write`**. But, TCP will try to send any data that is already **queued** to be sent to the other end, and after this occurs, the normal TCP connection **termination sequence** takes place (***Section 2.6***).

In ***Section 7.5***, we will describe the `SO_LINGER` socket option, which lets us change this default action with a TCP socket. In that section, we will also describe what a TCP application must do to be guaranteed that the peer application has received any outstanding data.

#### Descriptor Reference Counts

At the end of ***Section 4.8***, we mentioned that when the parent process in our concurrent server closes the connected socket, this **just decrements the reference count** for the descriptor. Since the reference count was still greater than `0`, this call to close did not initiate TCP's **four-packet connection termination** sequence. This is the behavior we want with our concurrent server with the connected socket that is shared between the parent and child.

If we really want to send a `FIN` on a TCP connection, the `shutdown` function can be used (***Section 6.6***) instead of `close`. We will describe the motivation for this in ***Section 6.5***.

We must also be aware of what happens in our concurrent server if the parent does not call `close` for each connected socket returned by `accept`. First, the parent will eventually **run out of descriptors**, as there is usually a limit to the number of descriptors that any process can have open at any time. But more importantly, **none of the client connections will be terminated**. When the child closes the connected socket, its reference count will go from `2` to `1` and it will remain at `1` since the parent never closes the connected socket. This will prevent TCP's connection termination sequence from occurring, and the connection will remain open.

### 4.10 getsockname and getpeername Functions

These two functions return either the local protocol address associated with a socket (`getsockname`) or the foreign protocol address associated with a socket (`getpeername`).

### 4.11 Summary

All clients and servers begin with a call to `socket`, returning a socket descriptor. Clients then call `connect`, while servers call `bind`, `listen`, and `accept`. Sockets are normally closed with the standard `close` function, although we will see another way to do this with the `shutdown` function (***Section 6.6***), and we will also examine the effect of the `SO_LINGER` socket option (***Section 7.5***).

**Most TCP servers are concurrent**, with the server calling `fork` for every client connection that it handles. We will see that most **UDP servers are iterative**. While these two models have been used successfully for many years, in ***Chapter 30*** we will look at other server design options that use threads and processes.

## Chapter 5. TCP Client/Server Example

### 5.1 Introduction

We will now use the elementary functions from the previous chapter to write a complete TCP client/server example. Our simple example is an echo server that performs the following steps:

1. The client reads a line of text from its standard input and writes the line to the server.

2. The server reads the line from its network input and echoes the line back to the client.

3. The client reads the echoed line and prints it on its standard output

### 5.2 TCP Echo Server: main Function

#### Create socket, bind server's well-known port

#### Wait for client connection to complete

#### Concurrent server

### 5.3 TCP Echo Server: `str_echo` Function

#### Read a buffer and echo the buffer

### 5.4 TCP Echo Client: main Function

#### Create socket, fill in Internet socket address structure

#### Connect to server

### 5.5 TCP Echo Client: `str_cli` Function

#### Read a line, write to server

#### Read echoed line from server, write to standard output

#### Return to `main`

### 5.6 Normal Startup

Although our TCP example is small (about 150 lines of code for the two main functions, str_echo, str_cli, readline, and writen), it is essential that we understand how the client and server start, how they end, and most importantly, what happens when something goes wrong: the **client host crashes**, the **client process crashes**, **network connectivity is lost**, and so on. Only by understanding these **boundary conditions**, and their interaction with the TCP/IP protocols, can we write robust clients and servers that can handle these conditions.

When the three-way handshake completes, we purposely list the client step first, and then the server steps. The reason can be seen in ***Figure 2.5***: **`connect` returns when the second segment of the handshake is received by the client**, but **`accept` does not return until the third segment of the handshake is received by the server**, one-half of the RTT after `connect` returns.

### 5.7 Normal Termination

We can follow through the steps involved in the normal termination of our client and server:

1. When we type our `EOF` character, `fgets` returns a null pointer and the function `str_cli` (***Figure 5.5***) returns.

2. When `str_cli` returns to the client `main` function (***Figure 5.4***), the latter terminates by calling `exit`.

3. Part of process termination is the closing of all open descriptors, so the client socket is **closed by the kernel**. This **sends a `FIN` to the server**, to which the **server TCP responds with an `ACK`**. This is the first half of the TCP connection termination sequence. At this point, the server socket is in the `CLOSE_WAIT` state and the client socket is in the `FIN_WAIT_2` state (***Figures 2.4*** and ***2.5***).

4. When the **server TCP receives the `FIN`**, the server child is blocked in a call to `readline` (***Figure 5.3***), and **`readline` then returns `0`**. This causes the `str_echo` function to return to the server child `main`.

5. The server child terminates by calling `exit` (***Figure 5.2***).

6. All open descriptors in the server child are closed. The closing of the connected socket by the child causes the final two segments of the TCP connection termination to take place: a **`FIN` from the server to the client**, and an **`ACK` from the client** (***Figure 2.5***). At this point, the **connection is completely terminated**. The client socket enters the `TIME_WAIT` state.

7. Finally, the `SIGCHLD` signal is sent to the parent when the server child terminates. This occurs in this example, but we do not catch the signal in our code, and the default action of the signal is to be ignored. Thus, the child enters the zombie state. We can verify this with the ps command.

### 5.8 POSIX Signal Handling

A signal is a notification to a process that an event has occurred. Signals are sometimes called **software interrupts**. Signals usually occur **asynchronously**. By this we mean that a process doesn't know ahead of time exactly when a signal will occur.

Signals can be sent

* By one process to another process (or to itself)

* By the kernel to a process

The `SIGCHLD` signal that we described at the end of the previous section is one that is **sent by the kernel** whenever a **process terminates**, to the parent of the terminating process.

Every signal has a **disposition**, which is also called the **action associated with the signal**. We set the disposition of a signal by calling the `sigaction` function (described shortly) and we have three choices for the disposition:

1. We can provide a function that is called whenever a specific signal occurs. This function is called a **signal handler** and this action is called **catching a signal**. The two signals **`SIGKILL` and `SIGSTOP` cannot be caught**. Our function is called with a single integer argument that is the signal number and the function returns nothing. Its function prototype is therefore

    `void handler (int signo);`

    For most signals, calling sigaction and specifying a function to be called when the signal occurs is all that is required to catch a signal. But we will see later that a few signals, `SIGIO`, `SIGPOLL`, and `SIGURG`, all **require additional actions** on the part of the process to catch the signal.

2. We can **ignore a signal** by setting its disposition to `SIG_IGN`. The two signals **`SIGKILL` and `SIGSTOP` cannot be ignored**.

3. We can set the **default disposition** for a signal by setting its disposition to `SIG_DFL`. The default is normally to terminate a process on receipt of a signal, with certain signals also generating a core image of the process in its current working directory. There are a few signals whose default disposition is **to be ignored**: `SIGCHLD` and `SIGURG` (sent on the arrival of out-of-band data, **Chapter 24**) are two that we will encounter in this text.

### POSIX Signal Semantics

We summarize the following points about signal handling on a POSIX-compliant system:

* Once a signal handler is installed, it **remains installed**. (Older systems removed the signal handler each time it was executed.)

* While a signal handler is executing, the signal being delivered is **blocked**. Furthermore, any additional signals that were specified in the `sa_mask` signal set passed to `sigaction` when the handler was installed are **also blocked**. In ***Figure 5.6***, we set `sa_mask` to the empty set, meaning no additional signals are blocked other than the signal being caught.

* If a signal is generated one or more times while it is blocked, it is normally **delivered only one time** after the signal is unblocked. That is, by default, Unix **signals are not queued**. We will see an example of this in the next section. The POSIX real-time standard, `1003.1b`, defines some reliable signals that are queued, but we do not use them in this text.

* It is possible to selectively block and unblock a set of signals using the `sigprocmask` function. This lets us **protect a critical region** of code by preventing certain signals from being caught while that region of code is executing.

### 5.9 Handling SIGCHLD Signals

The purpose of the zombie state is to maintain **information about the child** for the parent to fetch at some later time. This information includes the **process ID** of the child, its **termination status**, and information on the **resource utilization** of the child (CPU time, memory, etc.). If a process terminates, and that process has children in the zombie state, the **parent process ID of all the zombie children is set to 1** (the init process), which will **inherit the children and clean them up** (i.e., init will wait for them, which removes the zombie). Some Unix systems show the COMMAND column for a zombie process as `<defunct>`.

#### Handling Zombies

Obviously we do not want to leave zombies around. They **take up space in the kernel** and eventually we can **run out of processes**. Whenever we fork children, we must wait for them to prevent them from becoming zombies. To do this, we establish a signal handler to catch `SIGCHLD`, and within the handler, we call `wait`. (We will describe the `wait` and `waitpid` functions in ***Section 5.10***.) We establish the signal handler by adding the function call

    Signal (SIGCHLD, sig_chld);

The purpose of this example is to show that when writing network programs that catch signals, we must be cognizant of **interrupted system calls**, and we must handle them. In this specific example, running under `Solaris 9`, the `signal` function provided in the standard C library does not cause an interrupted system call to be automatically restarted by the kernel. That is, the `SA_RESTART` flag that we set in ***Figure 5.6*** is not set by the `signal` function in the system library. Some other systems automatically restart the interrupted system call. If we run the same example under 4.4BSD, using its library version of the `signal` function, the kernel restarts the interrupted system call and accept does not return an error. To handle this potential problem between different operating systems is one reason we define our own version of the signal function that we use throughout the text (***Figure 5.6***).

#### Handling Interrupted System Calls

We used the term "**slow system call**" to describe `accept`, and we use this term for any system call that **can block forever**. That is, the system call need never return. Most networking functions fall into this category. For example, there is no guarantee that a server's call to `accept` will ever return, if there are no clients that will connect to the server. Similarly, our server's call to `read` in ***Figure 5.3*** will never return if the client never sends a line for the server to echo. Other examples of slow system calls are reads and writes of pipes and terminal devices. A notable exception is disk I/O, which usually returns to the caller (assuming no catastrophic hardware failure).

The basic rule that applies here is that **when a process is blocked in a slow system call and the process catches a signal and the signal handler returns, the system call can return an error of EINTR**. Some kernels automatically restart some interrupted system calls. For portability, when we write a program that catches signals (most concurrent servers catch `SIGCHLD`), we must be prepared for slow system calls to return `EINTR`. Portability problems are caused by the qualifiers "can" and "some," which were used earlier, and the fact that support for the POSIX `SA_RESTART` flag is optional. **Even if an implementation supports the `SA_RESTART` flag, not all interrupted system calls may automatically be restarted**. Most Berkeley-derived implementations, for example, never automatically restart select, and some of these implementations never restart `accept` or `recvfrom`.

What we are doing in this piece of code is restarting the interrupted system call. This is fine for `accept`, along with functions such as `read`, `write`, `select`, and `open`. But there is one function that we cannot restart: `connect`. If this function returns `EINTR`, we cannot call it again, as doing so will return an immediate error. **When `connect` is interrupted by a caught signal and is not automatically restarted, we must call `select` to wait for the connection to complete**, as we will describe in ***Section 16.3***.

### 5.10 wait and waitpid Functions

If there are no terminated children for the process calling `wait`, but the process has one or more children that are still executing, then **`wait` blocks until the first of the existing children terminates**.

`waitpid` gives us more control over which process to wait for and whether or not to block. First, the `pid` argument lets us specify the process ID that we want to wait for. A value of `-1` says to wait for the first of our children to terminate. (There are other options, dealing with process group IDs, but we do not need them in this text.) The options argument lets us specify additional options. The most common option is `WNOHANG`. This option tells the kernel **not to block if there are no terminated children**.

Establishing a signal handler and calling `wait` from that handler are **insufficient for preventing zombies**. The problem is that all five signals are generated before the signal handler is executed, and the signal handler is executed only one time because **Unix signals are normally not queued**. Furthermore, this problem is nondeterministic.

The correct solution is to call `waitpid` instead of `wait`. ***Figure 5.11*** shows the version of our `sig_chld` function that handles `SIGCHLD` correctly. This version works because we call `waitpid` within a loop, fetching the status of any of our children that have terminated. We must specify the `WNOHANG` option: This tells `waitpid` not to block if there are running children that have not yet terminated. In ***Figure 5.7***, we cannot call wait in a loop, because there is no way to prevent `wait` from blocking if there are running children that have not yet terminated.

The purpose of this section has been to demonstrate three scenarios that we can encounter with network programming:

1. We must catch the `SIGCHLD` signal when forking child processes.

2. We must handle interrupted system calls when we catch signals.

3. A `SIGCHLD` handler must be coded correctly using `waitpid` to prevent any zombies from being left around.

### 5.11 Connection Abort before accept Returns

There is another condition similar to the interrupted system call example in the previous section that can cause `accept` to return a **nonfatal error**, in which case we should just **call `accept` again**. The sequence of packets shown in ***Figure 5.13*** has been seen on **busy servers** (typically busy Web servers).

Here, the three-way handshake completes, the connection is established, and then the client TCP sends an `RST` (reset). On the server side, the **connection is queued** by its TCP, **waiting for the server process to call `accept`** when the **`RST` arrives**. Sometime later, the server process calls `accept`.

Unfortunately, what happens to the aborted connection is **implementation-dependent**. **Berkeley-derived implementations** handle the aborted connection completely within the kernel, and the server process never sees it. Most **SVR4 implementations**, however, return an error to the process as the return from `accept`, and the **error depends on the implementation**. These SVR4 implementations return an errno of `EPROTO` ("protocol error"), but **POSIX specifies** that the return must be `ECONNABORTED` ("software caused connection abort") instead. The reason for the POSIX change is that `EPROTO` is also returned when some fatal protocol-related events occur on the streams subsystem. Returning the same error for the nonfatal abort of an established connection by the client makes it impossible for the server to know whether to call `accept` again or not. In the case of the `ECONNABORTED` error, the server can **ignore the error** and just **call accept again**.

### 5.12 Termination of Server Process

We will now start our client/server and then kill the server child process. This simulates the crashing of the server process, so we can see what happens to the client. (We must be careful to **distinguish between** the **crashing of the server process**, which we are about to describe, and the **crashing of the server host**, which we will describe in ***Section 5.14***.) The following steps take place:

1. We start the server and client and type one line to the client to verify that all is okay. That line is echoed normally by the server child.

2. We find the process ID of the server child and kill it. As part of process termination, all open descriptors in the child are closed. This causes a **`FIN` to be sent to the client**, and the **client TCP responds with an `ACK`**. This is the **first half** of the TCP connection termination.

3. The `SIGCHLD` signal is sent to the server parent and handled correctly (***Figure 5.12***).

4. Nothing happens at the client. The client TCP receives the `FIN` from the server TCP and responds with an `ACK`, but the problem is that the **client process is blocked** in the call to `fgets` waiting for a line from the terminal.

5. Running netstat at this point shows the state of the sockets.

        linux % netstat -a | grep 9877
        tcp        0      0 *:9877               *:*                 LISTEN
        tcp        0      0 localhost:9877       localhost:43604     FIN_WAIT2
        tcp        1      0 localhost:43604      localhost:9877      CLOSE_WAIT

6. We can still type a line of input to the client.

    This is allowed by TCP because the receipt of the `FIN` by the client TCP only indicates that the server process has closed **its end of** the connection and **will not be sending any more data**. The receipt of the `FIN` does not tell the client TCP that the server process has terminated (which in this case, it has). We will cover this again in ***Section 6.6*** when we talk about **TCP's half-close**.

    When the server TCP receives the data from the client, it **responds with an `RST`** since the process that had that socket open has terminated. We can verify that the `RST` was sent by watching the packets with `tcpdump`.

7. The client process will not see the `RST` because it calls `readline` immediately after the call to writen and `readline` returns `0` (`EOF`) immediately because of the `FIN` that was received in Step `2`. Our client is not expecting to receive an `EOF` at this point (***Figure 5.5***) so it quits with the error message "server terminated prematurely."

8. When the client terminates (by calling `err_quit` in ***Figure 5.5***), all its open descriptors are closed.

    What we have described also **depends on the timing** of the example. The client's call to `readline` may happen **before** the server's **`RST` is received** by the client, or it may happen **after**. If the `readline` happens before the `RST` is received, as we've shown in our example, the result is an **unexpected `EOF`** in the client. But if the `RST` arrives first, the result is an **`ECONNRESET` ("Connection reset by peer") error** return from `readline`.

### 5.13 SIGPIPE Signal

What happens if the client ignores the error return from `readline` and writes more data to the server? This can happen, for example, if the client needs to perform two writes to the server before reading anything back, with the first write eliciting the `RST`.

The rule that applies is: When a process **writes to a socket that has received an `RST`**, the `SIGPIPE` signal is sent to the process. The default action of this signal is to terminate the process, so the process must catch the signal to avoid being involuntarily terminated.

If the process either catches the signal and returns from the signal handler, or ignores the signal, the **`write` operation returns `EPIPE`**.

A frequently asked question (FAQ) on Usenet is how to obtain this signal on the first write, and not the second. This is not possible. Following our discussion above, the first write elicits the `RST` and the second write elicits the signal. **It is okay to write to a socket that has received a `FIN`, but it is an error to write to a socket that has received an `RST`**.

The recommended way to handle `SIGPIPE` depends on what the application wants to do when this occurs. If there is nothing special to do, then setting the signal disposition to `SIG_IGN` is easy, assuming that subsequent output operations will catch the error of `EPIPE` and terminate. If special actions are needed when the signal occurs (writing to a log file perhaps), then the signal should be caught and any desired actions can be performed in the signal handler. Be aware, however, that **if multiple sockets are in use, the delivery of the signal will not tell us which socket encountered the error**. If we need to know which write caused the error, then we must either ignore the signal or return from the signal handler and handle `EPIPE` from the `write`.

### 5.14 Crashing of Server Host

This scenario will test to see what happens when the **server host crashes**. To simulate this, we must run the client and server on different hosts. We then start the server, start the client, type in a line to the client to verify that the connection is up, **disconnect the server host** from the network, and type in another line at the client. This also covers the scenario of the **server host being unreachable** when the client sends data (i.e., some intermediate router goes down after the connection has been established).

5.14 Crashing of Server Host
This scenario will test to see what happens when the server host crashes. To simulate this, we must run the client and server on different hosts. We then start the server, start the client, type in a line to the client to verify that the connection is up, disconnect the server host from the network, and type in another line at the client. This also covers the scenario of the server host being unreachable when the client sends data (i.e., some intermediate router goes down after the connection has been established).

The following steps take place:

1. When the server host crashes, nothing is sent out on the existing network connections. That is, we are assuming the **host crashes** and is **not shut down** by an operator (which we will cover in **Section 5.16**).

2. We type a line of input to the client, it is written by writen (**Figure 5.5**), and is sent by the client TCP as a data segment. The client then blocks in the call to `readline`, waiting for the echoed reply.

3. If we watch the network with `tcpdump`, we will see the client TCP **continually retransmitting** the data segment, **trying to receive an `ACK`** from the server. ***Section 25.11*** of TCPv2 shows a typical pattern for TCP retransmissions: Berkeley-derived implementations retransmit the data segment 12 times, waiting for around 9 minutes before giving up. When the client TCP finally gives up (assuming the server host has not been rebooted during this time, or if the server host has not crashed but was unreachable on the network, assuming the host was still unreachable), **an error is returned to the client process**. Since the client is blocked in the call to **`readline`, it returns an error**. Assuming the server host crashed and there were no responses at all to the client's data segments, the error is `ETIMEDOUT`. But if some intermediate router determined that the server host was unreachable and responded with an `ICMP` "destination unreachable' message, the error is either `EHOSTUNREACH` or `ENETUNREACH`.

Although our client discovers (eventually) that the peer is down or unreachable, there are times when we want to detect this quicker than having to wait nine minutes. The solution is to place a `timeout` on the call to `readline`, which we will discuss in ***Section 14.2***.

The scenario that we just discussed detects that the server host has crashed only when we send data to that host. If we want to **detect the crashing of the server host even if we are not actively sending it data**, another technique is required. We will discuss the `SO_KEEPALIVE` socket option in ***Section 7.5***.
 
### 5.15 Crashing and Rebooting of Server Host

In this scenario, we will establish a connection between the client and server and then assume the server host crashes and reboots. In the previous section, the server host was still down when we sent it data. Here, we will let the server host reboot before sending it data. The easiest way to simulate this is to **establish the connection**, **disconnect the server** from the network, **shut down the server host** and then **reboot** it, and then **reconnect the server host** to the network. We **do not want the client to see the server host shut down** (which we will cover in ***Section 5.16***).

As stated in the previous section, if the client is not actively sending data to the server when the server host crashes, the **client is not aware that the server host has crashed**. (This **assumes we are not using the `SO_KEEPALIVE` socket option**.) The following steps take place:

1. We start the server and then the client. We type a line to verify that the connection is established.

2. The server host crashes and reboots.

3. We type a line of input to the client, which is sent as a TCP data segment to the server host.

4. When the server host reboots after crashing, its TCP **loses all information about connections** that existed before the crash. Therefore, the server TCP responds to the received data segment from the client with an `RST`.

5. Our client is blocked in the call to `readline` when the `RST` is received, causing `readline` to return the error `ECONNRESET`.

If it is important for our client to detect the crashing of the server host, even if the client is not actively sending data, then some other technique (such as the `SO_KEEPALIVE` socket option or some client/server **heartbeat** function) is required.

### 5.16 Shutdown of Server Host

The previous two sections discussed the crashing of the server host, or the server host being unreachable across the network. We now consider what happens if the server host is shut down by an operator while our server process is running on that host.

When a Unix system is shut down, the `init` process normally **sends the `SIGTERM` signal** to all processes (we can catch this signal), waits some fixed amount of time (often between `5` and `20` seconds), and then **sends the `SIGKILL` signal** (which we cannot catch) to any processes still running. This gives all running processes a short amount of time to clean up and terminate. If we do not catch `SIGTERM` and terminate, our server will be terminated by the `SIGKILL` signal. When the **process terminates**, all open descriptors are closed, and we then follow the same sequence of steps discussed in ***Section 5.12***. As stated there, we must use the `select` or `poll` function in our client to have the client **detect the termination of the server process as soon as it occurs**.

### 5.17 Summary of TCP Example

### 5.18 Data Format

### 5.19 Summary

## Chapter 6. I/O Multiplexing: The `select` and `poll` Functions

### 6.1 Introduction

In ***Section 5.12***, we saw our TCP client handling two inputs at the same time: standard input and a TCP socket. We encountered a problem when the client was blocked in a call to `fgets` (on standard input) and the server process was killed. The server TCP correctly sent a `FIN` to the client TCP, but since the client process was blocked reading from standard input, it never saw the `EOF` until it read from the socket (possibly much later). What we need is the capability to **tell the kernel that we want to be notified if one or more I/O conditions are ready** (i.e., input is ready to be read, or the descriptor is capable of taking more output). This capability is called **I/O multiplexing** and is provided by the `select` and `poll` functions. We will also cover a newer POSIX variation of the former, called `pselect`.

I/O multiplexing is typically used in networking applications in the following scenarios:

* When a client is **handling multiple descriptors** (normally interactive input and a network socket), I/O multiplexing should be used. This is the scenario we described previously.

* It is possible, but rare, for a client to **handle multiple sockets** at the same time. We will show an example of this using select in ***Section 16.5*** in the context of a Web client.

* If a TCP server handles both a **listening socket and its connected sockets**, I/O multiplexing is normally used, as we will show in ***Section 6.8***.

* If a server **handles both TCP and UDP**, I/O multiplexing is normally used. We will show an example of this in ***Section 8.15***.

* If a server handles **multiple services** and perhaps **multiple protocols** (e.g., the inetd daemon that we will describe in ***Section 13.5***), I/O multiplexing is normally used.

I/O multiplexing is **not limited to network programming**. Many nontrivial applications find a need for these techniques.

### 6.2 I/O Models

Before describing `select` and `poll`, we need to step back and look at the bigger picture, examining the basic differences in the five I/O models that are available to us under Unix:

* blocking I/O

* nonblocking I/O

* I/O multiplexing (`select` and `poll`)

* signal driven I/O (`SIGIO`)

* asynchronous I/O (the POSIX `aio_`functions)

As we show in all the examples in this section, there are normally **two distinct phases** for an input operation:

1. Waiting for the data to be ready

2. Copying the data from the kernel to the process

For an input operation on a socket, the first step normally involves **waiting for data to arrive on the network**. When the packet arrives, it is **copied into a buffer within the kernel**. The second step is **copying this data from the kernel's buffer into our application buffer**.

#### Blocking I/O Model

The most prevalent model for I/O is the blocking I/O model, which we have used for all our examples so far in the text. By default, all sockets are blocking. Using a datagram socket for our examples, we have the scenario shown in ***Figure 6.1***.

We use UDP for this example instead of TCP because with UDP, the concept of data being "ready" to read is simple: either an **entire datagram has been received or it has not**. With TCP it gets more complicated, as additional variables such as the socket's **low-water mark** come into play.

In the examples in this section, we also refer to `recvfrom` as a system call because we are differentiating between our application and the kernel. Regardless of how `recvfrom` is implemented (as a system call on a Berkeley-derived kernel or as a function that invokes the `getmsg` system call on a System V kernel), there is normally a **switch from running in the application to running in the kernel**, followed at some time later by a return to the application.

In ***Figure 6.1***, the process calls `recvfrom` and the system call does not return until the **datagram arrives** and is **copied into our application buffer**, or an error occurs. The most common error is the system call being interrupted by a signal, as we described in ***Section 5.9***. We say that our **process is blocked** the entire time from when it calls `recvfrom` until it returns. When `recvfrom` returns successfully, our application processes the datagram.

#### Nonblocking I/O Model

When we set a socket to be nonblocking, we are telling the kernel "when an I/O operation that I request cannot be completed without putting the process to sleep, **do not put the process to sleep, but return an error instead**." We will describe nonblocking I/O in ***Chapter 16***, but ***Figure 6.2*** shows a summary of the example we are considering.

The first three times that we call `recvfrom`, there is no data to return, so the kernel immediately **returns an error of `EWOULDBLOCK`** instead. The fourth time we call `recvfrom`, a datagram is ready, it is **copied into our application buffer**, and `recvfrom` returns successfully. We then process the data.

When an application sits in a **loop calling `recvfrom`** on a **nonblocking descriptor** like this, it is called **polling**. The application is continually polling the kernel to see if some operation is ready. This is often **a waste of CPU time**, but this model is **occasionally encountered**, normally on systems dedicated to one function.

#### I/O Multiplexing Model

With I/O multiplexing, we call `select` or `poll` and block in one of these two system calls, instead of blocking in the actual I/O system call. ***Figure 6.3*** is a summary of the I/O multiplexing model.

We block in a call to `select`, waiting for the datagram socket to be readable. When **`select` returns that the socket is readable**, we **then call `recvfrom`** to copy the datagram into our application buffer.

Comparing ***Figure 6.3*** to ***Figure 6.1***, there does not appear to be any advantage, and in fact, there is a slight disadvantage because using `select` requires two system calls instead of one. But the advantage in using `select`, which we will see later in this chapter, is that we **can wait for more than one descriptor** to be ready.

Another closely related I/O model is to use **multithreading with blocking I/O**. That model very closely resembles the model described above, except that instead of using `select` to **block on multiple file descriptors**, the program uses multiple threads (one per file descriptor), and **each thread is then free to call blocking system calls** like `recvfrom`.

#### Signal-Driven I/O Model

We can also use signals, telling the kernel to **notify us with the `SIGIO` signal** when the descriptor is ready. We call this signal-driven I/O and show a summary of it in ***Figure 6.4***.

We first enable the socket for signal-driven I/O (as we will describe in ***Section 25.2***) and install a signal handler using the `sigaction` system call. The return from this system call is immediate and our process continues; it is **not blocked**. When the datagram is ready to be read, the `SIGIO` signal is generated for our process. We can **either read the datagram from the signal handler** by calling recvfrom and then **notify the main loop** that the data is ready to be processed (this is what we will do in ***Section 25.3***), **or we can notify the main loop and let it read the datagram**.

Regardless of how we handle the signal, the advantage to this model is that we are not blocked while waiting for the datagram to arrive. The main loop can continue executing and just wait to be notified by the signal handler that **either the data is ready** to process **or the datagram is ready** to be read.

#### Asynchronous I/O Model

Asynchronous I/O is defined by the POSIX specification, and various differences in the real-time functions that appeared in the various standards which came together to form the current POSIX specification have been reconciled. In general, these functions work by telling the kernel to start the operation and to notify us when the entire operation (including the copy of the data from the kernel to our buffer) is complete. The main difference between this model and the signal-driven I/O model in the previous section is that with signal-driven I/O, the kernel tells us **when an I/O operation can be initiated**, but with asynchronous I/O, the kernel tells us **when an I/O operation is complete**. We show an example in **Figure 6.5**s.

We call `aio_read` (the POSIX asynchronous I/O functions begin with `aio_` or `lio_`) and pass the kernel the descriptor, buffer pointer, buffer size (the same three arguments for `read`), file `offset` (similar to `lseek`), and how to notify us when the entire operation is complete. This **system call returns immediately** and our **process is not blocked** while waiting for the I/O to complete. We assume in this example that we ask the kernel to generate **some signal** when the operation is complete. This signal is not generated **until the data has been copied into our application buffer**, which is different from the signal-driven I/O model.

As of this writing, few systems support POSIX asynchronous I/O. We are **not certain**, for example, if systems will support it for sockets. Our use of it here is as an example to compare against the signal-driven I/O model.

#### Comparison of the I/O Models

***Figure 6.6*** is a comparison of the five different I/O models. It shows that the main difference between the first four models is the first phase, as the **second phase in the first four models is the same**: the process is blocked in a call to `recvfrom` while the data is copied from the kernel to the caller's buffer. Asynchronous I/O, however, handles **both phases** and is different from the first four.

#### Synchronous I/O versus Asynchronous I/O

POSIX defines these two terms as follows:

* A synchronous I/O operation causes the requesting process to be **blocked until that I/O operation completes**.

* An asynchronous I/O operation does not cause the requesting process to be blocked.

Using these definitions, the first four I/O models—blocking, nonblocking, I/O multiplexing, and signal-driven I/O—are all synchronous because the actual I/O operation **(`recvfrom`) blocks the process**. Only the asynchronous I/O model matches the asynchronous I/O definition.

### 6.3 `select` Function





















