# Sending

NDI senders are created in exactly same way that they would be in the NDI SDK, using `NDIlib_send_create` or the newer function available to the Advanced SDK, `NDIlib_send_create_v2`.

{% hint style="warning" %}
It is strongly recommended that your device allow its name to be configured, so that individual devices can be identified on the network.
{% endhint %}

You are also able to specify that your device exists within different NDI groups should you desire.

An example of creating a sender might be:

```
// Setup the structure describing the sender
NDIlib_send_create_t send_create;
send_create.clock_audio = false; // Your audio is probably clocked by your own hardware.
send_create.clock_video = false; // Your video is probably clocked by your own hardware.
send_create.p_ndi_name = "Your name"; // Often configured in your web page.
send_create.p_groups = NULL; // You can allow this to be configured in your web page.
const char *p_config_json = NULL; // You can override the default json file settings
NDIlib_send_instance_t pSend = NDIlib_send_create_v2(&send_create, p_config_json);
```

{% hint style="warning" %}
It is crucial that you provide XML identification for all hardware devices. This should include your vendor's name, model number, serial number and firmware version.
{% endhint %}

For example, the following XML identification would work, although it should be filled in with the correct settings for your device (serial numbers should be unique to each device manufactured):

```
NDIlib_metadata_frame_t NDI_product_type;
NDI_product_type.p_data = "<ndi_product long_name=\NDILib Send Example.\ "
" short_name=\NDILib Send\ "
" manufacturer=\CoolCo, inc.\ "
" version=\1.000.000\ "
" session=\default\ "
" model_name=\S1\ "
" serial=\ABCDEFG\ />";
NDIlib_send_add_connection_metadata(pNDI_send, &NDI_product_type);
```

You are now going to be able to pass compressed frames directly to the SDK. The following is a very basic example of how this might be configured.

NDIlib\_video\_frame\_v2\_t video\_frame;

```
video_frame.xres = 1920;
video_frame.yres = 1080;
video_frame.frame_format_type = NDIlib_frame_format_type_progressive;
video_frame.FourCC = (NDIlib_FourCC_video_type_e)NDIlib_FourCC_type_SHQ2_highest_bandwidth;
video_frame.picture_aspect_ratio = 16.0f/9.0f;
video_frame.frame_rate_N = 60000;
video_frame.frame_rate_D = 1001;
video_frame.timecode = /* A timestamp from your hardware at 100 ns clock rate */;
video_frame.p_data = /* Your compressed data */;
video_frame.line_stride_in_bytes = /* The size of your compressed data in bytes */;
NDIlib_send_send_video_v2(pSend, &video_frame);
```

{% hint style="warning" %}
It is important to always submit both a program stream and a preview stream to the SDK. The program stream should be the full resolution video stream. The preview stream should always be progressive, have its longest dimension as 640 pixels, and a framerate that does not exceed 45 Hz.
{% endhint %}

* It is considered acceptable to simply drop every second frame to achieve the correct preview framerate when needed.
* It is also acceptable to always scale down by an integer scaling factor to be close to the correct resolution. (Scaling quality is not defined, although high quality scaling is preferred.)

Examples of the preview stream are listed below:

| Main Video Format         | Preview Video Format    |
| ------------------------- | ----------------------- |
| 1920x1080, 16:9, 59.94 Hz | 640x360, 16:9, 29.97 Hz |
| 1920x1080, 16:9, 29.97 Hz | 640x360, 16:9, 29.97 Hz |
| 1080x1920, 9:16, 50 Hz    | 360x640, 9:16, 25 Hz    |

To submit a preview resolution frame, one would use the following header (compare with previous example).

```
NDIlib_video_frame_v2_t video_frame;
video_frame.xres = 640;
video_frame.yres = 360;
video_frame.frame_format_type = NDIlib_frame_format_type_progressive;
video_frame.FourCC = (NDIlib_FourCC_video_type_e)NDIlib_FourCC_type_SHQ2_lowest_bandwidth;
video_frame.picture_aspect_ratio = 16.0f/9.0f;
video_frame.frame_rate_N = 30000;
video_frame.frame_rate_D = 1001;
video_frame.timecode = /* A timestamp from your hardware at 100 ns clock rate */;
video_frame.p_data = /* Your compressed data */;
NDIlib_send_send_video_v2(pSend, &video_frame);
```

It is common on Advanced SDK devices that you wish to have the SDK send frames without needing to wait for them to be complete. The asynchronous operations supported by the NDI SDK have been fully extended to the NDI Advanced SDK, and we recommend that these are used for best hardware performance.

The Advanced SDK has been designed to perform zero memory copy sending of frames over the network when using async operations. The SDK will assume that it can access each async buffer until either a) the next time that `NDIlib_send_send_video_async_v2` is called or b) the sender is closed.

Sending and receiving low and high bandwidth frames are entirely asynchronous with each other. It is also possible to send main, preview, and audio streams from separate threads, should this be beneficial.

For instance, the following might be used as a send loop:

```
NDIlib_video_frame_v2_t frame_main, frame_prvw;
while (true) {
    // Get the next frames to send
    NDIlib_video_frame_v2_t new_frame_main = get_frame_main();
    NDIlib_video_frame_v2_t new_frame_prvw = get_frame_prvw();
    // Send the frames
    NDIlib_send_send_video_async_v2(pSend, &new_frame_main);
    NDIlib_send_send_video_async_v2(pSend, &new_frame_prvw);
    // The previous frames are now guaranteed to no longer be needed by the SDK
    release_frame_main(&frame_main);
    release_frame_main(&frame_prvw);
    // These are now the next frames to use
    frame_main = new_frame_main;
    frame_prvw = new_frame_prvw;
}
```



Sending audio is nearly identical to sending video. The following example shows how to submit an audio buffer:

```
NDIlib_audio_frame_v3_t audio_frame;
audio_frame.sample_rate = 48000;
audio_frame.no_channels = 4;
audio_frame.no_samples = 1920;
audio_frame.FourCC = /* Fill in with the correct value */;
audio_frame.p_data = /* Fill in with the correct value */;
audio_frame.channel_stride_in_bytes = /* Fill in with the correct value */;
audio_frame.timecode = /* Fill in with a 100 ns timestamp from your device */;
NDIlib_send_send_audio_v3(pSend, &audio_frame);
```

{% hint style="warning" %}
It is crucial to ensure that your audio is provided to the SDK at the correct audio level, with a +4 dBU sinewave corresponding to a floating-point signal from -1.0 to +1.0. These levels can easily be debugged using the NDI Studio Monitor application.
{% endhint %}

The video bitrate should be controlled by your compressor by varying the Q level. You may determine this by calling the following function, which will return the size of the expected frame in bytes.

int NDIlib\_send\_get\_target\_frame\_size(

NDIlib\_send\_instance\_t p\_instance,

const NDIlib\_video\_frame\_v2\_t\* p\_video\_data

);

There are many possible mechanisms available to perform bitrate control, and it is expected that you are within 10% of the returned value. (It is understood that it is often impossible to adapt the Q value for the frame being compressed and that the update will often occur on the subsequent frame.)

The examples above show how video may be sent using NDIlib\_send\_send\_video\_v2.  By default, the SDK will not copy the memory buffers of video frames that are passed in (other than implicit copies required by your operating system for network sending events).  There may be times when the video frame is composed of multiple pieces instead of a single contiguous block of memory.  For the best performance, you can use one of the two scatter-gather sending functions provided to handle frames that are broken up into many pieces, either NDIlib\_send\_send\_video\_scatter, or NDIlib\_send\_send\_video\_scatter\_async.

The NDIlib\_send\_send\_video\_scatter\_async function will allow you to schedule entirely asynchronous sends. For this purpose, please review the comments above on buffer ownership and lifetime, which are the single most common problems reported due to incorrect SDK usage.

To work with high performance on high latency connections we very strongly recommend that you implement asynchronous sending completions, which are outlined in the next two sections. These allow data to be sent with NDI that does not require any memory-copies to send onto the network, while also allowing enough frames to be "in flight" that one can achieve high performance even on high latency networks.

Sending video is currently supported in 4:2:2:4, 4:2:2, and 4:2:0 formats (4:4:4:4 will be added in a future version; this is already internally supported by the software SDK). It has been our experience that that 4:2:0 yields better video quality at resolutions above 1920x1080, since more bits are assigned in the luminance.

#### Asynchronous Sending Completions

When using asynchronous sending, the default way that it works is that you provide a buffer to the call and ownership of that buffer is held by the SDK until the return of the next asynchronous sending call. While this allows fully asynchronous sending, a problem that can occur is that if multiple receivers are connected to a source and one of those receivers is not responsive (either because it has become ungracefully disconnected or because its connection is slow) then sending a new frame via an asynchronous send might lock until sending to that network receiver is complete, causing potential video delays or freezes.

It is possible to assign an asynchronous completion handler for an NDI sender. When this is assigned, each call to an asynchronous sending function like NDIlib\_send\_send\_video\_async\_v2 and NDIlib\_send\_send\_video\_scatter\_async will always return immediately and the frame submitted will be sent to all connections that are currently actively receiving. When the buffer is no longer needed by the SDK the completion routine is called which tells you that you may now free this buffer. Note that while it is rate, if a connection is stalled and holds onto a buffer until the connection times-out that there is a chance that completions are called out of order.

One can assign a completion handler for asynchronous sending calls, with opaque data that is passed into the completion routine with:

void NDIlib\_send\_set\_video\_async\_completion(

NDIlib\_send\_instance\_t p\_instance,

void\* p\_opaque,

NDIlib\_video\_send\_async\_completion\_t p\_deallocator

);

If one passes in a NULL as a completion callback pointer, then the behavior is returned to the default buffer ownership behavior which will lock on a send until the previous call has been asynchronously sent.

It is important to know that the completion will occur on a thread that is called by the NDI SDK, you should ensure that you return quickly from this allocation or free call if you wish to avoid video or audio stalling. Your code should be re-entrant since it might be called from multiple threads at once and should avoid any permanent locks that might occur between sending frames and completion handlers. The handler can be changed out at any time, including at run-time. When you provide a completion handler, the callback is called exactly once for every asynchronous sending call.

When a connection is closed (using NDIlib\_send\_destroy), all outstanding completions will be called before the destroy event is complete.

#### NDI Sending On High Latency Connections

When you are sending NDI data in compressed form (e.g., with NDIlib\_send\_send\_video\_scatter\_async or NDIlib\_send\_send\_video\_async) to the SDK then the data is transmitted from your buffers without any memory copies. The rules on buffer ownership are that your buffers may be accessed by the SDK until the next asynchronous call. In effect, this means that there will always be a single outstanding NDI compressed send at any time.

On a high latency network, or a network with high packet loss, it can take some significant time to be sure that the data being sent has been received by the other side. Because this round-trip-time might be 200ms or more, the NDI SDK needs to have access to your buffer in case any of that data needs to be resent (remember we are trying to work without any memory copies). Since there can only be one outstanding frame being sent by the NDI SDK at once time, when using networks of this kind it might cause poor frame sending performance because we might need to wait for one entire round-trip time to allow the second asynchronous frame to proceed and signal that the buffer is no longer in access by the library.

To best achieve high performance on high latency networks, starting SDK version 5.0.8 the behavior of NDI sending of compressed data has been updated, and the following recommendations apply to sending data.

* When sending compressed frames, the NDI libraries will make a copy of the frame so that the caller is not exposed to the round-trip time delays that might slow down the ability to submit frames to the SDK.
* When one specifies an async completion callback using NDIlib\_send\_set\_video\_async\_completion then the SDK will send without any memory copies. When in this mode there will be any number of allowed frames in flight and you will receive a call-back when they are no longer needed.
