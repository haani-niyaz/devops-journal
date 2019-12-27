---
layout: post
title:  "TCP/IP Notebook"
date:   2019-01-06
author: "Haani Niyaz"
tags: 
 - networking
 - tutorial
---

* TOC
{:toc}

### OSI Model Recap

#### 1. Presentation Layer

Ininitiates connection with data.

<sub>Protocol : HTTP</sub>

#### 2. Application Layer

Formats data with optional compression and encryption.

#### 3. Session Layer

Initiates, maintains and terminates session for data transfer.

#### 4. Transport Layer

Data is encapsulated into a **segment** with source and destination port.

<sub>Protocol : TCP</sub>

![TCP Segment](css/images/tcp-segment.png){:class="img-responsive"}


#### 5. Network Layer

Addressing and routing information is encapsulated into a **packet** with data.

<sub>Protocol : IP</sub>

![IP Packet](css/images/ip-packet.png){:class="img-responsive"}

#### 6. Data Link Layer

Forms the ethernet **frame** from IP packet with source and destination MAC address.

![Ethernet Frame](css/images/ethernet-frame.png){:class="img-responsive"}


#### 7. Physical Layer

Bits on the wire.


### Simple Connection

Each arrow in the flow diagram below depicts a packet. The first 3 lines illustrates a **3 way handshake**. The last 4 lines illustrates a **closing** connection.

![fin-ack](css/images/tcp-simple-flow.png){:class="img-responsive"}


#### 3 Way Handshake

As per described in [TCP in a nutshell](http://www.cs.miami.edu/home/burt/learning/Csc524.032/notes/tcp_nutshell.html):

> A Connection set-up uses the `SYN` flags. They are not used except for connection set-up. The client establishing a connection selects an initial sequence number X and sends a packet with sequence number X and `SYN` flag 1. The server will select its own initial sequence number Y and will send a packet with sequence number Y, `SYN` flag 1, acknowledgement number X+1 and `ACK` flag 1. The initiator will complete the three way handshake by sending a packet with `ACK` flag 1 and acknowledgement number Y+1. The connection is now established.


#### Closing a Connection

The client (i.e: 192.168.20.70) sends an `ACK` to inform the server (i.e: 74.125.131.2) it has received all of the data. The server then sends a `FIN` back to the client and an `ACK` to inform the client it received the last packet. The client in return send its own `FIN` along with the `ACK` for the last packet received from the server.

	

## Resources

- <sub>[tcpdump](https://www.tcpdump.org/manpages/pcap-filter.7.html)</sub>
- <sub>[TCP in a nutshell](http://www.cs.miami.edu/home/burt/learning/Csc524.032/notes/tcp_nutshell.html)</sub>
