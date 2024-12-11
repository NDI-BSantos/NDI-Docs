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

# Synchronization

&#x20;NDI transmitters and receivers do not require any specific synchronization method to connect and function. An NDI infrastructure can operate “sync-free”, avoiding the complexity and cost of network systems that support synchronization layers. However, for specific use cases, developers and hardware manufacturers have several approaches to achieve synchronization.  &#x20;

When processing multiple video streams in production workflows, it may be necessary to synchronize all video streams to the same frame rate, with their frame start times aligned.  &#x20;

For physical video sources (e.g., cameras or video output cards) that have a fundamental native video timebase, synchronization is achieved through genlocking the internal video timebase to a reference signal, typically a blackburst signal.&#x20;

For software-only implementations (e.g., character generators, DDR playback, graphics rendering engines) that lack an inherent internal video timebase, multiple platforms can maintain the same frame rate by locking system clocks using NTP, PTP, or similar protocols. However, even with locked system clocks, frame start times will not align across different applications since each application initializes its frame processing independently.  &#x20;

### NDI Genlock  &#x20;

NDI Genlock provides a solution for software applications that lack an internal video timebase by allowing them to use an NDI signal as a timing reference. The Genlock instance connects to any visible NDI sender on the network and synchronizes the frame rate and frame start times to the selected source. The NDI source can reside on a local network, a remote network, or in the cloud.  &#x20;

To ensure proper operation of an NDI source as a Genlock reference, the following conditions must be met:  &#x20;

#### NDI Version Compatibility

It is strongly recommended that the NDI source stream originates from NDI version 5 or higher, which includes significant enhancements for genlock support. Older NDI streams may not fully comply with NDI genlock operations.  &#x20;

#### Network Bandwidth &#x20;

The NDI source must have sufficient network bandwidth to provide the genlock stream to all NDI receivers. Configuring the source with multicast may help optimize bandwidth usage. &#x20;

#### Cross-Frame-Rate Locking

NDI Genlock supports cross-frame-rate locking, where, for instance, a 60Hz signal can synchronize to a 30Hz sender. However, this is not a recommended workflow and should be avoided if possible.&#x20;

#### Irregular Frame Streams

Some NDI sources, like the Test Pattern Generator and NDI Screen Capture, may not send a regular frame stream. These sources optimize CPU usage and save network bandwidth by skipping frames, making them unsuitable as genlock references.  &#x20;

#### Fallback Mechanism

If the NDI Genlock clock cannot successfully synchronize with the NDI sender, it will automatically fall back to using the system clock. While this fallback is not as precise, it allows video processing to continue with reasonable performance.  &#x20;

