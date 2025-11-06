# Chapter 59. Sockets: Internet Domains

As noted in Chapter 58, Internet domain socket addresses consist of an IP address and a port number. Although computers use binary representations of IP addresses and port numbers, humans are much better at dealing with names than with numbers. Therefore, we describe the techniques used to identify host computers and ports using names. Our discussion of hostnames includes a description of the Domain Name System (DNS), which implements a distributed database that maps hostnames to IP addresses and vice versa.

## 59.1 Internet Domain Sockets

Internet domain stream sockets are implemented on top of TCP. They provide a reliable, bidirectional, byte-stream communication channel.

Internet domain datagram sockets are implemented on top of UDP. UDP sockets are similar to their UNIX domain counterparts, but note the following differences:
    - UNIX domain datagram sockets are reliable, but UDP sockets are not—datagrams may be lost, duplicated, or arrive in a different order from that in which they were sent.
    - Sending on a UNIX domain datagram socket will block if the queue of data for the receiving socket is full. By contrast, with UDP, if the incoming datagram would overflow the receiver's queue, then the datagram is silently dropped.


## 59.3 Data Representation

- When writing network programs, we need to be aware of the fact that different computer architectures use different conventions for representing various data types.
- Because of these differences in data representation, applications that exchange data between heterogeneous systems over a network must adopt some common convention for encoding that data. The sender must encode data according to this convention, while the receiver decodes following the same convention. The process of putting data into a standard format for transmission across a network is referred to as marshalling.
- Typically, these standards define a fixed format for each data type (defining, for example, byte order and number of bits used). As well as being encoded in the required format, each data item is tagged with extra field(s) identifying its type (and, possibly, length).

## 59.5 Overview of Host and Service Conversion Functions
- Computers represent IP addresses and port numbers in binary. However, humans find names easier to remember than numbers. Employing symbolic names also provides a useful level of indirection; users and programs can continue to use the same name even if the underlying numeric value changes.
- A hostname is the symbolic identifier for a system that is connected to a network (possibly with multiple IP addresses). A service name is the symbolic representation of a port number.

The following methods are available for representing host addresses and ports:
- A host address can be represented as a binary value, as a symbolic hostname, or in presentation format (dotted-decimal for IPv4 or hex-string for IPv6).
- A port can be represented as a binary value or as a symbolic service name.

## 59.8 Domain Name System (DNS)
- Before the advent of DNS, mappings between hostnames and IP addresses were defined in a manually maintained local file, /etc/hosts. However, the /etc/hosts scheme scales poorly, and then becomes impossible, as the number of hosts in the network increases (e.g., the Internet, with millions of hosts).
- DNS was devised to address this problem. The key ideas of DNS are the following:
    - Hostnames are organized into a hierarchical namespace (Figure 59-2). Each node in the DNS hierarchy has a label (name), which may be up to 63 characters. At the root of the hierarchy is an unnamed node, the “anonymous root.”
    - A node's domain name consists of all of the names from that node up to the root concatenated together, with each name separated by a period (.). For example, google.com is the domain name for the node google.
    - A fully qualified domain name (FQDN), such as www.kernel.org., identifies a host within the hierarchy. A fully qualified domain name is distinguished by being terminated by a period, although in many contexts the period may be omitted.
    - No single organization or system manages the entire hierarchy. Instead, there is a hierarchy of DNS servers, each of which manages a branch (a zone) of the tree. Normally, each zone has a primary master name server, and one or more secondary name servers, which provide backup in the event that the primary master name server crashes. Zones may themselves be divided into separately managed smaller zones. When a host is added within a zone, or the mapping of a hostname to an IP address is changed, the administrator responsible for the corresponding local name server updates the name database on that server. (No manual changes are required on any other name-server databases in the hierarchy.)
    - When a program calls getaddrinfo() to resolve (i.e., obtain the IP address for) a domain name, getaddrinfo() employs a suite of library functions (the resolver library) that communicate with the local DNS server. If this server can't supply the required information, then it communicates with other DNS servers within the hierarchy in order to obtain the information. Occasionally, this resolution process may take a noticeable amount of time, and DNS servers employ caching techniques to avoid unnecessary communication for frequently queried domain names.

### Recursive and Iterative Resolution Requests
- DNS resolution requests fall into two categories: recursive and iterative.
    - In a recursive request, the requester asks the server to handle the entire task of resolution, including the task of communicating with any other DNS servers, if necessary. When an application on the local host calls getaddrinfo(), that function makes a recursive request to the local DNS server.
    - If the local DNS server does not itself have the information to perform the resolution, it resolves the domain name iteratively:
        - Suppose that the local DNS server is asked to resolve the name www.otago.ac.nz. To do this, it first communicates with one of a small set of root name servers that every DNS server is required to know about. (We can obtain a list of these servers using the command dig . NS or from the web page at http://www.root-servers.org/.)
        - Given the name www.otago.ac.nz, the root name server refers the local DNS server to one of the nz DNS servers.
        - The local DNS server then queries the nz server with the name www.otago.ac.nz, and receives a response referring it to the ac.nz server.
        - The local DNS server then queries the ac.nz server with the name www.otago.ac.nz, and is referred to the otago.ac.nz server.
        - Finally, the local DNS server queries the otago.ac.nz server with the name www.otago.ac.nz, and obtains the required IP address.

## 59.9 The /etc/services File
As noted in Section 58.6.1, well-known port numbers are centrally registered by IANA. Each of these ports has a corresponding service name. Because service numbers are centrally managed and are less volatile than IP addresses, an equivalent of the DNS server is usually not necessary. Instead, the port numbers and service names are recorded in the file /etc/services.

## 59.14 UNIX Versus Internet Domain Sockets
- However, when using sockets to communicate between applications on the same system, we have the choice of using either Internet or UNIX domain sockets. In the case, which domain should we use and why?
- **Writing an application using just Internet domain sockets is often the simplest approach, since it will work on both a single host and across a network. However, there are some reasons why we may choose to use UNIX domain sockets:**
    - On some implementations, UNIX domain sockets are faster than Internet domain sockets.
    - We can use directory (and, on Linux, file) permissions to control access to UNIX domain sockets, so that only applications with a specified user or group ID can connect to a listening stream socket or send a datagram to a datagram socket. This provides a simple method of authenticating clients. With Internet domain sockets, we need to do rather more work if we wish to authenticate clients.
    - Using UNIX domain sockets, we can pass open file descriptors and sender credentials, as summarized in Section 61.13.3.

## 59.16 Summary

- Internet domain sockets allow applications on different hosts to communicate via a TCP/IP network. An Internet domain socket address consists of an IP address and a port number. In IPv4, an IP address is a 32-bit number; in IPv6, it is a 128-bit number. Internet domain datagram sockets operate over UDP, providing connectionless, unreliable, message-oriented communication. Internet domain stream sockets operate over TCP, and provide a reliable, bidirectional, byte-stream communication channel between two connected applications.
- Consideration of hostname conversions led us into a discussion of DNS, which implements a distributed database for a hierarchical directory service. The advantage of DNS is that the management of the database is not centralized. Instead, local zone administrators update changes for the hierarchical component of the database for which they are responsible, and DNS servers communicate with one another in order to resolve a hostname.
