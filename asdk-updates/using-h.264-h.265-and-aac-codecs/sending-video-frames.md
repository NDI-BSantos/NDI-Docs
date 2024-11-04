# Sending Video Frames

Because you are undertaking all compression of the video yourself, you should provide a full bandwidth and a preview bandwidth video stream. The full bandwidth stream may be any resolution and framerate; the preview bandwidth should be 640 pixels wide and square pixels (e.g., 640x360 pixels when at 16:9 image aspect ratio).

You will then need to submit these two frames separately to the SDK. The creation of the frames is identical to the audio example in the previous section. The following is an example of just one of the two required streams:

```
uint8_t* p_h264_data;
uint32_t h264_data_size;

// This is probably zero for non I-frames, but MUST be set of I-frames
uint8_t* p_h264_extra_data;
uint32_t h264_extra_data_size;

// Compute the total size of the structure
uint32_t packet_size = sizeof(NDIlib_compressed_packet_t) + h264_data_size +
h264_extra_data_size;

// Allocate the structure
NDIlib_compressed_packet_t* p_packet = (NDIlib_compressed_packet_t*)malloc(packet_size);

// Fill in the settings
p_packet->version = NDIlib_compressed_packet_t::version_0;
p_packet->fourCC = NDIlib_FourCC_type_H264;
p_packet->pts = 0; // These should be filled in correctly !
p_packet->dts = 0;
p_packet->flags = is_I_frame ? NDIlib_compressed_packet_t::flags_keyframe
    : NDIlib_compressed_packet_t::flags_none;
p_packet->data_size = h264_data_size;
p_packet->extra_data_size = h264_extra_data_size;

// Compute the pointer to the compressed h264 data, then copy the memory into place.
uint8_t* p_dst_h264_data = (uint8_t*)(1 + p_packet);
memcpy(p_dst_h264_data, p_h264_data, h264_data_size);

// Compute the pointer to the ancillary extra data
uint8_t* p_dst_extra_h264_data = p_dst_h264_data + h264_data_size;
memcpy(p_dst_extra_h264_data, p_h264_extra_data, h264_extra_data_size);
```

Having filled in the audio compressed data structures, fill in a video header as shown in the following example.

{% hint style="warning" %}
It is crucial that the value of the FourCC specifies whether this is the full or preview resolution streams.
{% endhint %}

```
// Create a regular NDI audio frame, but of compressed format
NDIlib_video_frame_v2_t video_frame;
video_frame.xres = 1920; // Must match H.264 data
video_frame.yres = 1080; // Must match H.264 data

// This must value is dependent on whether this is a full or preview
// stream. This example shows the full resolution stream, the preview.
// resolution stream would specify NDIlib_FourCC_type_H264_lowest_bandwidth
video_frame.FourCC = (NDIlib_FourCC_video_type_e)NDIlib_FourCC_type_H264_highest_bandwidth;

// Any reasonable sample rate is supported
video_frame.frame_rate_N = 60000;
video_frame.frame_rate_D = 1001;

// Any reasonable aspect ratio is supported
video_frame.picture_aspect_ratio = 16.0f/9.0f;

// We only currently allow
video_frame.frame_format_type = NDIlib_frame_format_type_progressive;

// Choose a good value, this is an example
video_frame.timecode = p_packet->pts;

// Set the data
video_frame.p_data = (uint8_t*)p_packet;
video_frame.data_size_in_bytes = packet_size;

// Metadata is supported
video_frame.p_metadata = "<Hello/>";

// Transmit the H.264 video data to NDI, async is fully supported.
// But read SDK description of buffer lifetimes.
NDIlib_send_send_video_async_v2(pSender, &video_frame);
```

A critical part of NDI is that you ask the SDK if you should insert an I-frame. This can be achieved by making a call to `NDIlib_send_is_keyframe_required`. This call returns true when an I-Frame should be inserted for a down-stream source to correctly decode the image without errors.

For instance, when there is a new NDI connection this function will return true; when a down-stream source has dropped a packet and can no longer decode the rest of the GOP, then this will be detected and return true, etc.

You are free to insert I-frames when you want based on your GOP requirements, but when this function returns true then you should issue an I-Frame at the next possible time. This is required functionality for a good user experience and a compliant NDI HX source. The stream validation (described later) will verify that this practice is followed.

Although you can determine your own bitrates for H.264 or H.265 streams, the NDI HX SDK will also provide guidance if you wish to call `NDIlib_send_get_target_frame_size`. When used with H.264 or H.265 FourCC's in the structures, this will provide a reasonable _average_ bitrate recommendation based on the selected resolution, framerate, etc. This provides estimates for all combinations of framerates and resolutions, but compliance is not required.

Please note that NDI is a real-time API, meaning that you should make every effort to pass off video and audio as they arrive in synchronization with each other. These are passed through the transmission layers downstream to the remote device with the lowest possible latency and may be received by sources that are observing one or both streams (when possible, bandwidth is only allocated for the used streams in transmission).

Audio and video are sent when they are passed to the API, allowing one or both streams to stop or start as needed at any layer. As a result, frames are not "held" to synchronize streams, resulting in higher performance.
