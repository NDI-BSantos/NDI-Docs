---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Manual connection

One approach to manually interconnect NDI devices is to specify the IP[^1] address of the transmitter in the receiver.

![Screenshot of NDI Access Manager tool](<../../.gitbook/assets/2 (1).png>)

In Windows and MacOS, this can be achieved using the NDI Access Manager in the External Sources feature. Several NDI hardware decoders also support this functionality.

For Linux, the IP addresses of NDI senders can be added manually in the NDI configuration file called "ndi-config.v1.json." This file is in the home directory of the user currently logged in.

Specifying the IP address of an NDI source allows the receiver to receive NDI sources that are in a different subnet[^2] and may not be discoverable by mDNS (Multicast DNS[^3]). This method enables the reception of NDI sources that might be otherwise inaccessible due to network configurations or limitations.

In Linux manual connections can be added in the NDI configuration file located in the home directory of the effective user: "ndi-config.v1.json"

Here is the way to manually set up NDI sources in the configuration file:

`"networks": {`\
`"ips": "`<mark style="color:red;">`192.168.123.200,10.10.123.22,`</mark>`",`\
`"discovery": "",`

[^1]: IP (Internet Protocol) is the communications protocol for the Internet, many wide area networks (WANs) and most local area networks (LANs) that define the rules, formats, and address scheme for exchanging datagrams or packets between a source computer or device and a destination computer or device.

[^2]: Subnet (short for subnetwork) refers to a distinct subdivision of an IP network, usually created for performance or security purposes. Subnets typically include the computers, systems, and devices in one location, office, or building, with all nodes sharing the same IP address prefix.

[^3]: DNS (Doman Name System) is a system used by the Internet and private networks to translate domain names into IP addresses.
