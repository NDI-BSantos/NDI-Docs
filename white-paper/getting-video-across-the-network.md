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

# Getting video across the network

Video, just like voice data in VoIP systems, is a very demanding data stream and will immediately expose a weakness in a network. The network must support multiple video, audio, and data streams in a reliable, synchronized manner without disruption. When delay, packet loss, and jitter reach thresholds where the video is impacted visually, the usefulness of that video drops to zero. It is important to understand the complexities of video in IP data networks to mitigate these factors.

Networks that are designed to move NDI video streams should be thought of as being primarily utilized for video. IP networks are, by their very nature, “best effort delivery” systems and were originally developed for the transport of data. By contrast to video, data services can function happily with packet retransmissions, lost packets, and even packets arriving out of order.

Video streams, while still composed of data, are much more rigid in their requirements. With the use of modern networking equipment and proper configuration, video can move across networks whilst still obtaining low latency, frame accuracy, and high-quality requirements necessary for live video production.
