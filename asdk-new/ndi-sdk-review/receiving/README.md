# Receiving

### Receiving

NDI receivers are created in the same way as they would be in the NDI SDK, using NDIlib\_recv\_create\_v3, or the newer function available to the Advanced SDK, NDIlib\_recv\_create\_v4. As with senders, you provide the device name. It is strongly recommended that your device allow its name to be configured so that individual devices can be identified on the network. (This is not currently used by NDI applications; however it is anticipated that it will be in the future and your device will thus be future-proof.)

An example of creating a receiver follows below:

```
// The source to receive from (usually obtained from NDI_find_get_current_sources)
NDIlib_source_t recv_source;
recv_source.p_ndi_name = "Their name"; // The name of the NDI source
recv_source.p_url_address = NULL;
```

```
// Setup the structure describing the receiver
NDIlib_recv_create_v3_t recv_create;
recv_create.source_to_connect_to = recv_source;
recv_create.color_format = NDIlib_recv_color_format_compressed; // Compressed pass-through
recv_create.bandwidth = NDIlib_recv_bandwidth_highest; // If you want program quality video
recv_create.allow_video_fields = true; // Always true for pass-through
recv_create.p_ndi_name = "Your name"; // Often configured in your web page

const char* p_config_json = NULL; // You can override the default json file settings
NDIlib_recv_instance_t pRecv = NDIlib_recv_create_v4(&recv_create, p_config_json);
```

{% hint style="warning" %}
It is crucial that you provide an XML identification for all hardware devices. This should include your vendor name, model number, serial number and firmware version.
{% endhint %}

For example, the following XML identification would work (although it should be filled in with the correct settings for your device (serial numbers should be unique to each device manufactured):

<pre><code>NDIlib_metadata_frame_t NDI_product_type;
NDI_product_type.p_data = "&#x3C;ndi_product long_name=\NDILib Recv Example.\ "
<strong>    "     short_name=\NDILib Recv\ "
</strong>    "     manufacturer=\CoolCo, inc.\ "
    "     version=\1.000.000\ "
    "     session=\default\ "
    "     model_name=\S1\ "
    "     serial=\ABCDEFG\ />";
NDIlib_recv_add_connection_metadata(pRecv, &#x26;NDI_product_type);
</code></pre>



Note that you can create a receiver for the preview stream in addition to the program stream. Preview streams might be beneficial for picture-in-picture support, for example.

To do so, you would specify `NDIlib_recv_bandwidth_lowest` in the bandwidth field of the create struct, rather than `NDIlib_recv_bandwidth_highest`. You can capture audio with a program or preview stream receiver, or you can create a dedicated audio-only receiver and capture audio in a loop on another thread.

It is recommended that you also provide a "preferred" video format that you want to an up-stream device, since this has been commonly used by NDI applications. For instance, CG applications can be informed of a resolution you would like, and will automatically configure themselves to this resolution. An example might be as follows:

```
NDIlib_metadata_frame_t NDI_format_type;
NDI_format_type.p_data = "<ndi_format>"
    "    <video_format xres=\1920\ yres=\1080\ "
    "         frame_rate_n=\60000\ frame_rate_d=\1001\
    "         aspect_ratio=\1.77778\ progressive=\true\/>"
    "     <audio_format no_channels=\4\ sample_rate=\48000\/>"
    "</ndi_format>";
NDIlib_recv_add_connection_metadata(pSend, &NDI_format_type);
```

Here is a simple example of a loop capturing video only from the program stream receiver:

```
// Some code
```

while (true) {

// Capture a frame of video

NDIlib\_video\_frame\_v2\_t frame;

if (NDIlib\_frame\_type\_video == NDIlib\_recv\_capture\_v3(pRecv, \&frame, NULL, NULL, 100))

{

switch (frame.FourCC) {

case (NDIlib\_FourCC\_video\_type\_e)NDIlib\_FourCC\_type\_SHQ0\_highest\_bandwidth:

// Decode the frame.p\_data SpeedHQ 420 buffer here

break;

case (NDIlib\_FourCC\_video\_type\_e)NDIlib\_FourCC\_type\_SHQ2\_highest\_bandwidth:

// Decode the frame.p\_data SpeedHQ 422 buffer here

break;

case (NDIlib\_FourCC\_video\_type\_e)NDIlib\_FourCC\_type\_SHQ7\_highest\_bandwidth:

// Decode the frame.p\_data SpeedHQ 4224 buffer here

break;

}

NDIlib\_recv\_free\_video\_v2(pRecv, \&frame);

}

}

Hint: If desired, you could asynchronously capture and decode by moving decoding to another thread. Just don't forget to use NDIlib\_recv\_free\_video\_v2 to free the video buffers when you finish decoding.

The line\_stride\_in\_bytes field will be used to tell you about the size in bytes of the compressed video data.

#### Custom allocators

The Advanced SDK allows you to provide custom memory allocators for receiving audio and video frames either in compressed or uncompressed formats. This allows you to ask NDI to decompress or receive data into a buffer allocated by you own application.

Some possible use cases for this are:

* Decompress into your own memory buffers for use in inter-process memory sharing; when you might want one or more NDI inputs to exist in their own process.
* You wish to fill in buffers that might be more optimally accessible by dedicated hardware, including GPUs.
* Minimizing memory copies does to the internal structure of your own application.
* Allocating a buffer with memory alignment that matches your need.
* This allows you to enter your own video or audio stride, allowing you to have NDI provide buffers that closely match the needs of your own application.

> Note: It is often assumed that decompressing into a GPU accessible buffer will yield improved performance, however often these buffers are allocated using write combining memory that is not commonly CPU cacheable. Often decoding into these buffers is slower than decoding them into a regular memory buffer and performing a memcpy that is specifically optimized for copying into write combining memory,
>
> If you are going to simply use your own pooled memory allocator, it is unlikely that it will give you significant performance enhancements over what the NDI SDK already has implemented internally. The NDI SDK uses a lock-free memory pooling mechanism to offer very quick frame allocation.

It is important to know that the allocations will occur on a thread that is called by the NDI SDK, you should ensure that you return quickly from this allocation or free call if you wish to avoid video or audio stalling. Your code should be re-entrant since it might be called from multiple threads at once. The allocators can be changed at any time, including at run-time. When you provide an allocation and free callback, the free callback is called exactly once for every allocation call, even if the allocators are changed out. There are some error conditions under which it is possible that a frame will be allocated, but not returned by the SDK since there might be a network or data integrity problem. In this case the frame is simply freed using your customer de-allocator.

**Video Allocators**

To replace a memory allocator, one should implement two functions with one to allocate a new video frame and set the stride, and one to free that frame when the SDK no longer needs it. If you are implementing a memory allocator for use with uncompressed video frames, you should check the requested frame format within the allocator and fill in the p\_buffer and line\_stride\_in\_bytes members with the NDIlib\_video\_frame\_v2\_t structure that is passed into the function. When the SDK is free with the frame the corresponding de-allocation function will be called and you should free the p\_buffer member using whatever means you might need.

The p\_opaque pointer that may be passed into the function setup is your own custom data that will then be passed each time that an allocator or de-allocator is called.

An example uncompressed video frame allocator might look as follows. Note that you need not always support allocating under all possible frame formats since your allocator will only be called for the formats specified when receiving video with the SDK. This example is provided to illustrate how all formats might be used.

bool video\_custom\_allocator(void\* p\_opaque, NDIlib\_video\_frame\_v2\_t\* p\_video\_data)

{

switch (p\_video\_data->FourCC) {

case NDIlib\_FourCC\_video\_type\_UYVY:

p\_video\_data->line\_stride\_in\_bytes = p\_video\_data->xres \* 2;

p\_video\_data->p\_data = (uint8\_t\*)malloc(

p\_video\_data->line\_stride\_in\_bytes \* p\_video\_data->yres

);

break;

case NDIlib\_FourCC\_video\_type\_UYVA:

p\_video\_data->line\_stride\_in\_bytes = p\_video\_data->xres \* 2;

p\_video\_data->p\_data = (uint8\_t\*)malloc(

/\* UYVY \*/p\_video\_data->line\_stride\_in\_bytes \* p\_video\_data->yres +

/\* Alpha \*/p\_video\_data->line\_stride\_in\_bytes / 2 \* p\_video\_data->yres

);

break;

case NDIlib\_FourCC\_video\_type\_P216:

p\_video\_data->line\_stride\_in\_bytes = p\_video\_data->xres \* sizeof(uint16\_t);

p\_video\_data->p\_data = (uint8\_t\*)malloc(

/\* Y \*/p\_video\_data->line\_stride\_in\_bytes \* p\_video\_data->yres +

/\* CbCr \*/p\_video\_data->line\_stride\_in\_bytes \* p\_video\_data->yres

);

break;

case NDIlib\_FourCC\_video\_type\_PA16:

p\_video\_data->line\_stride\_in\_bytes = p\_video\_data->xres \* sizeof(uint16\_t);

p\_video\_data->p\_data = (uint8\_t\*)malloc(

/\* Y \*/p\_video\_data->line\_stride\_in\_bytes \* p\_video\_data->yres +

/\* CbCr \*/p\_video\_data->line\_stride\_in\_bytes \* p\_video\_data->yres +

/\* Alpha \*/p\_video\_data->line\_stride\_in\_bytes \* p\_video\_data->yres

);

break;

case NDIlib\_FourCC\_video\_type\_BGRA:

case NDIlib\_FourCC\_video\_type\_BGRX:

case NDIlib\_FourCC\_video\_type\_RGBA:

case NDIlib\_FourCC\_video\_type\_RGBX:

p\_video\_data->line\_stride\_in\_bytes = p\_video\_data->xres \* 4;

p\_video\_data->p\_data = (uint8\_t\*)malloc(

p\_video\_data->>line\_stride\_in\_bytes\*p\_video\_data->yres

);

break;

default:

// Error, not a supported FourCC

p\_video\_data->line\_stride\_in\_bytes = 0;

p\_video\_data->p\_data = NULL;

return false;

}

return true; // Success.

}

bool video\_custom\_deallocator(void\* p\_opaque, const NDIlib\_video\_frame\_v2\_t\* p\_video\_data)

{

free(p\_video\_data->p\_data);

return true;

}

One may then simply assign the allocator for any receiver with:

NDIlib\_recv\_set\_video\_allocator(

pNDI\_recv, NULL,

video\_custom\_allocator, video\_custom\_deallocator

);

If you wish to reset the video memory allocators, at any time you may simply pass in null pointers:

NDIlib\_recv\_set\_video\_allocator(pNDI\_recv, NULL, NULL, NULL);

As a final note, although it is not needed that often, it is possible to use your own memory allocators to receive compressed video format if the NDI receiver is being specified to receive compressed data.

**Audio Allocators**

Audio allocations are implemented almost identically to video allocations, simply with different FourCC codes. An example memory allocator for audio might look as follows:

bool audio\_custom\_allocator(void\* p\_opaque, NDIlib\_audio\_frame\_v3\_t\* p\_audio\_data)

{

// Allocate uncompressed audio

switch (p\_audio\_data->FourCC) {

case NDIlib\_FourCC\_audio\_type\_FLTP:

p\_audio\_data->channel\_stride\_in\_bytes = sizeof(float) \* p\_audio\_data->no\_samples;

p\_audio\_data->p\_data = (uint8\_t\*)malloc(

p\_audio\_data->channel\_stride\_in\_bytes \* p\_audio\_data->no\_channels

);

break;

default:

p\_audio\_data->channel\_stride\_in\_bytes = 0;

p\_audio\_data->p\_data = nullptr;

return false;

}

return true;

}

bool audio\_custom\_deallocator(void\* p\_opaque, const NDIlib\_audio\_frame\_v3\_t\* p\_audio\_data)

{

free(p\_audio\_data->p\_data);

return true;

}

One may then simply assign the allocator for any receiver with:

NDIlib\_recv\_set\_audio\_allocator(

pNDI\_recv, NULL,

audio\_custom\_allocator, audio\_custom\_deallocator

);

If you wish to reset the video memory allocators, at any time you may simply pass in null pointers:

NDIlib\_recv\_set\_audio\_allocator(pNDI\_recv, NULL, NULL, NULL);
