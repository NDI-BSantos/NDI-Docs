# Receiving

### Receiving

NDI receivers are created in the same way as they would be in the NDI SDK, using `NDIlib_recv_create_v3`, or the newer function available to the Advanced SDK, `NDIlib_recv_create_v4`. As with senders, you provide the device name. It is strongly recommended that your device allow its name to be configured so that individual devices can be identified on the network. (This is not currently used by NDI applications; however it is anticipated that it will be in the future and your device will thus be future-proof.)

An example of creating a receiver follows below:

```
// The source to receive from (usually obtained from NDI_find_get_current_sources)
NDIlib_source_t recv_source;
recv_source.p_ndi_name = "Their name"; // The name of the NDI source
recv_source.p_url_address = NULL;

// Setup the structure describing the receiver
NDIlib_recv_create_v3_t recv_create;
recv_create.source_to_connect_to = recv_source;
recv_create.color_format = NDIlib_recv_color_format_compressed; // Compressed pass-through
recv_create.bandwidth = NDIlib_recv_bandwidth_highest; // If you want program quality video
recv_create.allow_video_fields = true;                 // Always true for pass-through
recv_create.p_ndi_name = "Your name";                  // Often configured in your web page

const char* p_config_json = NULL; // You can override the default json file settings
NDIlib_recv_instance_t pRecv = NDIlib_recv_create_v4(&recv_create, p_config_json);
```

{% hint style="warning" %}
It is crucial that you provide an XML identification for all hardware devices. This should include your vendor name, model number, serial number and firmware version.
{% endhint %}

For example, the following XML identification would work (although it should be filled in with the correct settings for your device (serial numbers should be unique to each device manufactured):

```
NDIlib_metadata_frame_t NDI_product_type; 
NDI_product_type.p_data = "<ndi_product long_name=\"NDILib Recv Example.\" "
                          "             short_name=\"NDILib Recv\" "
                          "             manufacturer=\"CoolCo, inc.\" "
                          "             version=\"1.000.000\" "
                          "             session=\"default\" "
                          "             model_name=\"S1\" "
                          "             serial=\"ABCDEFG\" />";
NDIlib_recv_add_connection_metadata(pRecv, &NDI_product_type);
```



Note that you can create a receiver for the preview stream in addition to the program stream. Preview streams might be beneficial for picture-in-picture support, for example.

To do so, you would specify `NDIlib_recv_bandwidth_lowest` in the bandwidth field of the create struct, rather than `NDIlib_recv_bandwidth_highest`. You can capture audio with a program or preview stream receiver, or you can create a dedicated audio-only receiver and capture audio in a loop on another thread.

It is recommended that you also provide a "preferred" video format that you want to an up-stream device, since this has been commonly used by NDI applications. For instance, CG applications can be informed of a resolution you would like, and will automatically configure themselves to this resolution. An example might be as follows:

```
NDIlib_metadata_frame_t NDI_format_type; 
NDI_format_type.p_data = "<ndi_format>"
                         "  <video_format xres=\"1920\" yres=\"1080\" "
                         "                frame_rate_n=\"60000\" frame_rate_d=\"1001\" 
                         "                aspect_ratio=\"1.77778\" progressive=\"true\"/>"
                         "   <audio_format no_channels=\"4\" sample_rate=\"48000\"/>"
                         "</ndi_format>";

NDIlib_recv_add_connection_metadata(pSend, &NDI_format_type);
```

Here is a simple example of a loop capturing video only from the program stream receiver:

```
while (true) {
    // Capture a frame of video
    NDIlib_video_frame_v2_t frame;
    if (NDIlib_frame_type_video == NDIlib_recv_capture_v3(pRecv, &frame, NULL, NULL, 100))
    {
        switch (frame.FourCC) {
            case (NDIlib_FourCC_video_type_e)NDIlib_FourCC_type_SHQ0_highest_bandwidth:
                // Decode the frame.p_data SpeedHQ 420 buffer here
                break;
            case (NDIlib_FourCC_video_type_e)NDIlib_FourCC_type_SHQ2_highest_bandwidth:
                // Decode the frame.p_data SpeedHQ 422 buffer here
                break;
            case (NDIlib_FourCC_video_type_e)NDIlib_FourCC_type_SHQ7_highest_bandwidth:
                // Decode the frame.p_data SpeedHQ 4224 buffer here
                break;
        }

        NDIlib_recv_free_video_v2(pRecv, &frame);
    }
}
```

{% hint style="info" %}
&#x20;If desired, you could asynchronously capture and decode by moving decoding to another thread. Just don't forget to use NDIlib\_recv\_free\_video\_v2 to free the video buffers when you finish decoding.
{% endhint %}

The `line_stride_in_bytes` field will be used to tell you about the size in bytes of the compressed video data.
