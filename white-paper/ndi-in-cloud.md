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

# NDI in Cloud

Setting up an NDI-based video production in a Virtual Private Cloud is quite easy; the first step is to define how to make NDI Discovery and Registration work in a VPC. Cloud providers allow the creation of a multicast domain[^1]; multicast is required to use mDNS-based discovery and registration. This setup requires the creation of a transit gateway with multicast enabled. Enabling multicast in the cloud might require specific knowledge; for this reason, the easiest solution to enable NDI Discovery and registration is to set up a Discovery Service. NDI Discovery Service requires just a basic Windows or Linux-based instance to run.

[^1]: A domain refers to a LAN subnetwork of users, systems, devices, and servers. Domain can also refer to the IP address of a website on the Internet.
