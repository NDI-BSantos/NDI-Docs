---
layout:
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# NDI Protocols

### Reliable UDP – NDI 5

In NDI version 5 the default communication mechanism is a Reliable UDP[^1] protocol that represents the state-of-the-art communication protocol that is implemented by building upon all the experience we have seen in the real world with NDI across a massive variety of different installations

Reliable UDP, also known as RUDP, is a transport protocol that combines the advantages of UDP's low latency and simplicity with the reliability of TCP (Transmission Control Protocol). It is designed specifically for real-time multimedia applications, where maintaining the timeliness of data is crucial.

In the context of NDI, Reliable UDP is employed to ensure that video and audio streams are delivered reliably and with minimal delay. It achieves this by implementing several mechanisms:

Sequencing: Reliable UDP assigns a sequence number to each packet it sends. This allows the receiving end to detect missing or out-of-order packets and request retransmissions if necessary.

Retransmissions: If a packet is lost or arrives out of order, the receiving end can request a retransmission of the missing packet(s) using the sequence number information.

Flow control: Reliable UDP incorporates flow control mechanisms to manage the rate of data transmission. This prevents overwhelming the network or the receiving device with more data than it can handle, ensuring a smoother streaming experience.

Congestion control: RUDP also includes congestion control algorithms to prevent network congestion and avoid unnecessary packet loss. It dynamically adjusts the transmission rate based on network conditions, maintaining optimal throughput without overwhelming the network.

### Multipath TCP – NDI 4

This protocol permits transport across multiple NICs and all network paths, it is intended to use hardware-accelerated network adapters with adaptive bandwidth sharing across NICs.

Multipath TCP[^2] is a transmission protocol that offers advantages such as maximizing throughput, optimizing resource usage, and enhancing network redundancy. It can seamlessly integrate multiple network pathways, including wireless and mobile networks. It is especially efficient when used with NDI (equipment that utilizes multiple Gigabit connections to exchange a large number of NDI streams.

However, in scenarios where 10Gbit interfaces are connected with 1Gbit interfaces, Multipath TCP's efficiency is compromised. This is primarily due to network switches being unable to effectively manage network congestion in such situations. As a result, the protocol may not perform optimally in these specific network configurations.

### UDP with Forward Error Correction – NDI 3

This alternative protocol to TCP[^3] is used when reliable delivery of data [packets ](#user-content-fn-4)[^4]is not required. UDP[^5] is typically used for applications where timeliness is of higher priority than accuracy, such as streaming media, teleconferencing, and voice-over-IP (VoIP). Forward error correction (FEC) is a method of obtaining error control in data transmission in which the source (transmitter) sends redundant data and the destination (receiver).

UDP (User Datagram Protocol) with Forward Error Correction (FEC) is a beneficial approach when the network is prone to errors or not entirely reliable. It provides a solution for error correction when data packets get lost or corrupted during transmission.

However, it's important to note that using UDP with FEC requires additional computational processing on the receiver side. The receiver needs to implement algorithms and mechanisms to manage the error correction process. This involves decoding the received data and applying error correction techniques to recover any lost or corrupted packets.

### Single TCP – NDI 1

This network communications protocol enables two host systems to establish a connection, exchange data packets[^6], and ensure data is delivered intact to the correct destination. TCP[^7] is typically grouped with IP (Internet Protocol) and is collectively known as TCP/IP.

Single-TCP is supported on all NDI versions. While the other transmission modes are likely to perform better, **this mode offers baseline compatibility for all NDI clients**.

[^1]: UDP (User Datagram Protocol) is an alternative protocol to TCP that is used when reliable delivery of data packets is not required. UDP is typically used for applications where timeliness is of higher priority than accuracy, such as streaming media, teleconferencing, and voice-over IP (VoIP).

[^2]: TCP (Transmission Control Protocol) is a network communications protocol that enables two host systems to establish a connection and exchange data packets, ensuring that data is delivered to the correct destination. TCP is typically grouped with IP (Internet Protocol) and is known collectively as TCP/IP.

[^3]: TCP (Transmission Control Protocol) is a network communications protocol that enables two host systems to establish a connection and exchange data packets, ensuring that data is delivered to the correct destination. TCP is typically grouped with IP (Internet Protocol) and is known collectively as TCP/IP.

[^4]: A packet, also known as a frame or datagram, is a unit of data transmitted over a packet-switched network, such as a LAN, WAN, or the Internet.

[^5]: UDP (User Datagram Protocol) is an alternative protocol to TCP that is used when reliable delivery of data packets is not required. UDP is typically used for applications where timeliness is of higher priority than accuracy, such as streaming media, teleconferencing, and voice-over IP (VoIP).

[^6]: A packet, also known as a frame or datagram, is a unit of data transmitted over a packet-switched network, such as a LAN, WAN, or the Internet.

[^7]: TCP (Transmission Control Protocol) is a network communications protocol that enables two host systems to establish a connection and exchange data packets, ensuring that data is delivered to the correct destination.
