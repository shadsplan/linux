# Chapter 58. Sockets: Fundamentals of TCP/IP Networks

## 58.1 Internets

- An internetwork or, more commonly, internet (with a lowercase i), connects different computer networks, allowing hosts on all of the networks to communicate with one another. In other words, an internet is a network of computer networks.
- The term subnetwork, or subnet, is used to refer to one of the networks composing an internet.
- An internet aims to hide the details of different physical networks in order to present a unified network architecture to all hosts on the connected networks. This means, for example, that a single address format is used to identify all hosts in the internet.
- Figure 58-1 shows a simple internet. In this diagram, the machine tekapo is an example of a router, a computer whose function is to connect one subnetwork to another, transferring data between them.

## 58.2 Networking Protocol and Layers
- A networking protocol is a set of rules defining how information is to be transmitted across a network. Networking protocols are generally organized as a series of layers, with each layer building on the layer below it to add features that are made available to higher layers.
- One of the notions that lends great power and flexibility to protocol layering is transparency—each protocol layer shields higher layers from the operation and complexity of lower layers. Thus, for example, an application making use of TCP only needs to use the standard sockets API and to know that it is employing a reliable, bytestream transport service.

## 58.3 The Data-Link Layer
- The lowest layer in Figure 58-2 is the data-link layer, which consists of the device driver and the hardware interface (network card) to the underlying physical medium (e.g., a telephone line, a coaxial cable, or a fiber-optic cable). The data-link layer is concerned with transferring data across a physical link in a network.
- To transfer data, the data-link layer encapsulates datagrams from the network layer into units called frames. In addition to the data to be transmitted, each frame includes a header containing, for example, the destination address and frame size. The data-link layer transmits the frames across the physical link and handles acknowledgements from the receiver. (Not all data-link layers use acknowledgements.) This layer may perform error detection, retransmission, and flow control. Some data-link layers also split large network packets into multiple frames and reassemble them at the receiver.
- One characteristic of the data-link layer that is important for our discussion of IP is the maximum transmission unit (MTU). A data-link layer's MTU is the upper limit that the layer places on the size of a frame. Different data-link layers have different MTUs.

- Figure 58-3: Layered communication via the TCP/IP protocols
- Figure 58-4: Encapsulation within the TCP/IP protocol layers

## The Network Layer: IP
Above the data-link layer is the network layer, which is concerned with delivering packets (data) from the source host to the destination host. This layer performs a variety of tasks, including:
- breaking data into fragments small enough for transmission via the data-link layer (if necessary);
- routing data across the internet; and
- providing services to the transport layer

### IP transmits datagrams
IP transmits data in the form of datagrams (packets). Each datagram sent between two hosts travels independently across the network, possibly taking a different route. An IP datagram includes a header, which ranges in size from 20 to 60 bytes. The header contains the address of the target host, so that the datagram can be routed through the network to its destination, and also includes the originating address of the packet, so that the receiving host knows the origin of the datagram.

### IP is connectionless and unreliable
IP is described as a connectionless protocol, since it doesn't provide the notion of a virtual circuit connecting two hosts. IP is also an unreliable protocol: it makes a “best effort” to transmit datagrams from the sender to the receiver, but doesn't guarantee that packets will arrive in the order they were transmitted, that they won't be duplicated, or even that they will arrive at all. Nor does IP provide error recovery (packets with header errors are silently discarded). Reliability must be provided either by using a reliable transport-layer protocol (e.g., TCP) or within the application itself.

### IP may fragment datagrams
- We noted earlier that most data-link layers impose an upper limit (the MTU) on the size of data frames. For example, this upper limit is 1500 bytes on the commonly used Ethernet network architecture.
- When an IP datagram is larger than the MTU, IP fragments (breaks up) the datagram into suitably sized units for transmission across the network. These fragments
are then reassembled at the final destination to re-create the original datagram.

## 58.5 IP Addresses
An IP address consists of two parts: a network ID, which specifies the network on
which a host resides, and a host ID, which identifies the host within that network.

- When an organization applies for a range of IPv4 addresses for its hosts, it receives a 32-bit network address and a corresponding 32-bit network mask. In binary form, this mask consists of a sequence of 1s in the leftmost bits, followed by a sequence of 0s to fill out the remainder of the mask. The 1s indicate which part of the address contains the assigned network ID, while the 0s indicate which part of the address is available to the organization to assign as unique host IDs on its network. The size of the network ID part of the mask is determined when the address is assigned.
- Since the network ID component always occupies the leftmost part of the mask, the following notation is sufficient to specify the range of assigned addresses: `204.152.189.0/24`.
    - The /24 indicates that the network ID part of the assigned address consists of the leftmost 24 bits, with the remaining 8 bits specifying the host ID.
    - An organization holding this address can assign 254 unique Internet addresses to its computers—204.152.189.1 through 204.152.189.254. Two addresses can't be assigned. One of these is the address whose host ID is all 0 bits, which is used to identify the network itself. The other is the address whose host ID is all 1 bits—204.152.189.255 in this example—which is the subnet broadcast address
- Certain IPv4 addresses have special meanings. The special address 127.0.0.1 is normally defined as the loopback address, and is conventionally assigned the hostname localhost.
- Typically, IPv4 addresses are subnetted. Subnetting divides the host ID part of an IPv4 address into two parts: a subnet ID and a host ID.
    - The rationale for subnetting is that an organization often doesn't attach all of its hosts to a single network. Instead, the organization may operate a set of subnetworks (an “internal internetwork”), with each subnetwork being identified by the combination of the network ID plus the subnet ID. This combination is usually referred to as the extended network ID.
    - Figure 58-6 Example: For example, suppose that our assigned network ID is 204.152.189.0/24, and we choose to subnet this address range by splitting the 8 bits of the host ID into a 4-bit subnet ID and a 4-bit host ID. Under this scheme, the subnet mask would consist of 28 leading ones, followed by 4 zeros, and the subnet with the ID of 1 would be designated as 204.152.189.16/28.

## 58.6 The Transport Layer
There are two widely used transport-layer protocols in the TCP/IP suite:
- User Datagram Protocol (UDP) is the protocol used for datagram sockets.
- Transmission Control Protocol (TCP) is the protocol used for stream sockets.

### 58.6.1 Port Numbers

### Well-known, registered, and privileged ports
- Some well-known port numbers are permanently assigned to specific applications (also known as services). For example, the ssh (secure shell) daemon uses the well-known port 22, and HTTP (the protocol used for communication between web servers and browsers) uses the well-known port 80.
- In most TCP/IP implementations (including Linux), the port numbers in the range 0 to 1023 are also privileged, meaning that only privileged (CAP_NET_BIND_SERVICE) processes may bind to these ports. This prevents a normal user from implementing a malicious application that, for example, spoofs as ssh in order to obtain passwords.

### Ephemeral Ports
- If an application doesn't select a particular port (i.e., in sockets terminology, it doesn't bind() its socket to a particular port), then TCP and UDP assign a unique ephemeral port (i.e., short-lived) number to the socket. In this case, the application—typically a client—doesn't care which port number it uses, but assigning a port is necessary so that the transport-layer protocols can identify the communication endpoints. It also has the result that the peer application at the other end of the the communication channel knows how to communicate with this application. 
- TCP and UDP also assign an ephemeral port number if we bind a socket to port 0.
- IANA specifies the ports in the range 49152 to 65535 as dynamic or private, with the intention that these ports can be used by local applications and assigned as ephemeral ports.

### 58.6.2  User Datagram Protocol (UDP)
- UDP adds just two features to IP: port numbers and a data checksum to allow the detection of errors in the transmitted data.
- Like IP, UDP is connectionless. Since it adds no reliability to IP, UDP is likewise unreliable. If an application layered on top of UDP requires reliability, then this must be implemented within the application.

### 58.6.3 Transmission Control Protocol (TCP)
- TCP provides a reliable, connection-oriented, bidirectional, byte-stream communication channel between two endpoints (i.e., applications), as shown in Figure 58-8.
- We use the term TCP endpoint to denote the information maintained by the kernel for one end of a TCP connection. This information includes the send and receive buffers for this end of the connection, as well as state information that is maintained in order to synchronize the operation of the two connected endpoints.

### Connection Establishment
Before communication can commence, TCP establishes a communication channel between the two endpoints. During connection establishment, the sender and receiver can exchange options to advertise parameters for the connection.

### Packaging of data in segments
Data is broken into segments, each of which contains a checksum to allow the detection of end-to-end transmission errors. Each segment is transmitted in a single IP datagram.

### Acknowledgements, retransmissions, and timeouts
When a TCP segment arrives at its destination without errors, the receiving TCP sends a positive acknowledgement to the sender, informing it of the successfully delivered data. If a segment arrives with errors, then it is discarded, and no acknowledgement is sent. To handle the possibility of segments that never arrive or are discarded, the sender starts a timer when each segment is transmitted. If an acknowledgement is not received before the timer expires, the segment is retransmitted.

### Sequencing

Each byte that is transmitted over a TCP connection is assigned a logical sequence number. This number indicates the position of that byte in the data stream for the connection. (Each of the two streams in the connection has its own sequence numbering.) When a TCP segment is transmitted, it includes a field containing the sequence number of the first byte in the segment.

Attaching sequence numbers to each segment serves a variety of purposes:
- The sequence number allows TCP segments to be assembled in the correct order at the destination, and then passed as a byte stream to the application layer. (At any moment, multiple TCP segments may be in transit between sender and receiver, and these segments may arrive out of order.)
- The acknowledgement message passed from the receiver back to the sender can use the sequence number to identify which TCP segment was received.
- The receiver can use the sequence number to eliminate duplicate segments. Such duplicates may occur either because of the duplication of IP datagrams or because of TCP's own retransmission algorithm, which could retransmit a successfully delivered segment if the acknowledgement for that segment was lost or was not received in a timely fashion.

### Flow Control
- Flow control prevents a fast sender from overwhelming a slow receiver.
    - To implement flow control, the receiving TCP maintains a buffer for incoming data. (Each TCP advertises the size of this buffer during connection establishment.) Data accumulates in this buffer as it is received from the sending TCP, and is removed as the application reads data.
    - With each acknowledgement, the receiver advises the sender of how much space is available in its incoming data buffer (i.e., how many bytes the sender can transmit).
    - The TCP flow-control algorithm employs a so-called sliding window algorithm, which allows unacknowledged segments containing a total of up N (the offered window size) bytes to be in transit between the sender and receiver. If a receiving TCP's incoming data buffer fills completely, then the window is said to be closed, and the sending TCP stops transmitting.

### Congestion Control
- TCP's congestion-control algorithms are designed to prevent a fast sender from overwhelming a network. If a sending TCP transmits packets faster than they can be relayed by an intervening router, that router will start dropping packets. This could lead to high rates of packet loss and, consequently, serious performance degradation, if the sending TCP kept retransmitting these dropped segments at the same rate. TCP's congestion-control algorithms are important in two circumstances:
    - After connection establishment: At this time (or when transmission resumes on a connection that has been idle for some time), the sender could start by immediately injecting as many segments into the network as would be permitted by the window size advertised by the receiver. The problem here is that if the network can't handle this flood of segments, the sender risks overwhelming the network immediately.
    - When congestion is detected: If the sending TCP detects that congestion is occurring, then it must reduce its transmission rate. TCP detects that congestion is occurring based on the assumption that segment loss because of transmission errors is very low; thus, if a packet is lost, the cause is assumed to be congestion.
- TCP's congestion-control strategy employs two algorithms in combination: slow start and congestion avoidance.
    - The slow-start algorithm causes the sending TCP to initially transmit segments at a slow rate, but allows it to exponentially increase the rate as these segments are acknowledged by the receiving TCP. Slow start attempts to prevent a fast TCP sender from overwhelming a network. However, if unrestrained, slow start's exponential increase in the transmission rate could mean that the sender would soon overwhelm the network. TCP's congestion-avoidance algorithm prevents this, by placing a governor on the rate increase.
    - With congestion avoidance, at the beginning of a connection, the sending TCP starts with a small congestion window, which limits the amount of unacknowledged data that it can transmit. As the sender receives acknowledgements from the peer TCP, the congestion window initially grows exponentially. However, once the congestion window reaches a certain threshold believed to be close to the transmission capacity of the network, its growth becomes linear, rather than exponential. (An estimate of the capacity of the network is derived from a calculation based on the transmission rate that was in operation when congestion was detected, or is set at a fixed value after initial establishment of the connection.) At all times, the quantity of data that the sending TCP will transmit remains additionally constrained by the receiving TCP's advertised window and the local TCP's send buffer.
- In combination, the slow-start and congestion-avoidance algorithms allow the sender to rapidly raise its transmission speed up to the available capacity of the network, without overshooting that capacity. The effect of these algorithms is to allow data transmission to quickly reach a state of equilibrium, where the sender transmits packets at the same rate as it receives acknowledgements from the receiver.

## 58.8 Summary
- TCP/IP is a layered networking protocol suite.
- At the bottom layer of the TCP/IP protocol stack is the IP network-layer protocol.
    - IP transmits data in the form of datagrams. IP is connectionless, meaning that datagrams transmitted between source and destination hosts may take different routes across the network.
    - The original version of IP is IPv4. In the early 1990s, a new version of IP, IPv6, was devised. The most notable difference between IPv4 and IPv6 is that IPv4 uses 32 bits to represent a host address, while IPv6 uses 128 bits, thus allowing for a much larger number of hosts on the world-wide Internet.
- Various transport-layer protocols are layered on top of IP, of which the most widely used are UDP and TCP.
    - UDP is an unreliable datagram protocol.
    - TCP is a reliable, connection-oriented, byte-stream protocol.
        - TCP handles all of the details of connection establishment and termination.
        - TCP also packages data into segments for transmission by IP, and provides sequence numbering for these segments so that they can be acknowledged and assembled in the correct order by the receiver.
        - In addition, TCP provides flow control, to prevent a fast sender from overwhelming a slow receiver, and congestion control, to prevent a fast sender from overwhelming the network.