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

Forms the ehternet **frame** from IP packet with source and destination MAC address.

![Ethernet Frame](css/images/ethernet-frame.png){:class="img-responsive"}


#### 7. Physical Layer

Bits on the wire.


## Resources

- <sub>[tcpdump](https://www.tcpdump.org/manpages/pcap-filter.7.html)</sub>

*To Be Continued..*