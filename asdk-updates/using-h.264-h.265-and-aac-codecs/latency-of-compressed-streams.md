# Latency of Compressed Streams

It is very important that you look very closely at your streams and ensure that they are configured for low latency decoding which is non-trivial and many "off the shelf" H.264 and HEVC encoders do not provide without changing their settings. If an encoder has a low latency mode, then this is often a very good starting point. Other areas to look at are ensuring that you are using only I and P frames so that each incoming frame can be immediately decoded without a delay. In addition, it is very common that different codecs place "video delay" settings in the SPS NAL or SEI NAL units in the stream that instruct a decoder to delay the display of frames which introduces latency. In our experience, almost no HX implementation has been correctly configured for lowest latency use by default and work has always been needed to optimize the stream settings.

We have tools that can help measure the decode latency imposed by the stream if you wish.
