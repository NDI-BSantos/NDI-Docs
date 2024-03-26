# mDNS

MAC (Media Access Control) address refers to a unique physical address identifying a network node. Sending and receiving video streams across an IP[^1] network requires applications that support video and can discover receiving applications that are looking for video.

NDI resolves host names to IP addresses over the LAN[^2] and does so automatically. When you start an application that sends NDI, the devices that can receive NDI become aware instantaneously. While this is a typical function on almost all networks, there are some cases where it is important to know how this works to properly configure networks utilizing managed data flow protocols.

By default, NDI utilizes [mDNS (multicast Domain Name System)](#user-content-fn-3)[^3] to create the zero-configuration environment for discovery. This service sends an IP multicast message that asks the host to identify itself. The target machine then multicasts a message that includes its own IP address. This multicast is seen by all NDI-receiving machines on the subnet[^4], which then use the information in that message to update their own caches[^5].

These multicast queries are sent to a multicast address, and thus, no single device is required to have global knowledge.

When a service or device sees a query for any service it recognizes, it provides a DNS[^6] response with the information from its cache[^7]. The primary benefit of using mDNS is that it requires little or no administration to set up. Unless the network is specifically configured not to allow mDNS, NDI sources will be discovered. This format works when no infrastructure is present and can span infrastructure failures.

**The mDNS** [**Ethernet**](#user-content-fn-8)[^8] **frame is a** [**multicast UDP packet that broadcasts to**](#user-content-fn-9)[^9]**:**

| [MAC Adress](#user-content-fn-10)[^10] | 01:00:5E:00:00:FB (for IPv4) |
| -------------------------------------- | ---------------------------- |
| IPv4[^11] Address                      | 224.0.0.251                  |
| UDP Port                               | 5353                         |

Choosing the network location type on Windows devices is critical for the successful discovery and registration of NDI. Typically, the first time a Windows machine is connected to a network, a dialog window appears that allows the user to choose the network location type: Private or Public. By default, Windows sets a new network location to Public.

This location is designed to keep machines from being visible and responding to broadcast pings. This location type also affects mDNS responses and keeps NDI video streams from being discovered and registered on the network.

**Network locations should be set to Private for successful discovery and registration of NDI.** The Domain[^12] network location is used for domain networks, such as those at enterprise workplaces. The network administrator controls this type of network location, and it cannot be selected or changed. In this type of configuration, mDNS discovery must be allowed at the domain level. Because mDNS uses a link-local multicast address, its capacity is limited to a single physical or logical LAN[^13].

[^1]: IP (Internet Protocol) is the communications protocol for the Internet, many wide area networks (WANs) and most local area networks (LANs) that define the rules, formats, and address scheme for exchanging datagrams or packets between a source computer or device and a destination computer or device.

[^2]: LAN (Local Area Network) is a network that connects computers and devices in a room, building or group of buildings. LANs are typically deployed in homes, offices, and schools, where users share access to the same server, resources, and data storage. A system of LANs can also be connected to form a WAN (Wide Area Network).

[^3]: Apple’s mDNS is published as a standards track proposal (RFC 6762) [https://tools.ietf.org/html/rfc6762](https://tools.ietf.org/html/rfc6762)

[^4]: Subnet (short for subnetwork) refers to a distinct subdivision of an IP network, usually created for performance or security purposes. Subnets typically include the computers, systems, and devices in one location, office, or building, with all nodes sharing the same IP address prefix.

[^5]: Cache refers to a reserved section of computer memory or an independent high-speed storage device used to accelerate access and retrieval of commonly used data.

[^6]: DNS (Doman Name System) is a system used by the Internet and private networks to translate domain names into IP addresses.

[^7]: Cache refers to a reserved section of computer memory or an independent high-speed storage device used to accelerate access and retrieval of commonly used data.

[^8]: Ethernet, standardized as IEEE 802.3, refers to a series of LAN (Local Area Network) technologies used to connect computers and other devices to a home or business network.\
    Ethernet is a physical and data link layer networking protocol that supports data transfer rates starting at 10 Mbps, typically over twisted pair cabling, but also fiber optic and coaxial cabling.

[^9]: [https://en.wikipedia.org/wiki/Multicast\_DNS](https://en.wikipedia.org/wiki/Multicast\_DNS)

[^10]: 

[^11]: IPv4 (Internet Protocol Version 4) is the fourth and most used version of the Internet Protocol. IPv4 uses a 32-bit IP address scheme for network identification and communication, with each unique IP address expressed as four numbers (between 0 and 255) separated by decimal points.

[^12]: A domain refers to a LAN subnetwork of users, systems, devices, and servers. Domain can also refer to the IP address of a website on the Internet.

[^13]: LAN (Local Area Network) is a network that connects computers and devices in a room, building or group of buildings. LANs are typically deployed in homes, offices, and schools, where users share access to the same server, resources, and data storage. A system of LANs can also be connected to form a WAN (Wide Area Network).
