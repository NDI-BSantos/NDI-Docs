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

# Multicast

NDI supports multicast-based video sources using UDP[^1] with forward error correction. Multicasting allows a single NDI source to be delivered to multiple receivers by replicating the NDI packets from the sender to any number of receivers.&#x20;

{% hint style="warning" %}
Using multicast on a network that is not configured correctly can produce undesirable results and cripple network performance. For this reason, multicast sending is disabled by default.
{% endhint %}

For successful multicasting, IGMP[^2] is mandatory. IGMP allows the receiving NDI systems to request access to the sender. Without IGMP querying and snooping, Multicast traffic is treated the same as broadcast transmission, resulting in packet forwarding to all ports on the network. With IGMP snooping, multicast NDI traffic is forwarded only to the receivers that subscribe to the multicast NDI stream.

NDI subscribes to a multicast group and will unsubscribe when that stream is no longer needed. A routing querier on the network handles the management of multicast subscriptions.\
If multicast is enabled on the sender, the receiver is going to prioritize multicast receive over unicast.

While video and audio data are delivered to the network via multicast delivery, each receiver also connects to the sender via a unicast TCP connection for bi-directional communication of metadata (e.g., PTZ control, tally, etc.)

Basic multicast settings can be enabled in the NDI Access Manager on Windows and Mac machines.

<figure><img src="../.gitbook/assets/image (18).png" alt=""><figcaption><p>Screenshot of NDI Access Manager Tool</p></figcaption></figure>

In Linux, Multicast can be configured in the NDI configuration file located in the home directory of the effective user: "ndi-config.v1.json"

Here is the way to set up Multicast in the configuration file:

`},`\
&#x20;   `"multicast": {`\
&#x20;     `"send": {`\
&#x20;       `"ttl": 1,`\
&#x20;       `"enable": true,`\
&#x20;       `"netmask": "`<mark style="color:red;">`255.255.0.0`</mark>`",`\
&#x20;       `"netprefix": "`<mark style="color:red;">`239.255.0.0`</mark>`"`\
&#x20;     `}`

When multicast receiving is enabled, and a sender is available in the same local network, the receiver can negotiate for a multicast stream to be sent. If the sender is not on the same local network, this negotiation does not occur (since it could lead to a multicast stream being sent but never able to arrive at the receiver).

Suppose the network is correctly configured and can ensure a multicast stream to route reliably from a different network to the receiver’s local network. In that case, it is possible to specify the sender’s subnet in the “subnets” setting of the\
"ndi-config.v1.json" to allow multicast negotiation to occur.

`"multicast": {`\
&#x20; `"recv": {`\
&#x20;  `"enable": true,`\
&#x20;  `"subnets": [ "`<mark style="color:red;">`10.28.5.0/24`</mark>`", "`<mark style="color:red;">`10.28.4.0/24`</mark>`" ]`\
&#x20; `}`\
`}`

\
These settings pertain to the multicast NDI setting on this machine. The first setting determines whether multicast sending is enabled or not.

By default, multicast sending is disabled. Next is the IP address prefix and mask. In this example, multicast IP addresses will be chosen in the range `239.255.0.0 - 239.255.255.255`. NDI will attempt to use different multicast addresses to ensure that the streams can be filtered efficiently by the network adapter.

**NDI senders need a range of multicast addresses available.**

The TTL value controls how many “hops” the multicast sending traffic will take, allowing it to move outside the local network.

\
\


&#x20;





[^1]: UDP (User Datagram Protocol) is an alternative protocol to TCP that is used when reliable delivery of data packets is not required.

[^2]: Internet Group Management Protocol is the protocol used in IP multicasting that allows a host to report its multicast group membership to networked routers in order to receive data, messages, or content addressed to the designated multicast group.
