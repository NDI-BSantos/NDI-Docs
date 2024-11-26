# NDI Sending On High Latency Connections

When you are sending NDI data in compressed form (e.g., with `NDIlib_send_send_video_scatter_async` or `NDIlib_send_send_video_async`) to the SDK then the data is transmitted from your buffers without any memory copies. The rules on buffer ownership are that your buffers may be accessed by the SDK until the next asynchronous call. In effect, this means that there will always be a single outstanding NDI compressed send at any time.

On a high latency network, or a network with high packet loss, it can take some significant time to be sure that the data being sent has been received by the other side. Because this round-trip-time might be 200ms or more, the NDI SDK needs to have access to your buffer in case any of that data needs to be resent (remember we are trying to work without any memory copies). Since there can only be one outstanding frame being sent by the NDI SDK at once time, when using networks of this kind it might cause poor frame sending performance because we might need to wait for one entire round-trip time to allow the second asynchronous frame to proceed and signal that the buffer is no longer in access by the library.

To best achieve high performance on high latency networks, starting SDK version 5.0.8 the behavior of NDI sending of compressed data has been updated, and the following recommendations apply to sending data.

* When sending compressed frames, the NDI libraries will make a copy of the frame so that the caller is not exposed to the round-trip time delays that might slow down the ability to submit frames to the SDK.
* When one specifies an async completion callback using `NDIlib_send_set_video_async_completion` then the SDK will send without any memory copies. When in this mode there will be any number of allowed frames in flight and you will receive a call-back when they are no longer needed.
