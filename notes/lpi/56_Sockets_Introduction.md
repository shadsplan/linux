# Chapter 56. Sockets Introduction

Sockets are a method of IPC that allow data to be exchanged between applications, either on the same host (computer) or on different hosts connected by a network.

## 56.1 Overview
In a typical client-server scenario, applications communicate using sockets as follows:
- Each application creates a socket. A socket is the “apparatus” that allows communication, and both applications require one.
- The server binds its socket to a well-known address (name) so that clients can locate it.

A socket is created using the socket() system call, which returns a file descriptor used to refer to the socket in subsequent system calls: `fd = socket(domain, type, protocol);` Assume that protocol is always 0 for now.

### Communication domains
Sockets exist in a communication domain, which determines:
- the method of identifying a socket (i.e., the format of a socket “address”); and
- the range of communication (i.e., either between applications on the same host or between applications on different hosts connected via a network).

Modern operating systems support at least the following domains:
- The UNIX (AF_UNIX) domain allows communication between applications on the same host. (POSIX.1g used the name AF_LOCAL as a synonym for AF_UNIX, but this name is not used in SUSv3.)
- The IPv4 (AF_INET) domain allows communication between applications running on hosts connected via an Internet Protocol version 4 (IPv4) network.
- The IPv6 (AF_INET6) domain allows communication between applications running on hosts connected via an Internet Protocol version 6 (IPv6) network.
Although IPv6 is designed as the successor to IPv4, the latter protocol is currently still the most widely used.

Table 56-1 summarizes the characteristics of these socket domains.

### Socket Types

Every sockets implementation provides at least two types of sockets: stream and datagram. These socket types are supported in both the UNIX and the Internet domains. Table 56-2 summarizes the properties of these socket types.

- Stream sockets (SOCK_STREAM) provide a reliable, bidirectional, byte-stream communication channel. By the terms in this description, we mean the following:
    - Reliable means that we are guaranteed that either the transmitted data will arrive intact at the receiving application, exactly as it was transmitted by the sender (assuming that neither the network link nor the receiver crashes), or that we'll receive notification of a probable failure in transmission.
    - Bidirectional means that data may be transmitted in either direction between two sockets.
    - Byte-stream means that, as with pipes, there is no concept of message boundaries (refer to Section 44.1).
- Stream sockets operate in connected pairs. For this reason, stream sockets are described as connection-oriented. The term peer socket refers to the socket at the other end of a connection; peer address denotes the address of that socket; and peer application denotes the application utilizing the peer socket.
- Datagram sockets (SOCK_DGRAM) allow data to be exchanged in the form of messages called datagrams. With datagram sockets, message boundaries are preserved, but data transmission is not reliable. Messages may arrive out of order, be duplicated, or not arrive at all. Datagram sockets are an example of the more generic concept of a connectionless socket. Unlike a stream socket, a datagram socket doesn't need to be connected to another socket in order to be used.
    - In the Internet domain, datagram sockets employ the User Datagram Protocol (UDP), and stream sockets (usually) employ the Transmission Control Protocol (TCP). Instead of using the terms Internet domain datagram socket and Internet domain stream socket, we'll often just use the terms UDP socket and TCP socket, respectively.

- Figure 56-1: Overview of system calls used with stream sockets
- Figure 56-4: Overview of system calls used with datagram sockets

## 56.7 Summary

- Sockets allow communication between applications on the same host or on different hosts connected via a network.
- A socket exists within a communication domain, which determines the range of communication of communication and the address format used to identify the socket. SUSv3 specifies the UNIX (AF_UNIX), IPv4 (AF_INET), and IPv6 (AF_INET6) communication domains.
- Most applications use one of two socket types: stream or datagram. Stream sockets (SOCK_STREAM) provide a reliable, bidirectional, byte-stream communication channel between two endpoints. Datagram sockets (SOCK_DGRAM) provide unreliable, connectionless, message-oriented communication.
- A typical stream socket server creates its socket using socket(), and then binds the socket to a well-known address using bind(). The server then calls listen() to allow connections to be received on the socket. Each client connection is then accepted on the listening socket using accept(), which returns a file descriptor for a new socket that is connected to the client's socket. A typical stream socket client creates a socket using socket(), and then establishes a connection by calling connect(), specifying the server's well-known address. After two stream sockets are connected, data can be transferred in either direction using read() and write(). Once all processes with a file descriptor referring to a stream socket endpoint have performed an implicit or explicit close(), the connection is terminated.
- A typical datagram socket server creates a socket using socket(), and then binds it to a well-known address using bind(). Because datagram sockets are connectionless, the server's socket can be used to receive datagrams from any client. Datagrams can be received using read() or using the socket-specific recvfrom() system call, which returns the address of the sending socket. A datagram socket client creates a socket using socket(), and then uses sendto() to send a datagram to a specified (i.e., the server's) address. The connect() system call can be used with a datagram socket to set a peer address for the socket. After doing this, it is no longer necessary to specify the destination address for outgoing datagrams; a write() call can be used to send a datagram.