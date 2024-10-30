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

NDI transmitters or receivers do not need any synchronization method to be connected and work. **An NDI infrastructure can perfectly work “sync free”** without the complexity and cost of network infrastructure that supports a synchronization layer. However, with NDI, software developers and hardware manufacturers have several ways to approach synchronization:

### NDI Advanced SDK – Genlock API

When processing multiple video streams in a complex production environment it is frequently desirable to have all video streams operating at exactly the same frame rate with frame start times synchronized across all streams being processed. For physical video sources having a fundamental native video timebase (eg: a camera or video output card) this is handled by genlocking the internal video timebase to a reference signal, typically blackburst. Software only implementations with no inherent internal video timebase (eg: character generator, DDR playback, graphics rendering engine, etc.) across multiple platforms can run at exactly the same frame rate if system clocks are locked (via NTP, PTP, or other means) but frame start times will not be synchronized across different applications, as each individual application will start their frames at different points in time.

NDI genlock supports using an NDI signal as a timing reference for these software applications which otherwise lack an internal video timebase. The NDI genlock instance can connect to any NDI sender visible on the network and uses the timing from this NDI sender to correctly time and pace the sending of video frames, insuring both the overall framerate and the individual frame start times remain consistent with the selected NDI source. The NDI source used as a timing reference can be on the local network, a remote network, or in-cloud.

For an NDI source to correctly be able to operate as an NDI Genlock, it is important to bear in mind a couple of key ingredients as outlined below.

* It is very strongly recommended that the NDI source is a stream from NDI version 5 which has been significantly improved to support genlock capabilities. It is possible that some NDI streams from previous versions are not fully compliant with how NDI genlock operates.
* It is important that the source has enough network bandwidth to drive a reduced bandwidth signal to all the NDI genlock instances. Some Advanced SDK NDI converters might fail in this regard in which case an NDI Proxy may be used to relay the signal (and might also be used to relay cloud genlock as well). Configuring this source for multicast might also help, although multicast often is hard to fully support.
* NDI genlock is very robust and supports correct cross-frame-rate locking. For instance, a sender might be 30Hz and you are genlocking a 60Hz signal to it. This is however not a recommended workflow where it can be avoided.
* Some NDI sources like Test Pattern generator and NDI Screen Capture do not always send a regular stream of frames. They do this to save network bandwidth and CPU time. Sources such as these cannot be used as a basis for genlock.
* Remember that when creating NDI senders that you wish to use with genlock that you specify the `use_clock` values as false when the senders are created. The NDI genlock functions replace the system clock pacing provided by the NDI library when `use_clock` is set to true.
* If the genlock clock cannot correctly genlock to an NDI sender for some reason it will fall back to using the system clock and so can continue to work reasonably.
* Since there is some (low) overhead associated with each genlock instance it is recommended that you only have one for each source that you wish to lock too.

To create an NDI genlock instance, you use the function `NDIlib_genlock_create`, you may specify the NDI source that you wish to use as a reference and the NDI JSON settings associated with it. It is important that you remember to fill out the `vendor_id` in the JSON or this sender will have the same limitation as NDI receivers. It is legal to create an NDI genlock that has a null source name and then later use the `NDIlib_genlock_connect` function to change the connection being used. Like all other NDI SDK functions, a genlock object can be disposed of with the function `NDIlib_genlock_destroy`.

If you wish to change the NDI source that is being used for the genlock, one can call the function `NDIlib_genlock_connect`. If the parameter is nullptr then the NDI source will be disconnected and the genlock operation will fall back to using the system clock.

The `NDIlib_genlock_is_active` call may be used to determine if a particular NDI source is correctly operating as a genlock signal. If the NDI sender that is being used as the genlock source is not currently sending an active signal, then this will return false. Note that the functions to perform the clocking operation return whether the genlock is active and so this function need not be polled within a sending loop and is provided as a convenience.

Once you have created an NDI genlock instance and have it locked to an NDI receiver, then one may simply call the functions `NDIlib_genlock_wait_video` and `NDIlib_genlock_wait_audio`. These functions will wait until it is time for the next video or audio frame to be delivered and then return so that you can send it. While these structures take in a full frame descriptor, only the minimum number of members are used and so the rest need not be filled in. For video a `NDIlib_video_frame_v2_t` is used, however only the `frame_rate_N`, `frame_rate_D` and `frame_format_type` members need be valid. For audio a `NDIlib_audio_frame_v3_t` is used, however only the `no_samples` and `sample_rate` need be valid. These functions return a bool value which will tell you whether the source is currently genlocked or whether there is no signal and so the system clock has been used for timing.

The audio and video waiting functions are entirely thread safe and you may have a separate thread for the timing and of video and audio as needed.

### NDI Advanced SDK – Timestamp, AV Sync API

NDI transmitters and receivers utilize the system clock as a point of reference. This system clock can be aligned with robust sources like NTP or PTP. The timing details from these sources are integrated into the NDI Stream through timestamps embedded in each video and audio frame (with audio frames being smaller than video frames).

Subsequently, the receiver can realign audio and video sources using this timestamp information. This method provides an ingenious approach to achieving synchronization, making it an ideal solution for remote or cloud-based production scenarios.
