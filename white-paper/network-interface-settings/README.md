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

# Network Interface Settings

NDI is designed to enable successful video transport using the default configuration of network interface drivers; however, most recent network interface drivers do support the configuration of advanced properties that can help optimize NDI transmission.

Consider the following adjustments but note that adjusting individual adapters can significantly affect performance and reliability positively and negatively. It is important to consider testing performance with a network analyzer before and after each setting change. The following adjustments are intended to help; however, performance will depend on network and usage (names and available settings vary between vendors, adapter models, and even between different driver versions):

### Speed and Duplex

This setting allows for the selection of the desired speed and duplex of the network adapter. Usually, this is set to Auto Negotiation. If supported, this setting should be set to 1 Gbps Full Duplex or higher to ensure the maximum available throughput.

### Energy Efficient Ethernet

When enabled, this allows the adapter to engage power-saving features while keeping connections active. This technology uses the standard IEEE 802.3az to allow for less power during periods of low data activity. Adapters that utilize the IEEE 802.3az standard should have no impact on the performance of NDI; however, some integrated circuits exist that were developed before the standard was finalized or do not adhere to the standard at all. In these cases, it is best to disable the energy efficiency while determining the best network optimization.
