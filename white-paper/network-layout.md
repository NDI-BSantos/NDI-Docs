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

# Network Layout

NDI is designed for use with standard consumer off-the-shelf (COTS) network infrastructure devices. Looking closely at the network topology and configuration will help to ensure that the maximum possible bandwidth is available.

When selecting a network switch, it is important to check the throughput speeds. Ensure that each port is full duplex (i.e., bi-directional communication) and that each port's upstream and downstream data speeds are at least 1 Gigabit per second (Gbps). In some cases, it is best to force the ports on managed switches to utilize 1 Gbps in contrast with Auto-Negotiation. The use of Auto-Negotiation can sometimes (mostly because non fully compatible Network Interfaces) result in 100Mb connections or even lower, which does not renegotiate until the port is flooded with traffic for some time. Also, poor termination of RJ-45 connectors can impact Auto-Negotiation.

The same suggestion applies when considering network switches that include 10 Gigabit per second ports. Many switches manufactured at the time of writing may share bandwidth across the backplane of multiple ports. Since these ports are generally reserved for linking to other switches, the specification for throughput may be listed differently than the Gigabit port section in the product documentation.

It is best to use switches from the same manufacturer, or ideally, the same model of switch, throughout a single subnet. This will simplify configuration and lessen the chances of compatibility and configuration issues.
