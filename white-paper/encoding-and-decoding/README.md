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

# Encoding and Decoding

### SpeedHQ Compression

NDI uses compression to enable the transmission of many video streams across existing infrastructure, specifically discrete cosine transform (DCT), which converts video signals into elementary frequency components. This method of compression is commonly used in encoding formats and mezzanine codecs within the industry.

One of the most efficient codecs in existence, NDI achieves significantly better compression than many codecs that have been accepted for professional broadcast use. On a typical, modern Intel-based i7 processor, the codec can compress a video stream to the following benchmarks:

**The NDI codec's peak signal-to-noise ratio (PSNR) exceeds 70dB for typical video content.**

Uniquely and importantly, NDI is the first ever codec to provide multi-generational stability.\
This means there is no further loss once a video signal is compressed. As a practical example, generation 2 and generation 1000 of a decode-to-encode sequence would be identical.

The NDI codec is designed to run very fast and is largely implemented in hand-written assembly to ensure that the process of compressing video frames occurs as quickly as possible. Latency is both a factor of the network connection and the endpoint products. NDI has a technical latency of 16 video scan lines, although in practice, most implementations would be one field of latency. Hardware implementations can provide full end-to-end latency of within 8 scan lines.

### NDI HX

NDI is available in some devices and applications using a different compression codec than NDI High Bandwidth. This format is known as NDI HX. Devices using this NDI format will be labeled with the HX moniker. HX offers similar video quality at a much lower bit rate, which can be useful in contexts with limited bandwidth, like Fast Ethernet[^1] networks, Wi-Fi, or WAN[^2] connections.

NDI HX is commonly found in hardware devices, like PTZ cameras and mobile phones, but it is possible to have NDI HX in software applications as well. Software applications using NDI HX will leverage the GPU on the computer for enhanced encoding performance. For this reason, having a good GPU on the system is an advantage.

**There are two variations of NDI HX: NDI HX and NDI HX3**; both can be decoded by software applications thanks to the codec provided by the NDI Tools.

[^1]: Ethernet, standardized as IEEE 802.3, refers to a series of LAN (Local Area Network) technologies used to connect computers and other devices to a home or business network.\
    Ethernet is a physical and data link layer networking protocol that supports data transfer rates starting at 10 Mbps, typically over twisted pair cabling, but also fiber optic and coaxial cabling.

[^2]: WAN (Wide Area Network) is a network that spans a relatively broad geographical area, such as a state, region, or nation. WANs typically connect multiple smaller networks, such as LANs (Local Area Network) and MANs (Metropolitan Area Network). The Internet is an example of a WAN.
