---
description: >-
  This section provides some guidelines on how to get the best performance out
  of the SDK.
---

# Performance and Implementation

### Upgrading Your Applications

The libraries (DLLs) for the latest version of NDI should be entirely backward compatible with NDI v4 and, to the degree possible, even earlier versions. In many cases, you should be able to simply update these in your application to get most of the benefits of the new version – without a single code change.

{% hint style="info" %}
Note: There are several exceptions to the statement above, however. If you have software that was built before NDI 4.5 on macOS and Linux applications, you will need to recompile your applications; this change was made because the size of some struct members changed.
{% endhint %}

### General Recommendations

* Throughout the system, use YCbCr color, if possible, as it offers both higher performance and better quality.
* If your system has more than one NIC and you are using more than a few senders and receivers, it is worth connecting all available ports to the network. Bandwidth will be distributed across multiple network adapters.
* Use the latest version of the SDK whenever possible. Naturally, the experience of huge numbers of NDI users in the field provides numerous minor edge cases, and we work hard to resolve all of these as quickly as possible.
* We also have ambitious plans for the future of NDI and IP video, and we are continually laying the groundwork for these in each new version so that these will already be in place when the related enhancements become available for public use.
* The SDK is designed to take advantage of the latest CPU instructions available, particularly AVX2 (256-bit instructions) on Intel platforms and NEON instructions on ARM platforms. Generally, NDI speed limitations relate more to system memory bandwidth than CPU processing performance since the code is designed to keep all execution pipelines on a CPU busy.
* NDI takes advantage of multiple CPU cores when decoding and encoding one or more streams and, for higher resolutions, will use multiple cores to decode a single stream.

### Sending Video

* Use UYVY or UYVA color, if possible, as this avoids internal color conversions. If you cannot generate these color formats and you would use the CPU to perform the conversion, it is better to let the SDK perform the conversion.
* Doing so yields performance benefits in most cases, particularly when using asynchronous frame submission. If the data that you are sending to NDI is on the GPU, and you can have the GPU perform the color conversion before downloading it to system memory, you are likely to find that this has the best performance.
* Sending BGRA or BGRX video will incur a performance penalty. This is caused by the increased memory bandwidth required for these formats and the conversion to YCbCr color space for compression. With that said, performance was significantly improved in **NDI 4**.
* Using asynchronous frame submission almost always yields significant performance benefits.&#x20;

### Receiving Video

* Using `NDIlib_recv_color_format_fastest` for receiving will yield the best performance.
* Having separate threads query for audio and video via `NDIlib_recv_capture_v3` is recommended.

{% hint style="info" %}
Note that `NDIlib_recv_capture_v3` is multi-thread safe, allowing multiple threads to be waiting for data at once. Using a reasonable timeout on `NDIlib_recv_capture_v3` is better and more efficient than polling it with zero time-outs.
{% endhint %}

* In the modern versions of NDI, there are internal heuristics that attempt to guess whether hardware acceleration would enable better performance. That said, it is possible to explicitly enable hardware acceleration if you believe that it would be beneficial for your application. This can be enabled by sending an XML metadata message to a receiver as follows:

`<ndi_video_codec type="hardware"/>`

* Bear in mind that decoding resources on some machines are designed for processing a single video stream. In consequence, while hardware assistance might benefit some small number of streams, it may hurt performance as the number of streams increases.
* Modern versions of NDI almost certainly already default to using hardware acceleration in most situations where it would be beneficial, and so these settings are not likely to make a significant improvement. In earlier versions, concerns around hardware codec driver stability made us less likely to enable these by default, but we believe that this caution is no longer needed.

### Reliable UDP with Multi-stream Congestion Control <a href="#reliable-udp" id="reliable-udp"></a>

In **NDI 5**, the default communication mechanism is a reliable UDP protocol that represents the state-of-the-art communication protocol that is implemented by building upon all the experience we have seen in the real world with NDI across a massive variety of different installations. By using UDP, it does not rely on any direct round-trip, congestion control, or flow control issues that are typically associated with TCP. Most importantly, our observation has been that on real-world networks, as you approach the total bandwidth available across many different channels of video, the management of the network of bandwidth becomes the most common problem. Our reliable UDP solves this by moving all streams between sources into a single connection across which the congestion control is applied in aggregate to all streams at once, which represents a massive improvement in almost all installations. These streams are all entirely non-blocking with each other so that even under high packet loss situations, there is no possibility of a particular loss impacting any other portions of the connection.

One of the biggest problems with UDP packet sending is the need to send many small packets onto the network, which results in a very large OS kernel overhead. Our implementation works around this by supporting fully asynchronous sending of many packets at once and supporting packet coalescing on the receiving side to allow the kernel and network card driver to offload a huge fraction of the processing.

This protocol also supports high network latencies by having the current best congestion control systems published. This allows many packets to be in flight at a time and very accurately tracks which packets have been received and adjusts the number to the currently measured round-trip time.

On Windows, where this is supported, receiver-side scaling is used to achieve very optimized network receiving that ensures that, as the network card interrupts are received, they are always handed off directly to a thread that has an execution context available (which are then directly passed into the NDI processing chain).

For the absolute best performance, reliable UDP supports UDP Segmentation Offload (USO), which allows network interface cards to offload the segmentation of UDP datagrams that are larger than the maximum transmission unit (MTU) of the network medium, which can significantly reduce CPU usage.

On Linux, to get the best performance, we need to take advantage of Generic Segmentation Offload (GSO) for UDP sending, which might also be referred to as UDP\_SEGMENT, which was made available in Linux Kernel 4.18. Without this, UDP sending can see significantly increased CPU overhead.

### Multicast

NDI supports multicast-based video sources using multicast UDP with forward error correction to correct for packet loss.

It is important to be aware that using multicast on a network that is not configured correctly is very similar to a “denial of service” attack on the entire network; for this reason, multicast sending is disabled by default.

Every router that we have tested has treated multicast traffic as if it was broadcast traffic by default. Because most multicast traffic on a network is low bandwidth, this is of little consequence, and generally allows a network router to run more efficiently because no packet filtering is required.

What this means, though, is that every multicast packet received is sent to every destination on the network, regardless of whether it was needed there or not. Because NDI requires high bandwidth multicast, even with a limited number of sources on a large network, the burden of sending this much data to all network sources can cripple the entire network’s performance.

To avoid this _serious_ problem, it is essential to ensure that _every_ router on the network has proper multicast filtering enabled. This option is most referred to as “[**IGMP snooping**](#user-content-fn-1)[^1]”. If you are unable to find a way to enable this option, we recommend that you use multicast NDI with all due caution.

#### Debugging with Multicast

Another important cautionary note is that a software application like NDI will subscribe to a multicast group and will unsubscribe from it when it no longer needs that group.

Unlike most operations in the operating system, the un-subscription step is not automated by the OS; once you are subscribed to a group, your computer will continue to receive data until the router sends an IGMP query to verify whether it is still needed. This happens about every 5 minutes on typical networks.

The result is that if you launch an NDI multicast stream and kill your application _without closing the NDI connection correctly_, your computer will continue to receive the data from the network until this timeout expires.

[^1]: This topic is described in detail at [wikipedia](https://en.wikipedia.org/wiki/IGMP\_snooping).&#x20;
