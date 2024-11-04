# Custom allocators

The Advanced SDK allows you to provide custom memory allocators for receiving audio and video frames either in compressed or uncompressed formats. This allows you to ask NDI to decompress or receive data into a buffer allocated by you own application.

Some possible use cases for this are:

* Decompress into your own memory buffers for use in inter-process memory sharing; when you might want one or more NDI inputs to exist in their own process.
* You wish to fill in buffers that might be more optimally accessible by dedicated hardware, including GPUs.
* Minimizing memory copies does to the internal structure of your own application.
* Allocating a buffer with memory alignment that matches your need.
* This allows you to enter your own video or audio stride, allowing you to have NDI provide buffers that closely match the needs of your own application.

{% hint style="info" %}
It is often assumed that decompressing into a GPU accessible buffer will yield improved performance, however often these buffers are allocated using write combining memory that is not commonly CPU cacheable. Often decoding into these buffers is slower than decoding them into a regular memory buffer and performing a memcpy that is specifically optimized for copying into write combining memory,

If you are going to simply use your own pooled memory allocator, it is unlikely that it will give you significant performance enhancements over what the NDI SDK already has implemented internally. The NDI SDK uses a lock-free memory pooling mechanism to offer very quick frame allocation.
{% endhint %}

It is important to know that the allocations will occur on a thread that is called by the NDI SDK, you should ensure that you return quickly from this allocation or free call if you wish to avoid video or audio stalling. Your code should be re-entrant since it might be called from multiple threads at once. The allocators can be changed at any time, including at run-time. When you provide an allocation and free callback, the free callback is called exactly once for every allocation call, even if the allocators are changed out. There are some error conditions under which it is possible that a frame will be allocated, but not returned by the SDK since there might be a network or data integrity problem. In this case the frame is simply freed using your customer de-allocator.

## **Video Allocators**

To replace a memory allocator, one should implement two functions with one to allocate a new video frame and set the stride, and one to free that frame when the SDK no longer needs it. If you are implementing a memory allocator for use with uncompressed video frames, you should check the requested frame format within the allocator and fill in the `p_buffer` and `line_stride_in_bytes` members with the `NDIlib_video_frame_v2_t` structure that is passed into the function. When the SDK is free with the frame the corresponding de-allocation function will be called and you should free the `p_buffer` member using whatever means you might need.

The `p_opaque` pointer that may be passed into the function setup is your own custom data that will then be passed each time that an allocator or de-allocator is called.

An example uncompressed video frame allocator might look as follows. Note that you need not always support allocating under all possible frame formats since your allocator will only be called for the formats specified when receiving video with the SDK. This example is provided to illustrate how all formats might be used.

```
bool video_custom_allocator(void* p_opaque, NDIlib_video_frame_v2_t* p_video_data)
{
    switch (p_video_data->FourCC) {
        case NDIlib_FourCC_video_type_UYVY:
            p_video_data->line_stride_in_bytes = p_video_data->xres * 2;
            p_video_data->p_data = (uint8_t*)malloc(
                p_video_data->line_stride_in_bytes * p_video_data->yres
            );
            break;
        case NDIlib_FourCC_video_type_UYVA:
            p_video_data->line_stride_in_bytes = p_video_data->xres * 2;
            p_video_data->p_data = (uint8_t*)malloc(
                /* UYVY */p_video_data->line_stride_in_bytes * p_video_data->yres +
                /* Alpha */p_video_data->line_stride_in_bytes / 2 * p_video_data->yres
            );
            break;
        case NDIlib_FourCC_video_type_P216:
            p_video_data->line_stride_in_bytes = p_video_data->xres * sizeof(uint16_t);
            p_video_data->p_data = (uint8_t*)malloc(
                /* Y */p_video_data->line_stride_in_bytes * p_video_data->yres +
                /* CbCr */p_video_data->line_stride_in_bytes * p_video_data->yres
            );
            break;
        case NDIlib_FourCC_video_type_PA16:
            p_video_data->line_stride_in_bytes = p_video_data->xres * sizeof(uint16_t);
            p_video_data->p_data = (uint8_t*)malloc(
                /* Y */p_video_data->line_stride_in_bytes * p_video_data->yres +
                /* CbCr */p_video_data->line_stride_in_bytes * p_video_data->yres +
                /* Alpha */p_video_data->line_stride_in_bytes * p_video_data->yres
            );
            break;
        case NDIlib_FourCC_video_type_BGRA:
        case NDIlib_FourCC_video_type_BGRX:
        case NDIlib_FourCC_video_type_RGBA:
        case NDIlib_FourCC_video_type_RGBX:
            p_video_data->line_stride_in_bytes = p_video_data->xres * 4;
            p_video_data->p_data = (uint8_t*)malloc(
                p_video_data->>line_stride_in_bytes*p_video_data->yres
            );
            break;
        default:
            // Error, not a supported FourCC
            p_video_data->line_stride_in_bytes = 0;
            p_video_data->p_data = NULL;
            return false;
    }
    return true; // Success.
}
bool video_custom_deallocator(void* p_opaque, const NDIlib_video_frame_v2_t* p_video_data)
{
    free(p_video_data->p_data);
    return true;
}
```

One may then simply assign the allocator for any receiver with:

```
NDIlib_recv_set_video_allocator(
    pNDI_recv, NULL,
    video_custom_allocator, video_custom_deallocator
);
```

If you wish to reset the video memory allocators, at any time you may simply pass in null pointers:

```
NDIlib_recv_set_video_allocator(pNDI_recv, NULL, NULL, NULL);
```

As a final note, although it is not needed that often, it is possible to use your own memory allocators to receive compressed video format if the NDI receiver is being specified to receive compressed data.

## **Audio Allocators**

Audio allocations are implemented almost identically to video allocations, simply with different FourCC codes. An example memory allocator for audio might look as follows:

```
bool audio_custom_allocator(void* p_opaque, NDIlib_audio_frame_v3_t* p_audio_data)
{
    // Allocate uncompressed audio
    switch (p_audio_data->FourCC) {
        case NDIlib_FourCC_audio_type_FLTP:
            p_audio_data->channel_stride_in_bytes = sizeof(float) * p_audio_data->no_samples;
            p_audio_data->p_data = (uint8_t*)malloc(
                p_audio_data->channel_stride_in_bytes * p_audio_data->no_channels
            );
            break;
        default:
            p_audio_data->channel_stride_in_bytes = 0;
            p_audio_data->p_data = nullptr;
            return false;
        }
        return true;
}
bool audio_custom_deallocator(void* p_opaque, const NDIlib_audio_frame_v3_t* p_audio_data)
{
    free(p_audio_data->p_data);
    return true;
}
```

One may then simply assign the allocator for any receiver with:

```
NDIlib_recv_set_audio_allocator(
    pNDI_recv, NULL,
    audio_custom_allocator, audio_custom_deallocator
);
```

If you wish to reset the video memory allocators, at any time you may simply pass in null pointers:

`NDIlib_recv_set_audio_allocator(pNDI_recv, NULL, NULL, NULL);`
