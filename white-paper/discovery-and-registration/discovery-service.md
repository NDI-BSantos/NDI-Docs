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

# Discovery Service

NDI Discovery server is a command line application available for Windows, MacOS, and Linux. The NDI Discovery service is designed to allow you to replace the automatic discovery NDI uses with a server that operates as a centralized registry of NDI sources. This can be very helpful for installations where you wish to avoid having significant mDNS traffic for a large number of sources. It can also be useful when multicast is not possible or desirable; it is very common for cloud computing services not to allow multicast traffic. When using the Discovery service, NDI can operate entirely in unicast mode and thus in almost any installation. The Discovery server supports all NDI functionality, including NDI groups. Clients should be configured to connect with the Discovery service instead of using mDNS to locate sources.

When there is a Discovery server, NDI applications will use both mDNS and the Discovery server to find and receive sources on the local network that are not on machines configured to use discovery.

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

For senders, if a Discovery service is specified, then mDNS will not be used; these sources will only be visible to other finders and receivers configured to use the Discovery server.

#### To configure the Discovery service for NDI clients, **you may use Access Manager (included in the NDI Tools Core Suite)** to enter the IP[^1] address of the Discovery server machine.

**Within NDI version 5, there is full support for redundant NDI Discovery servers.** When configuring a Discovery server, it is possible to specify a comma-delimited list of servers (e.g., “`192.168.10.10, 192.168.10.12`”), and then they will all be used simultaneously. If one of these servers then goes down, as long as one remains active, then all sources will always remain visible, no matter what the others do, then all sources can be seen.

This multiple-server capability can also be used to ensure entirely separate servers to allow sources to be broken into separate groups, which can serve many workflows or security needs.

Once two NDI devices have discovered each other on the network, video can be passed from the sending device to the receiving device. After the compression of the video, the NDI sending device opens a session to the receiving NDI device. At this point, we have two endpoints that consist of an IP address and a port number.

![Screenshot of NDI Access Manager tool](<../../.gitbook/assets/1 (1).png>)

In Windows and MacOS, the Discovery Server addresses are configured in the **Advanced Feature Tab.** In Linux, the addresses of the NDI Discovery Servers can be manually added in the NDI configuration file located in the home directory of the effective user: "ndi-config.v1.json"

Here is the way to manually set up the **Discovery Service** in the configuration file:

`"networks": {`\
`"ips": "",`\
`"discovery": "`<mark style="color:red;">`192.168.10.10,192.168.10.12`</mark>`",`



**NDI Discovery Service to control devices' discoverability.**

As mentioned above, with NDI 5, we enabled support for multiple NDI Discovery Servers. NDI Receivers can now specify a comma-delimited list of Discovery Servers (e.g., "`192.168.25.10, 192.168.25.12`"), and they will all operate simultaneously.

Moreover, an NDI Discovery Server can be connected to multiple network interfaces and supports multiple subnets.\
\
Using multiple servers and subnets allows sources to be organized into distinct areas. These areas can cater to various workflows and security requirements.

**Here is an example** that explains how multiple Discovery Servers can be used for control device discoverability:

<figure><img src="../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

Spinning up or shutting down the Discovery Servers with IP Address 192.168.25.243 - 192.168.25.244 - 192.168.25.245 will turn on or off discoverability between the 3 NDI areas.

The example above is related to multiple Discovery Servers in a single subnet.

However, the Discovery Server can be connected and shared between different subnets.

Discovery Servers can be easily deployed on Linux-based virtual machines, and the process of spinning up or shutting down Discovery Servers can be efficiently automated and managed by solutions like Terraform.\
\


[^1]: IP (Internet Protocol) is the communications protocol for the Internet, many wide area networks (WANs) and most local area networks (LANs) that define the rules, formats, and address scheme for exchanging datagrams or packets between a source computer or device and a destination computer or device.
