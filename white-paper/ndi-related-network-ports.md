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

# NDI Related Network Ports

| Port        | Type     | Use                                                                                                                                                                                                                                                                                                                                                                                     |
| ----------- | -------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 5353        | UDP[^1]  | This is the standard port[^2] used for mDNS communication and is always used for multicast sending of the current sources onto the network.                                                                                                                                                                                                                                             |
| 5959        | TCP[^3]  | NDI Discovery Server is an optional method to have NDI devices perform discovery. This can be beneficial in large configurations when you need to connect NDI devices between subnets[^4] or if mDNS is blocked.                                                                                                                                                                        |
| 5960        | TCP[^5]  | This is a TCP port[^6] used for remote sources to query this machine and discover all the sources running on it. This is used, for instance, when a machine is added by an IP address in the access manager so that from an IP address alone, all the sources currently running on that machine can be discovered automatically.                                                        |
| 5961 and up | TCP[^7]  | These are the base TCP connections used for each NDI stream. For each current connection, at least one port[^8] number will be used in this range.                                                                                                                                                                                                                                      |
| 5960 and up | UDP[^9]  | In version 5 and above, when using Reliable UDP connections, it will use a very small number of ports in the range of 5960 for UDP. These port[^10] numbers are shared with the TCP connections. Because connection sharing is used in this mode, the number of ports required is very limited and only one port is needed per NDI process running and not one port per NDI connection. |
| 6960 and up | TCP/UDP  | When using multi-TCP or UDP receiving, at least one port[^11] number in this range will be used for each connection.                                                                                                                                                                                                                                                                    |
| 7960 and up | TCP/UDP  | When using multi-TCP, unicast UDP, or multicast UDP sending, at least one port[^12] number in this range will be used for each connection.                                                                                                                                                                                                                                              |
| Ephemeral   | TCP[^13] | Legacy to NDI v1 - The current versions (4.6 and later) no longer use any ports in the ephemeral port[^14] range.                                                                                                                                                                                                                                                                       |

[^1]: UDP (User Datagram Protocol) is an alternative protocol to TCP that is used when reliable delivery of data packets is not required. UDP is typically used for applications where timeliness is of higher priority than accuracy, such as streaming media, teleconferencing, and voice-over IP (VoIP).

[^2]: A port is a communications channel for data transmission to and from a computer on a network. Each port is identified by a 16-bit number between 0 and 65535, with each process, application, or service using a specific port, or multiple ports, for data transmission. Port can also refer to a hardware socket used to physically connect a device or device cable to your computer or network.

[^3]: TCP (Transmission Control Protocol) is a network communications protocol that enables two host systems to establish a connection and exchange data packets, ensuring that data is delivered to the correct destination. TCP is typically grouped with IP (Internet Protocol) and is known collectively as TCP/IP.

[^4]: Subnet (short for subnetwork) refers to a distinct subdivision of an IP network, usually created for performance or security purposes. Subnets typically include the computers, systems, and devices in one location, office, or building, with all nodes sharing the same IP address prefix.

[^5]: TCP (Transmission Control Protocol) is a network communications protocol that enables two host systems to establish a connection and exchange data packets, ensuring that data is delivered to the correct destination. TCP is typically grouped with IP (Internet Protocol) and is known collectively as TCP/IP.

[^6]: A port is a communications channel for data transmission to and from a computer on a network. Each port is identified by a 16-bit number between 0 and 65535, with each process, application, or service using a specific port, or multiple ports, for data transmission. Port can also refer to a hardware socket used to physically connect a device or device cable to your computer or network.

[^7]: TCP (Transmission Control Protocol) is a network communications protocol that enables two host systems to establish a connection and exchange data packets, ensuring that data is delivered to the correct destination. TCP is typically grouped with IP (Internet Protocol) and is known collectively as TCP/IP.

[^8]: A port is a communications channel for data transmission to and from a computer on a network. Each port is identified by a 16-bit number between 0 and 65535, with each process, application, or service using a specific port, or multiple ports, for data transmission. Port can also refer to a hardware socket used to physically connect a device or device cable to your computer or network.

[^9]: UDP (User Datagram Protocol) is an alternative protocol to TCP that is used when reliable delivery of data packets is not required. UDP is typically used for applications where timeliness is of higher priority than accuracy, such as streaming media, teleconferencing, and voice-over IP (VoIP).

[^10]: A port is a communications channel for data transmission to and from a computer on a network. Each port is identified by a 16-bit number between 0 and 65535, with each process, application, or service using a specific port, or multiple ports, for data transmission. Port can also refer to a hardware socket used to physically connect a device or device cable to your computer or network.

[^11]: A port is a communications channel for data transmission to and from a computer on a network. Each port is identified by a 16-bit number between 0 and 65535, with each process, application, or service using a specific port, or multiple ports, for data transmission. Port can also refer to a hardware socket used to physically connect a device or device cable to your computer or network.

[^12]: A port is a communications channel for data transmission to and from a computer on a network. Each port is identified by a 16-bit number between 0 and 65535, with each process, application, or service using a specific port, or multiple ports, for data transmission. Port can also refer to a hardware socket used to physically connect a device or device cable to your computer or network.

[^13]: TCP (Transmission Control Protocol) is a network communications protocol that enables two host systems to establish a connection and exchange data packets, ensuring that data is delivered to the correct destination. TCP is typically grouped with IP (Internet Protocol) and is known collectively as TCP/IP.

[^14]: A port is a communications channel for data transmission to and from a computer on a network. Each port is identified by a 16-bit number between 0 and 65535, with each process, application, or service using a specific port, or multiple ports, for data transmission. Port can also refer to a hardware socket used to physically connect a device or device cable to your computer or network.
