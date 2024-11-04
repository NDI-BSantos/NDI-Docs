# Frame Synchronization

{% hint style="info" %}
When using video, it is important to realize that often you are using different clocks for different parts of the signal chain.
{% endhint %}

Within NDI, the sender can send at the clock rate it wants, and the receiver will receive it at that rate. In many cases, however, the sender and receiver are extremely unlikely to share the _exact same_ clock rate. Bear in mind that computer clocks rely on crystals which -- while notionally rated for the same frequency -- are seldom truly identical.

For example, your sending computer might have an audio clock rated to operate at 48000 Hz. It might well actually run at 48001 Hz, or perhaps 47998 Hz, however. And similar variances affect receivers. While the differences appear miniscule, they accumulate -- causing audio sync to drift over time. A receiver may receive more samples than it plays back; or audible glitches can occur because too few audio samples are sent in each timespan. Naturally, the same problem affects video sources.

It is very common to address these timing discrepancies by having a "frame buffer" and displaying the most recently received video frame. Unfortunately, the deviations in clock-timing prevent this from being a perfect solution. Frequently, for example, video will appear to 'jitter' when the sending and receiving clocks are _almost_ aligned (which is the most common case).

A "time base corrector" (TBC) or frame-synchronizer for the video clock provides another mechanism to handle these issues. This approach uses hysteresis to determine the best time to either drop or insert a video frame to achieve smooth video playback (audio should be dynamically sampled with a high order resampling filter to adaptively track clocking differences).

It's quite difficult to develop something that is correct for all scenarios, so the NDI SDK provides an implementation to help you develop real time audio/video applications without assuming responsibility for the significant complexity involved. Another way to view what this component of the SDK does is to think of it as transforming 'push' sources (i.e., NDI sources in which the data is pushed from the sender to the receiver) into 'pull' sources, wherein the host application pulls the data down-stream. The frame-sync automatically tracks all clocks to achieve the best video and audio performance while doing so.

In addition to time-base correction operations the frame sync will also automatically detect and correct for timing jitter that might occur. This internally handles timing anomalies such as those caused by network, sender or receiver side timing errors related to CPU limitations, network bandwidth fluctuations, etc.

A very common application of the frame-synchronizer is to display video on screen timed to the GPU v-sync, in which case you should convert the incoming time-base to the time-base of the GPU. The following table lists some common scenarios in which you might want to use frame-synchronization:

| Scenario                                      | Recommendation                                                                                                                                                                                                                                                                   |
| --------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Video playback on screen or a multiviewer** | Yes – you want the clock to be synced with vertical refresh. On a multi-viewer you would have a frame-sync for every video source, then call all of them on each v-sync and redraw all sources at that time.                                                                     |
| **Audio playback through sound card**         | Yes – the clock should be synced with your sound card clock.                                                                                                                                                                                                                     |
| **Video mixing of sources**                   | Yes – all video input clocks need to be synced to your output video clock. You can take each of the video inputs and frame-synchronize them together.                                                                                                                            |
| **Audio mixing**                              | Yes – you want all input audio clocks to be brought into sync with your output audio clock. You would create a frame-synchronizer for each audio source and – when driving the output – call each one, asking for the correct number of samples and sample-rate for your output. |
| **Recording a single channel**                | No – you should record the signal in the raw form without any re-clocking.                                                                                                                                                                                                       |
| **Recording multiple channels**               | Maybe – If you want to sync some input channels to match a master clock so they can be ISO-edited, you might want a frame-sync for all sources _except one_ (allowing them all to be synchronized with a single channel).                                                        |

To create a frame synchronizer object, you will call the function below (that is based on an already instantiated NDI receiver from which it will get frames).

Once this receiver has been bound to a frame-sync, you should use it in order to recover video frames. You can continue to use the underlying receiver for other operations, such as tally, PTZ, metadata, etc. Remember, it remains your responsibility to destroy the receiver -- even when a frame-sync is using it (you should always destroy the receiver _after_ the `framesync` has been destroyed).

`NDIlib_framesync_instance_t NDIlib_framesync_create(NDIlib_recv_instance_t p_receiver);`

The frame-sync is destroyed with the corresponding call:

void NDIlib\_framesync\_destroy(NDIlib\_framesync\_instance\_t p\_instance);

To recover audio, the following function will pull audio samples from the frame-sync queue. This function will always return data immediately, inserting silence if no current audio data is present. You should call this at the rate that you want audio, and it will automatically use dynamic audio sampling to conform the incoming audio signal to the rate at which you are calling.

{% hint style="info" %}
Note that you have no obligation to ensure that your requested sample rate, channel count and number of samples match the incoming signal, and all combinations of conversions are supported.
{% endhint %}

Audio resampling is done with high order audio filters. Timecode and per frame metadata are inserted into the best possible audio samples. Also, if you specify the desired sample-rate as zero it will fill in the buffer (and audio data descriptor) with the original audio sample rate. And if you specify the channel count as zero, it will fill in the buffer (and audio data descriptor) with the original audio channel count.



```
void NDIlib_framesync_capture_audio(
        NDIlib_framesync_instance_t p_instance, // The frame sync instance
        NDIlib_audio_frame_v2_t* p_audio_data, // The destination audio buffer
        int sample_rate, // Your desired sample rate. 0 for "use source".
        int no_channels, // Your desired channel count. 0 for "use source".
        int no_samples // The number of audio samples that you wish to get.
);
```



The buffer returned is freed using the corresponding function:

```
void NDIlib_framesync_free_audio(
NDIlib_framesync_instance_t p_instance,
NDIlib_audio_frame_v2_t* p_audio_data
);
```

This function will pull video samples from the frame-sync queue. It will always immediately return a video sample by using time-based correction. You can specify the desired field type, which is then used to return the best possible frame.

{% hint style="info" %}
Field based frame-sync means that the frame-synchronizer attempts to match the fielded input phase with the frame requests so that you have the most correct possible field ordering on output.

The same frame can be returned multiple times if duplication is needed to match the timing criteria.
{% endhint %}



It is assumed that progressive video sources can i) correctly display either a field 0 or field 1, ii) that fielded sources can correctly display progressive sources, and iii) that the display of field 1 on a field 0 (or vice versa) should be avoided at all costs.

If no video frame has ever been received, this will return NDIlib\_video\_frame\_v2\_t as an empty (all zero) structure. This allows you to determine that there has not yet been any video and act accordingly (for instance you might want to display a constant frame output at a particular video format, or black).

```
void NDIlib_framesync_capture_video(
    NDIlib_framesync_instance_t p_instance, // The frame-sync instance
    NDIlib_video_frame_v2_t* p_video_data, // The destination video frame
    NDIlib_frame_format_type_e field_type // The frame type that you prefer
);
```

The buffer returned is freed using the corresponding function:

```
void NDIlib_framesync_free_video(
    NDIlib_framesync_instance_t p_instance,
    NDIlib_video_frame_v2_t* p_video_data
);
```



