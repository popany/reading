# UNIX Network Programming

## Chapter 1. Introduction

### 1.1 Introduction

Client and server is used by most network-aware applications. Deciding that the client always initiates requests tends to simplify the protocol as well as the programs themselves. Of course, some of the more complex network applications also require **asynchronous callback communication**, where the server initiates a message to the client. But it is far more common for applications to stick to the **basic client/server model**.

### 1.2 A Simple Daytime Client 

**Figure 1.5 TCP daytime client.**

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
