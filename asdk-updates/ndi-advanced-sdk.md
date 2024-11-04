# NDI Advanced SDK

## Overview

This section of the NDI Advanced SDK is designed for use by device manufacturers who wish to provide hardware assisted encoding or decoding. It follows the same API as the NDI SDK so that any experience, sample code, documentation from one can be applied to the other.

Importantly, this part of the NDI Advanced SDK provides direct access to the video data in compressed form so that it can be sent and received directly. Thus, it is also likely that this SDK can be used for other tasks that require or benefit from interacting directly with the compressed video data for sending and receiving.

There are currently two primary uses of the NDI Advanced SDK.

1.  You can use NDI compression (sometimes called "SpeedHQ"), a high performance and high quality I-Frame video codec. An FPGA compressor for native NDI is provided within this SDK.

    If you are using SpeedHQ compression, you are likely also using a (SoC) device with an ARM core and an FPGA that can be used for real-time compression. The NDI Advanced SDK provides NDI Encode and Decode IP cores for Xilinx and Altera FPGAs, along with example FPGA projects and reference C++ applications including full source code.

    To help get you started, prebuilt bootable uSD drive images for several standard Development kits are provided. The FPGA core supplied will easily encode 4K video in real-time on relatively modest FPGA designs, assuming the device has sufficient memory bandwidth (multiple banks of RAM are recommended); on latest generation SoC systems it can easily encode one or more streams of 8K if sufficient network bandwidth is provided.
2. You may use H.264 or H.265 for video and AAC audio with this SDK if you are developing using devices that already have hardware compressors for these available.

Because many Advanced SDK systems require custom tool chains, NDI can provide a compiled version of the SDK expressly built for your specific system. Please email [sdk@ndi.video](mailto:sdk@ndi.video) if you have such requirements.

## Configuration Files

All default settings for NDI® are controlled through a configuration file. This file is located at $HOME/.ndi/ndi-config.v1.json. It contains settings for multicast and unicast sending as well as vendor information. Your vendor ID is specified here and must be registered with NDI for compressed data pass-through to be enabled.

The configuration file will automatically be loaded off disk by default and these settings used. If you wish to work this way, you can simply restart your application when configuration settings on the device are changed, and the settings will be updated.

Alternatively, the NDI creation functions can be passed in a string representation of this configuration file, allowing you to change it for senders and receivers by simply recreating them at run time.

There is a setting for p\_config\_json that may be set in creation of devices to permit an in-memory version of the configuration file to be passed in. An example showing how to set p\_config\_json, if that is your preferred method of specifying the configuration, follows below.

const char \*p\_config\_json ="{"

"\ndi\\: {"

"\vendor\\: {"

"\name\\: \CoolCo, inc.\\,"

"\id\\: \00000000-0000-0000-0000-000000000000\\"

"}"

"}"

"}";

The ability to specify per connection settings for any sender, finder and receiver is a very powerful ability since it allows every connection to be completely customized per use, allowing for instance different NICs to be used for different connection types, multicast to be manually specified by connection, different groups and much more.

The full details of the configuration file are provided in the manual under the section on _Performance and Implementation Details_.

## NDI SDK Review

This section provides a brief introduction to using the NDI® SDK and demonstrates how to create and use a sender and receiver. While this is likely enough to get started developing applications, there are many additional examples and documentation in the SDK.

### Sending

NDI senders are created in exactly same way that they would be in the NDI SDK, using NDIlib\_send\_create or the newer function available to the Advanced SDK, NDIlib\_send\_create\_v2.

Note: It is strongly recommended that your device allow its name to be configured, so that individual devices can be identified on the network.

You are also able to specify that your device exists within different NDI groups should you desire.

An example of creating a sender might be:

// Setup the structure describing the sender

NDIlib\_send\_create\_t send\_create;

send\_create.clock\_audio = false; // Your audio is probably clocked by your own hardware.

send\_create.clock\_video = false; // Your video is probably clocked by your own hardware.

send\_create.p\_ndi\_name = "Your name"; // Often configured in your web page.

send\_create.p\_groups = NULL; // You can allow this to be configured in your web page.

const char \*p\_config\_json = NULL; // You can override the default json file settings

NDIlib\_send\_instance\_t pSend = NDIlib\_send\_create\_v2(\&send\_create, p\_config\_json);

It is crucial that you provide XML identification for all hardware devices. This should include your vendor's name, model number, serial number and firmware version.

For example, the following XML identification would work, although it should be filled in with the correct settings for your device (serial numbers should be unique to each device manufactured):

NDIlib\_metadata\_frame\_t NDI\_product\_type;

NDI\_product\_type.p\_data = "\<ndi\_product long\_name=\NDILib Send Example.\ "

" short\_name=\NDILib Send\ "

" manufacturer=\CoolCo, inc.\ "

" version=\1.000.000\ "

" session=\default\ "

" model\_name=\S1\ "

" serial=\ABCDEFG\ />";

NDIlib\_send\_add\_connection\_metadata(pNDI\_send, \&NDI\_product\_type);

You are now going to be able to pass compressed frames directly to the SDK. The following is a very basic example of how this might be configured.

NDIlib\_video\_frame\_v2\_t video\_frame;

video\_frame.xres = 1920;

video\_frame.yres = 1080;

video\_frame.frame\_format\_type = NDIlib\_frame\_format\_type\_progressive;

video\_frame.FourCC = (NDIlib\_FourCC\_video\_type\_e)NDIlib\_FourCC\_type\_SHQ2\_highest\_bandwidth;

video\_frame.picture\_aspect\_ratio = 16.0f/9.0f;

video\_frame.frame\_rate\_N = 60000;

video\_frame.frame\_rate\_D = 1001;

video\_frame.timecode = /\* A timestamp from your hardware at 100 ns clock rate \*/;

video\_frame.p\_data = /\* Your compressed data \*/;

video\_frame.line\_stride\_in\_bytes = /\* The size of your compressed data in bytes \*/;

NDIlib\_send\_send\_video\_v2(pSend, \&video\_frame);

It is important to always submit both a program stream and a preview stream to the SDK. The program stream should be the full resolution video stream. The preview stream should always be progressive, have its longest dimension as 640 pixels, and a framerate that does not exceed 45 Hz.

* It is considered acceptable to simply drop every second frame to achieve the correct preview framerate when needed.
* It is also acceptable to always scale down by an integer scaling factor to be close to the correct resolution. (Scaling quality is not defined, although high quality scaling is preferred.)

Examples of the preview stream are listed below:

\+-------------------------------+--------------------------------------+ | Main Video Format | Preview Video Format | +===============================+======================================+ | 1920x1080, 16:9, 59.94 Hz | 640x360, 16:9, 29.97 Hz | +-------------------------------+--------------------------------------+ | 1920x1080, 16:9, 29.97 Hz | 640x360, 16:9, 29.97 Hz | +-------------------------------+--------------------------------------+ | 1080x1920, 9:16, 50 Hz | 360x640, 9:16, 25 Hz | +-------------------------------+--------------------------------------+

To submit a preview resolution frame, one would use the following header (compare with previous example).

NDIlib\_video\_frame\_v2\_t video\_frame;

video\_frame.xres = 640;

video\_frame.yres = 360;

video\_frame.frame\_format\_type = NDIlib\_frame\_format\_type\_progressive;

video\_frame.FourCC = (NDIlib\_FourCC\_video\_type\_e)NDIlib\_FourCC\_type\_SHQ2\_lowest\_bandwidth;

video\_frame.picture\_aspect\_ratio = 16.0f/9.0f;

video\_frame.frame\_rate\_N = 30000;

video\_frame.frame\_rate\_D = 1001;

video\_frame.timecode = /\* A timestamp from your hardware at 100 ns clock rate \*/;

video\_frame.p\_data = /\* Your compressed data \*/;

NDIlib\_send\_send\_video\_v2(pSend, \&video\_frame);

It is common on Advanced SDK devices that you wish to have the SDK send frames without needing to wait for them to be complete. The asynchronous operations supported by the NDI SDK have been fully extended to the NDI Advanced SDK, and we recommend that these are used for best hardware performance.

The Advanced SDK has been designed to perform zero memory copy sending of frames over the network when using async operations. The SDK will assume that it can access each async buffer until either a) the next time that NDIlib\_send\_send\_video\_async\_v2 is called or b) the sender is closed.

Sending and receiving low and high bandwidth frames are entirely asynchronous with each other. It is also possible to send main, preview, and audio streams from separate threads, should this be beneficial.

For instance, the following might be used as a send loop:

NDIlib\_video\_frame\_v2\_t frame\_main, frame\_prvw;

while (true) {

// Get the next frames to send

NDIlib\_video\_frame\_v2\_t new\_frame\_main = get\_frame\_main();

NDIlib\_video\_frame\_v2\_t new\_frame\_prvw = get\_frame\_prvw();

// Send the frames

NDIlib\_send\_send\_video\_async\_v2(pSend, \&new\_frame\_main);

NDIlib\_send\_send\_video\_async\_v2(pSend, \&new\_frame\_prvw);

// The previous frames are now guaranteed to no longer be needed by the SDK

release\_frame\_main(\&frame\_main);

release\_frame\_main(\&frame\_prvw);

// These are now the next frames to use

frame\_main = new\_frame\_main;

frame\_prvw = new\_frame\_prvw;

}

Sending audio is nearly identical to sending video. The following example shows how to submit an audio buffer:

NDIlib\_audio\_frame\_v3\_t audio\_frame;

audio\_frame.sample\_rate = 48000;

audio\_frame.no\_channels = 4;

audio\_frame.no\_samples = 1920;

audio\_frame.FourCC = /\* Fill in with the correct value \*/;

audio\_frame.p\_data = /\* Fill in with the correct value \*/;

audio\_frame.channel\_stride\_in\_bytes = /\* Fill in with the correct value \*/;

audio\_frame.timecode = /\* Fill in with a 100 ns timestamp from your device \*/;

NDIlib\_send\_send\_audio\_v3(pSend, \&audio\_frame);

It is \[crucial]{.underline} to ensure that your audio is provided to the SDK at the correct audio level, with a +4 dBU sinewave corresponding to a floating-point signal from -1.0 to +1.0. These levels can easily be debugged using the NDI Studio Monitor application.

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

### Receiving

NDI receivers are created in the same way as they would be in the NDI SDK, using NDIlib\_recv\_create\_v3, or the newer function available to the Advanced SDK, NDIlib\_recv\_create\_v4. As with senders, you provide the device name. It is strongly recommended that your device allow its name to be configured so that individual devices can be identified on the network. (This is not currently used by NDI applications; however it is anticipated that it will be in the future and your device will thus be future-proof.)

An example of creating a receiver follows below:

// The source to receive from (usually obtained from NDI\_find\_get\_current\_sources)

NDIlib\_source\_t recv\_source;

recv\_source.p\_ndi\_name = "Their name"; // The name of the NDI source

recv\_source.p\_url\_address = NULL;

// Setup the structure describing the receiver

NDIlib\_recv\_create\_v3\_t recv\_create;

recv\_create.source\_to\_connect\_to = recv\_source;

recv\_create.color\_format = NDIlib\_recv\_color\_format\_compressed; // Compressed pass-through

recv\_create.bandwidth = NDIlib\_recv\_bandwidth\_highest; // If you want program quality video

recv\_create.allow\_video\_fields = true; // Always true for pass-through

recv\_create.p\_ndi\_name = "Your name"; // Often configured in your web page

const char\* p\_config\_json = NULL; // You can override the default json file settings

NDIlib\_recv\_instance\_t pRecv = NDIlib\_recv\_create\_v4(\&recv\_create, p\_config\_json);

It is crucial that you provide an XML identification for all hardware devices. This should include your vendor name, model number, serial number and firmware version.

For example, the following XML identification would work (although it should be filled in with the correct settings for your device (serial numbers should be unique to each device manufactured):

NDIlib\_metadata\_frame\_t NDI\_product\_type;

NDI\_product\_type.p\_data = "\<ndi\_product long\_name=\NDILib Recv Example.\ "

" short\_name=\NDILib Recv\ "

" manufacturer=\CoolCo, inc.\ "

" version=\1.000.000\ "

" session=\default\ "

" model\_name=\S1\ "

" serial=\ABCDEFG\ />";

NDIlib\_recv\_add\_connection\_metadata(pRecv, \&NDI\_product\_type);

Note that you can create a receiver for the preview stream in addition to the program stream. Preview streams might be beneficial for picture-in-picture support, for example.

To do so, you would specify NDIlib\_recv\_bandwidth\_lowest in the bandwidth field of the create struct, rather than NDIlib\_recv\_bandwidth\_highest. You can capture audio with a program or preview stream receiver, or you can create a dedicated audio-only receiver and capture audio in a loop on another thread.

It is recommended that you also provide a "preferred" video format that you want to an up-stream device, since this has been commonly used by NDI applications. For instance, CG applications can be informed of a resolution you would like, and will automatically configure themselves to this resolution. An example might be as follows:

NDIlib\_metadata\_frame\_t NDI\_format\_type;

NDI\_format\_type.p\_data = "\<ndi\_format>"

" \<video\_format xres=\1920\ yres=\1080\ "

" frame\_rate\_n=\60000\ frame\_rate\_d=\1001\\

" aspect\_ratio=\1.77778\ progressive=\true\\/>"

" \<audio\_format no\_channels=\4\ sample\_rate=\48000\\/>"

"\</ndi\_format>";

NDIlib\_recv\_add\_connection\_metadata(pSend, \&NDI\_format\_type);

Here is a simple example of a loop capturing video only from the program stream receiver:

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

### Finding

As documented elsewhere, the NDI filter allows you to locate all NDI sources on the network. When using a discovery server, the Advanced SDK allows you to specify per connection metadata by specifying a "source.metadata" field in the sender JSON (see documentation for configuration files). When using an NDI finder one can receive the list of all sources on the network with their associated metadata using the function:

const NDIlib\_source\_v2\_t\* NDIlib\_find\_get\_current\_sources\_v2(

NDIlib\_find\_instance\_t p\_instance, uint32\_t\* p\_no\_sources

);

This function returns the list of sources, including their metadata in a fashion that is identical to the regular NDIlib\_find\_get\_current\_sources function.

### Video Formats

Decoding is more complex than encoding, because you do not get to specify what format you are sent. It is recommended that you support as many possible video formats as you wish, although if this is not possible (e.g., non-video resolutions) you can either scale, or simply provide a place-holder image. It is important that you support the three possible video formats (4:2:0, 4:2:2, 4:2:2:4) since these are in common use. If you cannot process the alpha channel, it is recommended that you multiply the image against black.

It is also important to understand that it is the NDI sender that determines the video and audio clock rates. A simple framebuffer is not sufficient to smoothly display audio and video without glitches.

#### Receiver Codec Support Level

When creating an NDI receiver to receive compressed data, it is very important to specify the color\_format field correctly on the NDIlib\_recv\_create\_v3\_t structure. The following table will list all the available values introduced with the Advanced SDK and what the values mean. If you specify a value but do not know how to handle certain frame types, it is very important that you check the FourCC of the frame and discard accordingly.

Audio for all formats, except the ones with the with\_audio suffix, will be delivered in floating-point format. If the NDI source is sending AAC audio, the NDI library will attempt to decompress the audio frame and return that as floating-point format.

\+---------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | color\_format value | Description | +===================================================+==================================================================================================================================================================================================================================================================================================================================================================================================================================================================+ | NDIlib\_recv\_color\_format\_compressed | This value has the same meaning as NDIlib\_recv\_color\_format\_compressed\_v1. | +---------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | NDIlib\_recv\_color\_format\_compressed\_v1 | When connected to an NDI source that is sending SpeedHQ video, the compressed frames will be delivered to you. This mode assumes you only know how to handle SpeedHQ frames and no other format, not even uncompressed. If the NDI source sends any other video compression format frames will not be delivered to you, nor will there be an attempt to decompress the frames in the NDI library. | +---------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | NDIlib\_recv\_color\_format\_compressed\_v2 | When connected to an NDI source that is sending SpeedHQ video, the compressed frames will be delivered to you. This mode assumes you only know how to handle SpeedHQ frames and uncompressed frames, but no other format. If the NDI source is sending any other compression format, the frames will be delivered in UYVY format if the NDI library could decompress it. | +---------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | NDIlib\_recv\_color\_format\_compressed\_v2\_best | This value has the same meaning as NDIlib\_recv\_color\_format\_compressed\_v2, however, the list of uncompressed formats that can be delivered has been expanded to include 16-bit formats such as P216. | +---------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | NDIlib\_recv\_color\_format\_compressed\_v3 | When connected to an NDI source that is sending SpeedHQ video or H.264 video, the compressed frames will be delivered to you. This mode assumes you know how to handle SpeedHQ, H.264, and uncompressed frames, but no other format. If the NDI source is sending any other video compression format, they will be delivered to you in UYVY format if the NDI library can decompress it. | +---------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | NDIlib\_recv\_color\_format\_compressed\_v3\_best | This value has the same meaning as NDIlib\_recv\_color\_format\_compressed\_v3, however, the list of uncompressed formats that can be delivered has been expanded to include 16-bit formats such as P216. | +---------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | NDIlib\_recv\_color\_format\_compressed\_v3\_with\_audio | This value has the same meaning as NDIlib\_recv\_color\_format\_compressed\_v3 but allows AAC audio frames to be passed to your layer without the NDI library decompressing them. If the NDI source is not sending AAC audio, then you will receive audio in floating-point format. | +---------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | NDIlib\_recv\_color\_format\_compressed\_v4 | When connected to an NDI source that is sending SpeedHQ, H.264 or H.265 video, the compressed frames will be delivered to you. This mode assumes you know how to handle SpeedHQ, H.264, H.265, and uncompressed frames, but no other format. If the NDI source is sending any other video compression format, they will be delivered to you in UYVY format if the NDI library can decompress it. | +---------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | NDIlib\_recv\_color\_format\_compressed\_v4\_best | This value has the same meaning as NDIlib\_recv\_color\_format\_compressed\_v4, however, the list of uncompressed formats that can be delivered has been expanded to include 16-bit formats such as P216. | +---------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | NDIlib\_recv\_color\_format\_compressed\_v4\_with\_audio | This value has the same meaning as NDIlib\_recv\_color\_format\_compressed\_v4 but allows AAC audio frames to be passed to your layer without the NDI library decompressing them. If the NDI source is not sending AAC audio, then you will receive audio in floating-point format. | +---------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | NDIlib\_recv\_color\_format\_compressed\_v5 | When connected to an NDI source that is sending SpeedHQ, H.264/H.265 video or H.264/H.265 video with alpha, the compressed frames will be delivered to you. This mode assumes you know how to handle SpeedHQ, H.264, H.265, H.264 with alpha, H.265 with alpha and uncompressed frames, but no other format. If the NDI source is sending any other video compression format, they will be delivered to you in UYVY format if the NDI library can decompress it. | +---------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | NDIlib\_recv\_color\_format\_compressed\_v5\_best | This value has the same meaning as NDIlib\_recv\_color\_format\_compressed\_v5, however, the list of uncompressed formats that can be delivered has been expanded to include 16-bit formats such as P216. | +---------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | NDIlib\_recv\_color\_format\_compressed\_v5\_with\_audio | This value has the same meaning as NDIlib\_recv\_color\_format\_compressed\_v5 but allows AAC or OPUS audio frames to be passed to your layer without the NDI library decompressing them. If the NDI source is not sending AAC or OPUS audio, then you will receive audio in floating-point format. | +---------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

#### Frame Synchronization

Note: When using video, it is important to realize that often you are using different clocks for different parts of the signal chain.

Within NDI, the sender can send at the clock rate it wants, and the receiver will receive it at that rate. In many cases, however, the sender and receiver are extremely unlikely to share the _exact same_ clock rate. Bear in mind that computer clocks rely on crystals which -- while notionally rated for the same frequency -- are seldom truly identical.

For example, your sending computer might have an audio clock rated to operate at 48000 Hz. It might well actually run at 48001 Hz, or perhaps 47998 Hz, however. And similar variances affect receivers. While the differences appear miniscule, they accumulate -- causing audio sync to drift over time. A receiver may receive more samples than it plays back; or audible glitches can occur because too few audio samples are sent in each timespan. Naturally, the same problem affects video sources.

It is very common to address these timing discrepancies by having a "frame buffer" and displaying the most recently received video frame. Unfortunately, the deviations in clock-timing prevent this from being a perfect solution. Frequently, for example, video will appear to 'jitter' when the sending and receiving clocks are _almost_ aligned (which is the most common case).

A "time base corrector" (TBC) or frame-synchronizer for the video clock provides another mechanism to handle these issues. This approach uses hysteresis to determine the best time to either drop or insert a video frame to achieve smooth video playback (audio should be dynamically sampled with a high order resampling filter to adaptively track clocking differences).

It's quite difficult to develop something that is correct for all scenarios, so the NDI SDK provides an implementation to help you develop real time audio/video applications without assuming responsibility for the significant complexity involved. Another way to view what this component of the SDK does is to think of it as transforming 'push' sources (i.e., NDI sources in which the data is pushed from the sender to the receiver) into 'pull' sources, wherein the host application pulls the data down-stream. The frame-sync automatically tracks all clocks to achieve the best video and audio performance while doing so.

In addition to time-base correction operations the frame sync will also automatically detect and correct for timing jitter that might occur. This internally handles timing anomalies such as those caused by network, sender or receiver side timing errors related to CPU limitations, network bandwidth fluctuations, etc.

A very common application of the frame-synchronizer is to display video on screen timed to the GPU v-sync, in which case you should convert the incoming time-base to the time-base of the GPU. The following table lists some common scenarios in which you might want to use frame-synchronization:

\+-------------------------------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | Scenario | Recommendation | +:==========================================+=====================================================================================================================================================================================================================================================================================+ | Video playback on screen or a multiviewer | Yes -- you want the clock to be synced with vertical refresh. On a multi-viewer you would have a frame-sync for every video source, then call all of them on each v-sync and redraw all sources at that time. | +-------------------------------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | Audio playback through sound card | Yes -- the clock should be synced with your sound card clock. | +-------------------------------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | Video mixing of sources | Yes -- all video input clocks need to be synced to your output video clock. You can take each of the video inputs and frame-synchronize them together. | +-------------------------------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | Audio mixing | Yes -- you want all input audio clocks to be brought into sync with your output audio clock. You would create a frame-synchronizer for each audio source and -- when driving the output -- call each one, asking for the correct number of samples and sample-rate for your output. | +-------------------------------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | Recording a single channel | No -- you should record the signal in the raw form without any re-clocking. | +-------------------------------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | Recording multiple channels | Maybe -- If you want to sync some input channels to match a master clock so they can be ISO-edited, you might want a frame-sync for all sources _except one_ (allowing them all to be synchronized with a single channel). | +-------------------------------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

To create a frame synchronizer object, you will call the function below (that is based on an already instantiated NDI receiver from which it will get frames).

Once this receiver has been bound to a frame-sync, you should use it in order to recover video frames. You can continue to use the underlying receiver for other operations, such as tally, PTZ, metadata, etc. Remember, it remains your responsibility to destroy the receiver -- even when a frame-sync is using it (you should always destroy the receiver _after_ the framesync has been destroyed).

NDIlib\_framesync\_instance\_t NDIlib\_framesync\_create(NDIlib\_recv\_instance\_t p\_receiver);

The frame-sync is destroyed with the corresponding call:

void NDIlib\_framesync\_destroy(NDIlib\_framesync\_instance\_t p\_instance);

To recover audio, the following function will pull audio samples from the frame-sync queue. This function will always return data immediately, inserting silence if no current audio data is present. You should call this at the rate that you want audio, and it will automatically use dynamic audio sampling to conform the incoming audio signal to the rate at which you are calling.

Note that you have no obligation to ensure that your requested sample rate, channel count and number of samples match the incoming signal, and all combinations of conversions are supported.

Audio resampling is done with high order audio filters. Timecode and per frame metadata are inserted into the best possible audio samples. Also, if you specify the desired sample-rate as zero it will fill in the buffer (and audio data descriptor) with the original audio sample rate. And if you specify the channel count as zero, it will fill in the buffer (and audio data descriptor) with the original audio channel count.

void NDIlib\_framesync\_capture\_audio(

NDIlib\_framesync\_instance\_t p\_instance, // The frame sync instance

NDIlib\_audio\_frame\_v2\_t\* p\_audio\_data, // The destination audio buffer

int sample\_rate, // Your desired sample rate. 0 for "use source".

int no\_channels, // Your desired channel count. 0 for "use source".

int no\_samples // The number of audio samples that you wish to get.

);

The buffer returned is freed using the corresponding function:

void NDIlib\_framesync\_free\_audio(

NDIlib\_framesync\_instance\_t p\_instance,

NDIlib\_audio\_frame\_v2\_t\* p\_audio\_data

);

This function will pull video samples from the frame-sync queue. It will always immediately return a video sample by using time-based correction. You can specify the desired field type, which is then used to return the best possible frame.

Note that:

* Field based frame-sync means that the frame-synchronizer attempts to match the fielded input phase with the frame requests so that you have the most correct possible field ordering on output.
* The same frame can be returned multiple times if duplication is needed to match the timing criteria.

It is assumed that progressive video sources can i) correctly display either a field 0 or field 1, ii) that fielded sources can correctly display progressive sources, and iii) that the display of field 1 on a field 0 (or vice versa) should be avoided at all costs.

If no video frame has ever been received, this will return NDIlib\_video\_frame\_v2\_t as an empty (all zero) structure. This allows you to determine that there has not yet been any video and act accordingly (for instance you might want to display a constant frame output at a particular video format, or black).

void NDIlib\_framesync\_capture\_video(

NDIlib\_framesync\_instance\_t p\_instance, // The frame-sync instance

NDIlib\_video\_frame\_v2\_t\* p\_video\_data, // The destination video frame

NDIlib\_frame\_format\_type\_e field\_type // The frame type that you prefer

);

The buffer returned is freed using the corresponding function:

void NDIlib\_framesync\_free\_video(

NDIlib\_framesync\_instance\_t p\_instance,

NDIlib\_video\_frame\_v2\_t\* p\_video\_data

);

## Genlock

When processing multiple video streams in a complex production environment it is frequently desirable to have all video streams operating at exactly the same frame rate with frame start times synchronized across all streams being processed. For physical video sources having a fundamental native video timebase (eg: a camera or video output card) this is handled by genlocking the internal video timebase to a reference signal, typically blackburst. Software only implementations with no inherent internal video timebase (eg: character generator, DDR playback, graphics rendering engine, etc.) across multiple platforms can run at exactly the same frame rate if system clocks are locked (via NTP, PTP, or other means) but frame start times will not be synchronized across different applications, as each individual application will start their frames at different points in time.

NDI genlock supports using an NDI signal as a timing reference for these software applications which otherwise lack an internal video timebase. The NDI genlock instance can connect to any NDI sender visible on the network and uses the timing from this NDI sender to correctly time and pace the sending of video frames, insuring both the overall framerate and the individual frame start times remain consistent with the selected NDI source. The NDI source used as a timing reference can be on the local network, a remote network, or in-cloud.

For an NDI source to correctly be able to operate as an NDI Genlock, it is important to bear in mind a couple of key ingredients as outlined below.

* It is very strongly recommended that the NDI source is a stream from NDI version 5 which has been significantly improved to support genlock capabilities. It is possible that some NDI streams from previous versions are not fully compliant with how NDI genlock operates.
* It is important that the source has enough network bandwidth to drive a reduced bandwidth signal to all the NDI genlock instances. Some Advanced SDK NDI converters might fail in this regard in which case an NDI Proxy may be used to relay the signal (and might also be used to relay cloud genlock as well). Configuring this source for multicast might also help, although multicast often is hard to fully support.
* NDI genlock is very robust and supports correct cross-frame-rate locking. For instance, a sender might be 30Hz and you are genlocking a 60Hz signal to it. This is however not a recommended workflow where it can be avoided.
* Some NDI sources like Test Pattern generator and NDI Screen Capture do not always send a regular stream of frames. They do this to save network bandwidth and CPU time. Sources such as these cannot be used as a basis for genlock.
* Remember that when creating NDI senders that you wish to use with genlock that you specify the use\_clock values as false when the senders are created. The NDI genlock functions replace the system clock pacing provided by the NDI library when use\_clock is set to true.
* If the genlock clock cannot correctly genlock to an NDI sender for some reason it will fall back to using the system clock and so can continue to work reasonably.
* Since there is some (low) overhead associated with each genlock instance it is recommended that you only have one for each source that you wish to lock too.

To create an NDI genlock instance, you use the function NDIlib\_genlock\_create, you may specify the NDI source that you wish to use as a reference and the NDI JSON settings associated with it. It is important that you remember to fill out the vendor\_id in the JSON or this sender will have the same limitation as NDI receivers. It is legal to create an NDI genlock that has a null source name and then later use the NDIlib\_genlock\_connect function to change the connection being used. Like all other NDI SDK functions, a genlock object can be disposed of with the function NDIlib\_genlock\_destroy.

If you wish to change the NDI source that is being used for the genlock, one can call the function NDIlib\_genlock\_connect. If the parameter is nullptr then the NDI source will be disconnected and the genlock operation will fall back to using the system clock.

The NDIlib\_genlock\_is\_active call may be used to determine if a particular NDI source is correctly operating as a genlock signal. If the NDI sender that is being used as the genlock source is not currently sending an active signal, then this will return false. Note that the functions to perform the clocking operation return whether the genlock is active and so this function need not be polled within a sending loop and is provided as a convenience.

Once you have created an NDI genlock instance and have it locked to an NDI receiver, then one may simply call the functions NDIlib\_genlock\_wait\_video and NDIlib\_genlock\_wait\_audio. These functions will wait until it is time for the next video or audio frame to be delivered and then return so that you can send it. While these structures take in a full frame descriptor, only the minimum number of members are used and so the rest need not be filled in. For video a NDIlib\_video\_frame\_v2\_t is used, however only the frame\_rate\_N, frame\_rate\_D and frame\_format\_type members need be valid. For audio a NDIlib\_audio\_frame\_v3\_t is used, however only the no\_samples and sample\_rate need be valid. These functions return a bool value which will tell you whether the source is currently genlocked or whether there is no signal and so the system clock has been used for timing.

The audio and video waiting functions are entirely thread safe and you may have a separate thread for the timing and of video and audio as needed.

An example application is given below that shows how one might lock to an NDI source and then send frames that would match the clock of that NDI source.

// We are going to start by creating an NDI genlock instance

NDIlib\_source\_t src\_name(**< Insert your NDI source here >**);

NDIlib\_genlock\_instance\_t p\_genlock = NDIlib\_genlock\_create(\&src\_name,

> NULL/\* Note that your vendor JSON is required here \*/);

// We are now going to loop and genlock to the signal at 59.94Hz

while (**\<Your application is running>**) {

// Setup the frame header.

NDIlib\_video\_frame\_v2\_t frame;

frame.frame\_rate\_N = 60000;

frame.frame\_rate\_D = 1001;

frame.frame\_format\_type = NDIlib\_frame\_format\_type\_progressive;

// We wait for a frame to send

const bool genlock\_locked = NDIlib\_genlock\_wait\_video(p\_genlock, \&frame);

printf("Send a frame, currently locked to %s.\n", genlock\_locked ? "remote source"

: "system clock");

}

NDIlib\_genlock\_destroy(p\_genlock);

## AV Sync

NDI relies on timestamps to synchronize incoming audio and video. You can of course fill these in yourself although by default NDI will use the system time. If these system times are synchronized (via NTP, PTP, GPS, or similar) multiple streams from different systems may be synchronized.

This API allows you to synchronize audio and video from either the same or different NDI sources with accuracy approaching one audio sample if a sender uses sample-accurate timestamps. While this of course can be achieved at a regular API user level this is a non-trivial task (for instance by ensuring that you audio and video frames have the same exact timestamps), with this API helping solve at least the following problems in a relatively simple way:

* The audio and video streams might be generated by applications that are not generating nanosecond accurate timestamps and so a significant level of filtering and accuracy improvement is needed so that frames can be aligned with sample-level accuracy.
* Packetizing the audio so that it correctly matches the video frames exactly is complex, and preserving and computing the correct timestamps, metadata and timecodes in this process is non-trivial.
* Often a particular pattern of output samples is needed for interfacing with hardware is needed (e.g., NTSC often requires audio samples on frames in alternating patterns of 1601 and 1602 samples) and it is very beneficial to ask the API for the number of samples wanted and it correctly aligns audio to this pattern.
* Correctly detecting when a stream has no audio or there are format changes requires special consideration.
* Timeouts and ensuring that errors remain within the Nyquist sampling limits when running on a computer that might already be under load or have inaccurate clocks makes a solution to this challenging.

This API is a little more complex than some of the others in this SDK because it allows for great flexibility within just a few functions. Please pay close attention to the exact definition of the return results and how they work.

### Guidelines

This API will take audio for a NDIlib\_recv\_instance\_t instance, returning a NDIlib\_avsync\_instance\_t object. It will then look at the audio being received on this input. One may then take video frames from either the same NDIlib\_recv\_instance\_t instance or a different one and one may query the NDIlib\_avsync\_instance\_t for the audio that matches that video frame. This takes the timestamps that are attached to each frame to synchronize the audio and video with high prevision.

It is important when working with this object that you ensure that the following conditions are met for the best results.

* This API does not resample or otherwise modify the audio samples. Special care must be taken when trying to synchronize audio and video streams that are not synchronized (genlocked).
* The audio and video do not need to be on the same clock (i.e., they do not need to be genlocked). However, if the streams are not synchronized then the number of audio samples returned with the video frame might not be exactly what is expected and can vary over time. For instance, if the audio being received is running on a clock that is 1% faster than the video clock then you will receive the exactly synchronized audio although there will be 1% more audio samples than one might expect based on the audio sample rate.
* The audio and video are expected to behave _relatively_ well (but not perfectly). If there are significant gaps in the audio or video streams, or the streams are delivered such that they have very significant jitter (well beyond the Nyquist sampling limit) then it is hard to ensure that they are reconstructed perfectly although all effort is made to act gracefully in these situations. As an example, if audio is delivered significantly later than the corresponding video then this device will end up needing to run behind on the video to find the exactly matching audio.
* You may pass an NDIlib\_recv\_instance\_t into NDIlib\_avsync\_create and the returned NDIlib\_avsync\_instance\_t will capture the audio from the device. You may simultaneously use this same NDIlib\_recv\_instance\_t to capture the audio and use the corresponding NDIlib\_avsync\_instance\_t as the means of synchronizing the audio and video from the same device. You similarly capture audio and video from different devices if you wish to use an NDIlib\_avsync\_instance\_t to synchronize audio and video from different sources, however, note the comments above on the clocking from disparate sources.
* This implementation will only support and process audio that is supported on your platform by the decoders within NDI (e.g., on Linux AAC audio might not correctly be used with this implementation).

### Creating and destroying devices

It is very simple to create and destroy a device. Simply create an NDI receiver that one wishes to receive audio from and pass it into NDIlib\_avsync\_create, this will return an instance of type NDIlib\_avsync\_instance\_t which may be used until you no longer need it. Once you no longer need it you should call NDIlib\_avsync\_destroy. Please note that you should destroy a device before the corresponding NDIlib\_recv\_instance\_t is called since an internal reference to this object is kept within the NDIlib\_avsync\_instance\_t\*.\*

### Recovering Audio

The most typical use of the synchronizing function would be to pass capture a video frame by the means that you normally would, then to make a call to:

NDIlib\_avsync\_ret\_e NDIlib\_avsync\_synchronize(

NDIlib\_avsync\_instance\_t p\_avsync,

const NDIlib\_video\_frame\_v2\_t \*p\_video\_frame,

NDIlib\_audio\_frame\_v3\_t\* p\_audio\_frame

);

This function is however deceptively powerful and the key to understanding this is correctly passing the correct values into the NDIlib\_audio\_frame\_v3\_t\* p\_audio\_frame parameter of this function. The following describes the values that may be specified.

\+---------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | **Parameter** | **Description** | +===============+================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================+ | no\_samples | If this value is "0" when you pass it into the function, then if audio may be recovered then the synchronization function will automatically return the exact length of audio that matches the video frame that was passed in as the first parameter. The number of samples returned in this way will almost exactly match the values related to the timestamps but can be assumed to very accurately reflect the audio that matches the frame. Because the timestamps are often subject to noise when frames are stamped, the number of samples might vary slightly. This will result in a return code of NDIlib\_avsync\_ret\_success. | | | | | | If this value is some constant (e.g., 1601 or 1602) then the AV sync will attempt to return this number of samples if it is close to the true number of audio samples that are aligned with this frame. This is designed so that you let this class correctly recover audio that might follow some external constraint on the number of samples that are used with video frames. If the number of samples requested is sufficiently close to the number of audio samples that match this frame, then a return code of NDIlib\_avsync\_ret\_success is returned. If it was not possible to correctly return this number of samples because it did not closely match the number of samples that are truly associated with this video frame, then the function will instead return the correct audio (which might be too many or too few samples) and return a value NDIlib\_avsync\_ret\_success\_num\_samples\_not\_matched. It would then be the responsibility of the caller to determine how to best handle this condition. This condition is normally caused by trying to synchronize audio and video that are not on the same clock. | | | | | | When requesting a specific number of audio samples, this is normally computed externally to this function under the assumption of some known audio sample rate. Because incoming audio might change sample rates which would render the number of requested samples invalid, please review the section below on how specifying the sample\_rate parameter of p\_audio\_frame\*.\* | +---------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | sample\_rate | When this function is "0", then there is no assumed sample rate, and the function will return an audio frame that specifies the current audio sample rate. | | | | | | If you are specifying the number of samples to be captured as non-zero, it is likely that this was computed at a given audio sample-rate. If you specify this on the p\_audio\_frame _as input,_ then if the sample rate of this audio source does not match it will not capture any audio and will return a result of NDIlib\_avsync\_ret\_format\_changed and fill in the audio format only in the returned structure. One can then simply recompute the number of required audio samples and simply call the function again to capture the audio with that video frame. | +---------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

If the NDIlib\_video\_frame\_v2\_t \*p\_video\_frame is not specified (is NULL) then the audio parameter can be used either to capture all current audio (no\_samples=0) or a specified number of audio samples (no\_samples is not zero) and behaves exactly as specified above although all audio is handled and not simply the audio associated with the current video frame.

Please note that this function will fill all parameters of the return frame, including the timecode, timestamp, and metadata. The metadata is chosen from the closest matching audio frame and is returned just one time. If the audio frames are all much smaller in duration than the corresponding video frames, then some metadata fields might be missing since they no longer have corresponding audio data to be assigned too.

It is important that you call NDIlib\_avsync\_free\_audio in order to return the frames returned by NDIlib\_avsync\_syncronize.

The full set of return codes from this function are documented below. Please note that these have integer values which are positive for success and negative for failure.

\+-------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | **Error code** | **Meaning** | +=====================================+===================================================================================================================================================================================================================================================================================================================================================================================================================================+ | \*\_success | This function succeeded and returned audio that matches the frame, and if you specified sample\_rate or no\_samples then correctly match those constraints. | +-------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | \*t\_success\_num\_samples\_not\_matched | This function succeeded, but you specified a no\_samples that could not be matched exactly because this would push the audio and video frame sufficiently out of alignment. The full audio samples associated with this video frame are returned and it does not match no\_samples. It might be a higher or lower number. | +-------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | \*\_no\_audio\_stream\_received | This indicates that there is currently no audio stream and so it would not be possible to return any audio data. This might indicate a video only stream. | +-------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | \*\_ret\_no\_samples\_found | This indicates that this video frame did not have matching audio data. This might be because the time at which this video frame was sent there was no corresponding audio. It might also indicate that the sending of the audio and video streams is sufficiently unaligned that they cannot easily be resynchronized; for instance, the audio data arriving is more than a second out of sync with the corresponding video data. | +-------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | \*\_format\_changed | A sample\_rate and no\_samples was specified; however the sample rate did not match and so this function has returned and correctly filled in the | +-------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | \*\_ret\_internal\_error | This function was called with incorrect | +-------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

## Using H.264, H.265, and AAC codecs

We recommend that you start by reviewing the sending of frames over the network using the NDI® SDK. The Advanced SDK is designed to operate almost identically, although you can send compressed data streams directly.

Currently the Advanced SDK supports H.264 or H.265 compression at a very wide variety of bitrates, resolutions, and framerates; and it would be common for you to use hardware-assisted compression to generate the compressed video stream on Advanced SDK devices, then send this onto the network using the NDI Advanced SDK. AAC audio is supported for audio transmission.

To send a compressed video frame, you should use the Advanced SDK structure NDIlib\_compressed\_packet\_t to pack your data. You should allocate memory for your packet so that the following data will be in a single block:

\+-----------------------+-----------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | Size and Type | Name | Details | +=======================+=================+=====================================================================================================================================================================================================================================================================================+ | uint32\_t, 4 bytes | version | This represents the current version number of the structure. This should be set to NDIlib\_compressed\_packet\_t::version\_0, which has a value of 44 currently (representing the structure size). | +-----------------------+-----------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | uint32\_t, 4 bytes | fourCC | This is the FourCC for the current compression format. Currently H.264 is supported, although other formats might be available in the future. | | | | | | | | - H.264 should be specified using NDIlib\_FourCC\_type\_H264. | | | | | | | | - H.265 should be specified using NDIlib\_FourCC\_type\_HEVC. | | | | | | | | - AAC audio should be submitted using NDIlib\_FourCC\_type\_AAC. | +-----------------------+-----------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | int64\_t, 8 bytes | pts | The stream presentation time stamp. See notes in the next section. | +-----------------------+-----------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | int64\_t, 8 bytes | dts | The stream display time stamp. See notes in the next section. | +-----------------------+-----------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | int64\_t, 8 bytes | reserved | This is currently a reserved field and will not be propagated by the SDK. | +-----------------------+-----------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | uint32\_t, 4 bytes | flags | The flags that apply to this frame. Currently there are two supported values for this setting: | | | | | | | | - NDIlib\_compressed\_packet\_t::flags\_none. Nothing. | | | | | | | | - NDIlib\_compressed\_packet\_t::flags\_keyframe. This is a frame that can be decoded without dependence on other stream data. This normally means that this is considered an I-Frame. For H.264 and H.265, key-frames must have extra data. This should always be set for AAC audio. | +-----------------------+-----------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | uint32\_t, 4 bytes | data\_size | The size of the compressed video frame. | +-----------------------+-----------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | uint32\_t, 4 bytes | extra\_data\_size | The size of the ancillary extra data for the current codec settings. This is required for key-frames in most compressed video formats. | +-----------------------+-----------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | data\_size bytes | \[data] | This is the compressed video frame data in byte format. | +-----------------------+-----------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | extra\_data\_size bytes | \[extra\_data] | This is the compressed ancillary data if extra\_data\_size is not zero. | +-----------------------+-----------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

### Sending Audio frames

To submit audio frames, start by building a structure of type NDIlib\_compressed\_packet\_t that has the compressed AAC audio data within it.

The following example creates a compressed audio frame when you have compressed audio data of size audio\_data\_size, at pointer p\_audio\_data with audio\_extra\_data\_size[^1] at pointer p\_audio\_extra\_data:

// See notes above

uint8\_t\* p\_audio\_data;

uint32\_t audio\_data\_size;

// See notes above

uint8\_t\* p\_audio\_extra\_data;

uint32\_t audio\_extra\_data\_size;

// Compute the total size of the structure

uint32\_t packet\_size = sizeof(NDIlib\_compressed\_packet\_t) + audio\_data\_size +

audio\_extra\_data\_size;

// Allocate the structure

NDIlib\_compressed\_packet\_t\* p\_packet = (NDIlib\_compressed\_packet\_t\*)malloc(packet\_size);

// Fill in the settings

p\_packet->version = NDIlib\_compressed\_packet\_t::version\_0;

p\_packet->fourCC = NDIlib\_FourCC\_type\_AAC;

p\_packet->pts = 0; // These should be filled in correctly if possible.

p\_packet->dts = 0;

p\_packet->flags = NDIlib\_compressed\_packet\_t::flags\_keyframe; // All AAC packets are a keyframe

p\_packet->data\_size = audio\_data\_size;

p\_packet->extra\_data\_size = audio\_extra\_data\_size;

// Compute the pointer to the compressed audio data, then copy the memory into place.

uint8\_t\* p\_dst\_audio\_data = (uint8\_t\*)(1 + p\_packet);

memcpy(p\_dst\_audio\_data, p\_audio\_data, audio\_data\_size);

// Compute the pointer to the ancillary extra data

uint8\_t\* p\_dst\_extra\_audio\_data = p\_dst\_audio\_data + audio\_data\_size;

memcpy(p\_dst\_extra\_audio\_data, p\_audio\_extra\_data, audio\_extra\_data\_size);

Once you have the compressed data structure that describes the frames, then you need simply create a regular NDIlib\_audio\_frame\_v3\_t to pass to NDI SDK functions as shown in the following example:

// Create a regular NDI audio frame, but of compressed format

NDIlib\_audio\_frame\_v3\_t audio\_frame;

audio\_frame.sample\_rate = 48000; // This must match AAC format

audio\_frame.no\_channels = 2; // This must match AAC format

audio\_frame.no\_samples = 1024; // This must match AAC format

audio\_frame.timecode = p\_packet->pts; // Might be a good value. Read docs!

audio\_frame.FourCC = (NDIlib\_FourCC\_audio\_type\_e)NDIlib\_FourCC\_audio\_type\_ex\_AAC;

audio\_frame.p\_data = (uint8\_t\*)p\_packet;

audio\_frame.data\_size\_in\_bytes = packet\_size;

audio\_frame.p\_metadata = "\<something/>"; // Per frame metadata is fully supported.

// Transmit the AAC audio data to NDI

NDIlib\_send\_send\_audio\_v3(pSender, \&audio\_frame);

Once this is sent the audio data will be transmitted, and you may free or re-use any audio data pointers that you allocated to represent the data. It is obviously possible to use a pool of memory to build audio packets without per-packet memory allocations.

The send audio function is thread-safe and may be called on a separate thread from compression transmission.

### Sending Video Frames

Because you are undertaking all compression of the video yourself, you should provide a full bandwidth and a preview bandwidth video stream. The full bandwidth stream may be any resolution and framerate; the preview bandwidth should be 640 pixels wide and square pixels (e.g., 640x360 pixels when at 16:9 image aspect ratio).

You will then need to submit these two frames separately to the SDK. The creation of the frames is identical to the audio example in the previous section. The following is an example of just one of the two required streams:

uint8\_t\* p\_h264\_data;

uint32\_t h264\_data\_size;

// This is probably zero for non I-frames, but MUST be set of I-frames

uint8\_t\* p\_h264\_extra\_data;

uint32\_t h264\_extra\_data\_size;

// Compute the total size of the structure

uint32\_t packet\_size = sizeof(NDIlib\_compressed\_packet\_t) + h264\_data\_size +

h264\_extra\_data\_size;

// Allocate the structure

NDIlib\_compressed\_packet\_t\* p\_packet = (NDIlib\_compressed\_packet\_t\*)malloc(packet\_size);

// Fill in the settings

p\_packet->version = NDIlib\_compressed\_packet\_t::version\_0;

p\_packet->fourCC = NDIlib\_FourCC\_type\_H264;

p\_packet->pts = 0; // These should be filled in correctly !

p\_packet->dts = 0;

p\_packet->flags = is\_I\_frame ? NDIlib\_compressed\_packet\_t::flags\_keyframe

: NDIlib\_compressed\_packet\_t::flags\_none;

p\_packet->data\_size = h264\_data\_size;

p\_packet->extra\_data\_size = h264\_extra\_data\_size;

// Compute the pointer to the compressed h264 data, then copy the memory into place.

uint8\_t\* p\_dst\_h264\_data = (uint8\_t\*)(1 + p\_packet);

memcpy(p\_dst\_h264\_data, p\_h264\_data, h264\_data\_size);

// Compute the pointer to the ancillary extra data

uint8\_t\* p\_dst\_extra\_h264\_data = p\_dst\_h264\_data + h264\_data\_size;

memcpy(p\_dst\_extra\_h264\_data, p\_h264\_extra\_data, h264\_extra\_data\_size);

Having filled in the audio compressed data structures, fill in a video header as shown in the following example.

Note: It is crucial that the value of the FourCC specifies whether this is the full or preview resolution streams.

// Create a regular NDI audio frame, but of compressed format

NDIlib\_video\_frame\_v2\_t video\_frame;

video\_frame.xres = 1920; // Must match H.264 data

video\_frame.yres = 1080; // Must match H.264 data

// This must value is dependent on whether this is a full or preview

// stream. This example shows the full resolution stream, the preview.

// resolution stream would specify NDIlib\_FourCC\_type\_H264\_lowest\_bandwidth

video\_frame.FourCC = (NDIlib\_FourCC\_video\_type\_e)NDIlib\_FourCC\_type\_H264\_highest\_bandwidth;

// Any reasonable sample rate is supported

video\_frame.frame\_rate\_N = 60000;

video\_frame.frame\_rate\_D = 1001;

// Any reasonable aspect ratio is supported

video\_frame.picture\_aspect\_ratio = 16.0f/9.0f;

// We only currently allow

video\_frame.frame\_format\_type = NDIlib\_frame\_format\_type\_progressive;

// Choose a good value, this is an example

video\_frame.timecode = p\_packet->pts;

// Set the data

video\_frame.p\_data = (uint8\_t\*)p\_packet;

video\_frame.data\_size\_in\_bytes = packet\_size;

// Metadata is supported

video\_frame.p\_metadata = "\<Hello/>";

// Transmit the H.264 video data to NDI, async is fully supported.

// But read SDK description of buffer lifetimes.

NDIlib\_send\_send\_video\_async\_v2(pSender, \&video\_frame);

A critical part of NDI is that you ask the SDK if you should insert an I-frame. This can be achieved by making a call to NDIlib\_send\_is\_keyframe\_required. This call returns true when an I-Frame should be inserted for a down-stream source to correctly decode the image without errors.

For instance, when there is a new NDI connection this function will return true; when a down-stream source has dropped a packet and can no longer decode the rest of the GOP, then this will be detected and return true, etc.

You are free to insert I-frames when you want based on your GOP requirements, but when this function returns true then you should issue an I-Frame at the next possible time. This is required functionality for a good user experience and a compliant NDI HX source. The stream validation (described later) will verify that this practice is followed.

Although you can determine your own bitrates for H.264 or H.265 streams, the NDI HX SDK will also provide guidance if you wish to call NDIlib\_send\_get\_target\_frame\_size. When used with H.264 or H.265 FourCC's in the structures, this will provide a reasonable _average_ bitrate recommendation based on the selected resolution, framerate, etc. This provides estimates for all combinations of framerates and resolutions, but compliance is not required.

Please note that NDI is a real-time API, meaning that you should make every effort to pass off video and audio as they arrive in synchronization with each other. These are passed through the transmission layers downstream to the remote device with the lowest possible latency and may be received by sources that are observing one or both streams (when possible, bandwidth is only allocated for the used streams in transmission).

Audio and video are sent when they are passed to the API, allowing one or both streams to stop or start as needed at any layer. As a result, frames are not "held" to synchronize streams, resulting in higher performance.

### H.264 Support

#### Supported Formats

NDI assumes that all H.264 data is as specified in Annex B of ITU-T H.264 | ISO/IEC 14496-10. The data must include the start codes. We support 4:2:0 in Baseline, Main, and High profiles up to level 5.1. We recommend that an I-Frame interval of between one and two seconds; it is however very important that you recall that the SDK will inform you when I-Frame insertions are required. The recommended bitrate of the stream is provided by NDIlib\_send\_get\_target\_bit\_rate; for HX it is considered reasonable that you allow the user to select whether you wish a bit rate in the range of 0.1 -- 2.0x the bitrate provided by this function.

The current NDI implementation supports H.264 decoding without any installed plugins across the Windows and macOS targets. If you require Linux support, please contact NDI SDK support.

Full hardware acceleration is provided on these platforms if the relevant hardware acceleration recommendations are provided by an end user application as described in the NDI Advanced SDK documentation. On Windows 7, the maximum decoder resolution is 1920x1080 due to OS limitations. On other platforms the maximum resolution is currently 4096x2304.

While you may use all H.264 frame types, we recommend considering the use of B frames with caution, since these will cause increased decoder latency in some configurations. Because NDI is a low latency, real-time scheme, it is crucial that you focus your great effort on achieving low latency and high quality.

It is crucial that your codec provide reasonable accurately-timed frames; a common problem that we have observed is that -- with some bitrate control algorithms -- that the I-Frames are sufficiently large that subsequent P frames are delayed far more than the Nyquist sampling limit, which tends to cause jittery video transmission.

#### Extra Data

The extra ancillary data required to configure an H.264 codec must correctly be provided. This must exist for all I-frames. The structure of this data should contain concatenated NAL units in Annex B format, along with their start codes. For H.264, they are the SPS and PPS NAL units.

#### PTS and DTS

The PTS and DTS are not used directly by the NDI SDK, however they are important to ensure that frames may be decoded and displayed in correct order. While we recommend that you use 100 ns intervals for these values, any time-base is technically supported if the ordering of integers is correct.

### H.265 Support

#### Supported Formats

NDI assumes that all H.265 data is as specified in Annex B of ITU-T H.265 | ISO/IEC 23008-2. The data must include the start codes. We support 4:2:0 in Main, Main Still Picture, and Main10 profiles in resolutions up to 4096x2304 pixels if it is supported by decoding hardware. We recommend that an I-Frame interval of between one and two seconds; it is however very important that you recall that the SDK will inform you when I-Frame insertions are required. The recommended bitrate of the stream is provided by NDIlib\_send\_get\_target\_bit\_rate; for HX it is considered reasonable that you allow the user to select whether you wish a bit rate in the range of 0.1 -- 2.0x the bitrate provided by this function.

The recommended rules for the H.265 frame types are the same as those for the H.264 frame types described in the above section.

#### Extra Data

The extra ancillary data required to configure an H.265 codec must correctly be provided. This must exist for all I-frames. The structure of this data should contain concatenated NAL units in Annex B format, along with their start codes. For H.265, they are the VPS, SPS, and PPS NAL units.

#### PTS and DTS

The PTS and DTS are not used directly by the NDI SDK, however they are important to ensure that frames may be decoded and displayed in correct order. While we recommend that you use 100 ns intervals for these values, any time-base is technically supported if the ordering of integers is correct.

### AAC Support

#### Supported Formats

AAC audio submitted to the NDI SDK should be in the raw AAC format.

All reasonable AAC bitrates are supported, although it is recommended that you use 192 kbps as a reasonable compromise. Any valid AAC channel count and sample-rate should be supported.

#### Extra Data

The extra ancillary data required to configure an AAC codec must correctly be provided. This must exist for all AAC frames. The structure of this data should be the AudioSpecificConfig structure followed by GASpecificConfig as specified from ISO/IEC 14496-03. In practice, this means the extra-data section should be two bytes for each audio frame that describes the sample rate, channel count, etc. The information set in this structure should correctly match the audio settings header on the same frame.

#### Audio Levels

The AAC audio level matches that of the NDI SDK, meaning that a floating-point sine-wave value of 1.0 represents a +4 dBU equivalent audio level.

### OPUS Support

Starting in version 5 of the NDI SDK one may transmit Opus audio through NDI. All audio is in the multi-channel Opus format, allowing up to 255 channels of audio within a packet. The FourCC used should be be NDIlib\_FourCC\_audio\_type\_ex\_OPUS,

The properties provided within NDIlib\_audio\_frame\_v3\_t must reflect all legal values for the Opus codec, meaning that all bitrates, channel counts and valid packet sizes are supported. All audio within NDI uses the floating-point decompression functions, meaning that clipping would not occur if full range audio streams and it is recommended that you use the floating-point versions of the compression. The Opus audio level matches that of the NDI SDK, meaning that a floating-point sine-wave value of 1.0 represents a +4 dBU equivalent audio level.

Unlike other compressed formats, the raw audio data is provided within the NDIlib\_audio\_frame\_v3\_t structure data fiels and a NDIlib\_compressed\_packet\_t is not used. This distinction is made to reduce the bandwidth required for Opus audio over WAN networks by removing the need for extra PTS, DTS values which are not required for the decompression of the Opus bit-stream.

### Latency of Compressed Streams

It is very important that you look very closely at your streams and ensure that they are configured for low latency decoding which is non-trivial and many "off the shelf" H.264 and HEVC encoders do not provide without changing their settings. If an encoder has a low latency mode, then this is often a very good starting point. Other areas to look at are ensuring that you are using only I and P frames so that each incoming frame can be immediately decoded without a delay. In addition, it is very common that different codecs place "video delay" settings in the SPS NAL or SEI NAL units in the stream that instruct a decoder to delay the display of frames which introduces latency. In our experience, almost no HX implementation has been correctly configured for lowest latency use by default and work has always been needed to optimize the stream settings.

We have tools that can help measure the decode latency imposed by the stream if you wish.

### Stream Validation

To help highlight some potential problems, the NDI HX SDK will validate many aspects of streams that are passed to it. It is designed so that it will display the associated error on STDOUT if your stream is incorrect and then terminate the stream (to highlight that it is not a valid stream).

\[Obviously, this can make it quite important to monitor STDOUT in your implementation]{.underline}.

Important Note: To ensure that NDI HX works correctly across all devices and that a great end-user experience can be expected, it is very important that you take significant time testing to confirm that your implementation fully complies with this document and works well in practice.

## External Tally Support

Some sending might have knowledge of whether they are on output that might come from another non-NDI device. In effect, the sender side wants to indicate how it is known to be used and have that be reflected both to the local device but also down-stream in the "echoed tally messages". To achieve this there is a function on the sender side that updates the tally state.

bool NDIlib\_send\_set\_tally(

NDIlib\_send\_instance\_t p\_instance, const NDIlib\_tally\_t\* p\_tally

);

This function will "reference" count the local tally and the remote tally to give an indicator both on the local device and on the remote connections on the combined NDI and local tally values.

## KVM Support

The NDI advanced SDK includes support for controlling a remote KVM enabled device over NDI. This SDK allows you to send keyboard, mouse, clipboard, and touch messages to a sender that is marked as supporting these messages. This functionality is implemented on an NDI receiver and messages are sent from this to a sender to offer KVM control.

It is critical that you test any applications you have very thoroughly since full support of a remote machine is not simple and has many edge cases that are hard to ensure solve -- for instance for every keyboard and mouse "press" there must also be a corresponding "release message" or the remote machine will interpret this as a "stuck key".

The following documentation describes all the relevant functions required for KVM control.

bool NDIlib\_recv\_kvm\_is\_supported(NDIlib\_recv\_instance\_t p\_instance);

This function will tell you whether the current source that this receiver is connected to is able to be KVM controlled. The value returned by this function might change based on whether the remote source currently has KVM enabled or not. If you are receiving data from the video source using NDIlib\_recv\_capture\_v2, then you will be notified of a potential change of KVM status when that function returns NDIlib\_frame\_type\_status\_change.

bool NDIlib\_recv\_kvm\_send\_left\_mouse\_click(NDIlib\_recv\_instance\_t p\_instance);

bool NDIlib\_recv\_kvm\_send\_middle\_mouse\_click(NDIlib\_recv\_instance\_t p\_instance);

bool NDIlib\_recv\_kvm\_send\_right\_mouse\_click(NDIlib\_recv\_instance\_t p\_instance);

When you want to send a mouse down (i.e., a mouse click has been started) message to the source that this receiver is connected too then you would call this function. There are individual functions to allow you to send a mouse click message for the left, middle and right mouse buttons. To simulate a double-click operation you would send three messages; the first two would be mouse click messages with the third being the mouse release message. It is important that for each click message there must be at least one mouse release message or the operating system that you are connected to will continue to believe that the mouse button remains pressed forever.

bool NDIlib\_recv\_kvm\_send\_left\_mouse\_release(NDIlib\_recv\_instance\_t p\_instance);

bool NDIlib\_recv\_kvm\_send\_middle\_mouse\_release(NDIlib\_recv\_instance\_t p\_instance);

bool NDIlib\_recv\_kvm\_send\_right\_mouse\_release(NDIlib\_recv\_instance\_t p\_instance);

When you wish to send a mouse release message to a source, then you will call one of these functions. They simulate the release of a mouse message and would always be sent after a mouse click message has been sent.

bool NDIlib\_recv\_kvm\_send\_vertical\_mouse\_wheel(

NDIlib\_recv\_instance\_t p\_instance, const float no\_units

);

bool NDIlib\_recv\_kvm\_send\_horizontal\_mouse\_wheel(

NDIlib\_recv\_instance\_t p\_instance, const float no\_units

);

To simulate a mouse-wheel update you will call this function. You may simulate either a vertical or a horizontal wheel update which are separate controls on most platforms. The floating-point unit represents the number of "units" that you wish the mouse-wheel to be moved, with 1.0 representing a downwards or right-hand single unit.

bool NDIlib\_recv\_kvm\_send\_mouse\_position(

NDIlib\_recv\_instance\_t p\_instance, const float posn\[2]

);

To set the mouse cursor position you will call this this function. The coordinates are specified in resolution independant settings in the range 0.0 - 1.0 for the current display that you are connected to. The resolution of the screen can be known if you are receiving the video source from that device, and this might change on the fly of course. Position (0,0) is the top left of the screen. posn\[0] is the x-coordinate of the mouse cursor, posn\[1] is the x-coordinate of the mouse cursor.

bool NDIlib\_recv\_kvm\_send\_clipboard\_contents(

NDIlib\_recv\_instance\_t p\_instance, const char\* p\_clipboard\_contents

);

In order to send a new "clipboard buffer" to the destination one would call this function and pass a null terminated string that represents the text to be placed on the destination machine buffer.

bool NDIlib\_recv\_kvm\_send\_touch\_positions(

NDIlib\_recv\_instance\_t p\_instance, const int no\_posns, const float posns\[]

);

This function will send a touch event to the source that this receiver is connected to. There can be any number of simultanous touch points in an array that is transmitted. The value of no\_posns represents how many current touch points are used, and the array of posns\[] is a list of the \[x,y] coordinates in floating point of the touch positions. These positions are processed at a higher precision that the screen resolution on many systems, with (0,0) being the top-left corner of the screen and (1,1) being the bottom-right corner of the screen. As an example, if there are two touch positions, then posns\[0] would be the x-coordinate of the first position, posns\[1] would be the y-coordinate of the first position, posns\[2] would be the x-coordinate of the second position, posns\[2] would be the y-coordinate of the second position.

bool NDIlib\_recv\_kvm\_send\_keyboard\_press(

NDIlib\_recv\_instance\_t p\_instance, const int key\_sym\_value

);

To send a keyboard press event, this function is called. Keyboard messages use the standard X-Windows Keysym values. A copy of the standard define that is used for these is included in the NDI SDK for your convenience with the filename "Processing.NDI.KVM.keysymdef.h". Since this file includes many #defines, it is not included by default when you simply include the NDI SDK files, if you wish it to be included you may #define the value NDI\_KVM\_INCLUDE\_KEYSYM or simply include this file into your application manually. For every key press event it is important to also send a keyboard release event, or the destination will believe that there is a "stuck key"! Additional information about keysym values may easily be located online, for instance https://www.tcl.tk/man/tcl/TkCmd/keysyms.html

bool NDIlib\_recv\_kvm\_send\_keyboard\_release(

NDIlib\_recv\_instance\_t p\_instance, const int key\_sym\_value

);

Once you wish a keyboard value to be "release" then one simply sends the matching keyboard release message with the associated key-sym value.

## NDI Embedded SDK Example Design

The NDI Embedded SDK contains working example designs intended to assist in using the Embedded NDI software SDK and the FPGA NDI IP cores.

Building a working embedded NDI design requires wide-ranging expertise, and these example designs are provided to make this process more accessible. Pre-built uSD card images are available for the supported platforms, and details are provided regarding how to rebuild each required piece from scratch.

### Quick Start

1. Obtain one of the supported development boards:

* [Altera Arria-10 SoC Devkit](https://www.intel.com/content/www/us/en/products/details/fpga/development-kits/arria/10-sx.html)
* [Digilent Zybo-Z7-20](https://store.digilentinc.com/zybo-z7-zynq-7000-arm-fpga-soc-development-board/)
* [Digilent Arty-Z7-20](https://digilent.com/shop/arty-z7-zynq-7000-soc-development-board/) (partial support, the Zybo-Z7-20 is preferred)
* [Terasic SoCKit](https://www.terasic.com.tw/cgi-bin/page/archive.pl?Language=English\&No=816) + [DVI-HSMC](https://www.terasic.com.tw/cgi-bin/page/archive.pl?Language=English\&CategoryNo=68\&No=359) (optional)
* [Xilinx ZCU104](https://www.xilinx.com/products/boards-and-kits/zcu104.html)

2. Boot your board using a pre-built image

* See the README.uSD.md file for details on obtaining the uSD image, writing it to a uSD card, and using it with one of the supported platforms.

### Directory structure:

\+-----------------+-------------------------------------------------------------+ | Filename | Contents | +=================+=============================================================+ | `README.md` | High level overview of the provided example projects | +-----------------+-------------------------------------------------------------+ | `README.uSD.md` | Details on using the pre-built uSD images | +-----------------+-------------------------------------------------------------+ | `CHANGELOG.md` | List of notable changes | +-----------------+-------------------------------------------------------------+ | `src/cpp` | Software source code and example projects | +-----------------+-------------------------------------------------------------+ | `src/fpga` | Hardware design files and example projects | +-----------------+-------------------------------------------------------------+ | `linux_kernel` | Projects to build kernel and boot loader | +-----------------+-------------------------------------------------------------+ | `os_uSD` | Scripts to generate root filesystem and bootable uSD images | +-----------------+-------------------------------------------------------------+

### Design Overview

There are a lot of different pieces required to create a working example system:

* FPGA hardware design
* Boot loader
* Linux kernel
* Device-tree
* Linux root file system
* Application software
* Bootable image including all of the above

While each of these components is important and necessary, not all of them have to be customized in order to create a custom project. The three components that need to be customized for almost any project are the actual FPGA hardware, the application software, and the device tree. The Linux kernel and boot loader can typically be used without customizations and there are several options for creating a Linux root file system.

This example design uses vendor specific processes to generate the boot loader, the Linux kernel, and most of the device tree. The root file system is a standard Debian install. The FPGA logic and application software are custom, and some device-tree customizations are required to support the custom FPGA hardware. Each of these components is discussed in more detail below.

### Theory of Operation

Embedded systems require interaction between hardware and software. This is a complex process even on small micro-controllers, and is even more so on systems running a high-level OS such as Linux. This example design was created with the intent to make the interaction between hardware and software as simple as possible to implement and modify, with no need to write kernel-space code.

#### Common Encode and Decode Base Functionality

**Low Level Details and Device-Tree**

In embedded systems, it is necessary to communicate details of the system across several fundamentally different environments (e.g., hardware, the Linux kernel, and application software). Rather than using lots of "magic" numbers stashed in cryptic header files, this is now mostly done at the system level with device-tree. Details provided via device-tree can control which kernel modules get loaded (or do not), as well as modify the behavior of those modules.

**Hardware IP**

* Vendor IP

&#x20;

* Standard vendor IP blocks for GPIO and I2C are implemented along with the Hard IP ARM cores in the example design files. These vendor IP blocks are added to the device-tree and are controled by standard Linux kernel drivers. This allows the pushbutton switches and LEDs connected to the FPGA fabric to be made available to the Linux kernel as part of the standard gpio subsystem.

&#x20;

* Xilinx HDMI IP

&#x20;

* The ZCU104 uses the Xilinx HDMI IP core, however the Linux drivers for this core are disabled since the NDI encoding data flow operates outside of the Linux kernel's standard video input and output systems. The control software for the HDMI core instead operates on one of the Cortex-R5 cores, which means all the hardware used by the HDMI core (uart1, i2c0, i2c1) must be disabled in Linux to avoid contention.

&#x20;

* Altera HDMI IP

&#x20;

* The Arria 10 examples use the Altera HDMI IP cores. Management of these IP cores is done outside of Linux using the NIOS CPU core instantiated as part of the Altera HDMI example design. This code is used as-is for HDMI Rx, and modified to allow configuration of the HDMI Tx logic to support several different resolutions for the NDI Decode example design.

&#x20;

* Custom IP

&#x20;

* The NDI hardware implements a 64K block of register space which is manually added to the device-tree along with IRQ settings and other hardware details. The generic-uio driver is used to map the hardware register space and IRQs so they are available to user-space code.

**Reserved Memory Regions**

This design requires two large blocks of buffer memory, one for compressed video and raw audio data that must be visible to Linux, and one for raw video data which can be visible to Linux but may also be implemented as a separate memory bank for improved system performance.

* NewTek\_Reserved

&#x20;

* This memory region is used to store compressed video data and raw audio data. This region is marked cacheable for performance. Cache coherency is maintained since the hardware accesses this memory region using a cache coherent bus interface.

&#x20;

* NewTek\_Video

&#x20;

* This memory region is used to store raw (uncompressed) video data and thus requires high bandwidth. The application does not access this memory in normal operation so this region may be implemented as a dedicated FPGA-side memory bank to improve performance. This region will only appear in the device-tree if implemented using shared memory visible to Linux. If a live video input is not available and this memory region is visible to Linux, a video pattern can be written to allow testing of the compression core and software application.

**Software**

* Standard Vendor IP Blocks

&#x20;

* The standard Vendor IP blocks (for I2C and GPIO) are added to the device tree and stock Linux kernel drivers are used to communicate with this hardware.

&#x20;

* Custom IP blocks

&#x20;

*   The device-tree entries for the custom FPGA IP use the generic-uio kernel driver. This driver makes the memory regions and IRQs specified by the device-tree entries accessible to user-space code. The libuio library is used to facilitate accessing the hardware.

    The application also reads details regarding the reserved memory regions directly from the device-tree, as this functionality is not supported by libuio.

**Initial Startup**

\+-----------------------------------+------------------------------------------------+ | File(s) | Description | +===================================+================================================+ | `<device-tree>` | Contains memory addresses and IRQ details | +-----------------------------------+------------------------------------------------+ | `src/cpp/ndi_common/hardware.*` | Low-level access to register space and IRQs | +-----------------------------------+------------------------------------------------+ | `src/cpp/ndi_common/device.*` | Machine specific details | +-----------------------------------+------------------------------------------------+ | `src/fpga/ip/common/Version.vhd` | Version information compiled into the hardware | +-----------------------------------+------------------------------------------------+ | `src/cpp/ndi_encode/ndi_encode.*` | Top level application file (Encode) | +-----------------------------------+------------------------------------------------+ | `src/cpp/ndi_decode/ndi_decode.*` | Top level application file (Decode) | +-----------------------------------+------------------------------------------------+

When the application is launched, quite a bit of setup is performed before continuing operation is passed to a number of created threads. Most of this setup happens when the hardware class is initialized. After the software processes any command-line options, an instance of the hardware class is created and initialized.

The hardware class uses libuio to access the memory regions and interrupts specified in the device-tree for the custom hardware IP. In addition, details for the reserved memory regions are read from the device-tree and the memory is mmap()'d to make it available for access. At this point, version information is read from the hardware and a device class specific to the hardware platform is created. This allows some behaviors to change between different physical boards (currently used to control any required ACP address translation). Finally, details regarding the UIO devices and reserved memory are printed to the console, and the reserved memory region for compressed video is written with the value 0xdeaddead, making it easier to identify uninitialized memory locations when debugging.

#### NDI Encode Operation

**Video Input**

\+--------------------------------------+---------------------------------------+ | File(s) | Description | +======================================+=======================================+ | `src/cpp/ndi_encode/video_capture.*` | video\_capture:: class | +--------------------------------------+---------------------------------------+ | `src/cpp/ndi_encode/track.*` | track:: class | +--------------------------------------+---------------------------------------+ | `src/fpga/ip/common/Vid_Track.vhd` | video format tracking hardware | +--------------------------------------+---------------------------------------+ | `src/fpga/ip/common/Vid_In.vhd` | Video input DMA logic | +--------------------------------------+---------------------------------------+ | `src/fpga/ip/common/Preview.vhd` | Creates low-resolution preview stream | +--------------------------------------+---------------------------------------+ | Filename varies | Low level video input logic | +--------------------------------------+---------------------------------------+

Video input starts with the low level video input logic. This logic is platform specific:

\+------------+------+----------------------------------------------------+ | Platform | Dir | Implementation | +============+======+====================================================+ | a10socdk | Rx | Altera HDMI RxTx example design | +------------+------+----------------------------------------------------+ | sockit | Rx | Not yet implemented | +------------+------+----------------------------------------------------+ | ZCU104 | Rx | Xilinx HDMI Rx example design | +------------+------+----------------------------------------------------+ | Zybo-Z7-20 | Rx | Open-source HDMI or DVI Rx logic | +------------+------+----------------------------------------------------+

The software for controlling the low-level video input logic is currently outside the scope of this example. The Zybo-Z7 HDMI input requires no control software, the Xilinx HDMI control software is running bare-metal on one of the Cortex-R5 cores, and the Altera HDMI control software runs on a dedicated NIOS core (see the README file accompanying the hardware project files for details). There is, however, software support for low-level input monitoring code to send details (locked/unlocked, resolution, etc) to the video\_capture:: class. See video\_capture::set\_signal() and the do\_commands() function in ndi\_send.cpp for details.

Once the low-level video signal is received by the FPGA hardware, the signal is synchronized to the main clock and converted to a standard interface (HDMI\_T). This signal is sent to the format tracking logic (Vid\_Track.vhd) and is filtered (Preview.vhd) to create a low resolution preview stream. The resulting full and preview resolution streams are each sent to a bus mastering DMA engine (Vid\_In.vhd) which assembles pixels into words and writes the raw video data into memory.

The track:: class monitors the video format tracking hardware to detect the acquisition or loss of signal lock. When either event happens, appropriate details are communicated to the video\_capture:: class to start or stop video streaming.

The video\_capture:: class manages the recording of video data into the reserved memory buffer as well as hardware settings controlling things like the data format and preview decimation ratio. Since there are two input streams, the video\_capture:: class operates on frame pairs. Each pair always contains a full resolution frame and may or may not contain a preview resolution frame. The maximum preview framerate is 30 fps, so if the full framerate is greater than 30 fps, some full frames will not have a corresponding preview frame. For simplicity, this is represented as a frame pair with an invalid preview pointer (NULL).

The video\_capture::capture\_frames thread loops through the following process to capture frame pairs:

* Create a new frame pair
  * Allocate memory from the reserved buffer
  * Fill in frame details
* Queue the frame (send details to the hardware)
* Post the frame (tell hardware the new frame data is valid)
* Wait for an interrupt indicating the frame has been captured
* Send the frame to the compression logic

Once a frame pair is captured, it is placed on a work queue for the video compression engine. If this queue is too full, the oldest pending frame pair is dropped and a warning is printed.

NOTE: The video input logic assumes a "clean" video signal and is an example design intended to be simple and easy to understand. This logic does not attempt to deal with real world problems like random noise on an unterminated SDI input or when users plug/unplug the cable while the system is running. An actual production device should be more tolerant of signal errors.

**Video Pattern**

\+-------------------------------------------+--------------------------+ | File(s) | Description | +===========================================+==========================+ | `src/cpp/ndi_encode/video_pattern.*` | video\_pattern:: class | +-------------------------------------------+--------------------------+

The video\_pattern:: class is a sub-class of the video\_capture:: class that does not rely on hardware to generate a video stream. Rather than capturing frames from hardware, this class generates a video pattern any time the signal format changes (it is possible to manually change the resolution at run-time from the console). The video pattern generated is twice as tall as the video format (see the m\_scroll\_dist member variable), and a preview resolution version of the pattern is also created (since both full and preview resolution streams are required). In normal operation, a frame-sized window of the over-sized video pattern is used to provide source video for the video\_compress:: class, with the start point of the window moving down one line with each new frame, providing the illusion of moving video.

**Video Compression**

\+---------------------------------------+-------------------------------------------+ | File(s) | Description | +=======================================+===========================================+ | `src/cpp/ndi_encode/video_compress.*` | video\_compress:: class | +---------------------------------------+-------------------------------------------+ | `src/fpga/NDI_Enc/Encode_x4.vhd` | 4-core NDI Hardware Encoder | +---------------------------------------+-------------------------------------------+ | `src/fpga/NDI_Enc/Encode_Xilinx.vhdp` | Single NDI Hardware Encoder core (Xilinx) | +---------------------------------------+-------------------------------------------+ | `src/fpga/NDI_Enc/Encode_Altera.vhdp` | Single NDI Hardware Encoder core (Altera) | +---------------------------------------+-------------------------------------------+

One, two, or four copies of the single NDI Encoder core are instantiated in hardware (Encode\_x4.vhd). The cores operate in parallel, with each core operating on one or more quarters of the video frame known as a "slice". This increases the effective throughput of the compression core and lowers system latency.

The video\_compress:: class monitors a work queue filled with raw video frames by the video\_capture:: class (above). When a new frame pair is received the full resolution frame is sent to hardware for processing. Once the hardware generates an interrupt indicating processing is finished, the software performs minimal post-processing on the compression thread. Slice lengths are read from the hardware and written to memory by fixup\_frame(), the resulting frame length is used to update the quality setting, then the frame is passed to the NDI send threads via add\_frame\_ndi().

The send threads (send\_full, send\_prvw) monitor independent work queues for the full and preview resolution video streams and pass new frames to the NDI stack. Sending a frame to the NDI stack is a synchronizing event which will block until the frame is fully processed, which is why there are independent threads for the two video streams. This gives each stream the maximum amount of time to send data to the listeners.

**NDI Transmission**

\+---------------------------------------------+------------------------------+ | File(s) | Description | +=============================================+==============================+ | `src/cpp/ndi_encode/video_compress.*` | video\_compress:: class | +---------------------------------------------+------------------------------+ | `src/cpp/ndi_encode/network_send_video.cpp` | network\_send:: video support | +---------------------------------------------+------------------------------+

The send\_full() and send\_prvw() threads in the video\_compress:: class monitor independent work queues for the full and preview resolution video streams and pass new frames to network\_send::add\_frame(). This function is basically a shim which converts the video\_compress::frame\_t type into an NDIlib\_video\_frame\_v2\_t type and sends it to the NDI stack. Sending a frame to the NDI stack is a synchronizing event which will block until the frame is fully processed, which is why there are independent threads for the two video streams. This gives each stream the maximum amount of time to send data to the listeners.

NOTE: The various frame types exist primarily because much of the example code predates the availability of the official NDI Embedded SDK. Expect future releases of this code to migrate to using native NDI frame types.

**Audio Input**

\+-----------------------------------------+----------------------------+ | File(s) | Description | +=========================================+============================+ | `src/cpp/ndi_encode/audio_capture.*` | audio\_capture:: class | +-----------------------------------------+----------------------------+ | `src/fpga/ip/common/Aud_In.vhd` | Audio input DMA logic | +-----------------------------------------+----------------------------+

Audio input starts with the low level audio input logic. This logic supports standard iis serial audio streams. Support for parallel audio data extracted from the HDMI streams will be supported soon. Once assembled into complete audio samples, the data is queued in a FIFO allowing burst writes to system memory.

The audio\_capture:: class manages the recording of audio data into the reserved memory buffer as well as hardware settings controlling things like the data format (typically left-justified).

The audio\_capture::capture\_frames thread loops through a process very similar to the video\_capture thread. One major difference is the audio logic currently supports "overlapped" commands, meaning the next command is written to hardware while the current command is still in progress:

* Create a new frame
  * Allocate memory from the reserved buffer
  * Fill in frame details
* Queue the frame (send details to the hardware)
* Post the frame (tell hardware the new frame data is valid)
* Loop until signaled to exit
  * Create a new frame
    * Allocate memory from the reserved buffer
    * Fill in frame details
  * Queue the frame (send details to the hardware)
  * Post the frame (tell hardware the new frame data is valid)
  * Wait for an interrupt indicating the frame has been captured
  * Send the frame to the compression logic

**Audio Compression**

\+---------------------------------------+---------------------------------+ | File(s) | Description | +=======================================+=================================+ | `src/cpp/ndi_encode/audio_compress.*` | audio\_compress:: class | +---------------------------------------+---------------------------------+

This class simply passes captured audio frames from the audio\_capture:: class to the NDI stack. It exists primarily to match the video path data flow, to allow any processing of the audio samples that might be required (eg: masking lower-order bits that might contain AES packet data), and because when this code was initially written the NDI API did not support 32-bit signed audio samples.

In future versions, expect the native NDI types to be used and the audio\_compress class to be removed.

**Audio Transmission**

\+---------------------------------------------+------------------------------+ | File(s) | Description | +=============================================+==============================+ | `src/cpp/ndi_encode/network_send_audio.cpp` | network\_send:: audio support | +---------------------------------------------+------------------------------+

This function is basically a shim which converts the audio\_capture::frame\_t type into an NDIlib\_audio\_frame\_interleaved\_32s\_t type and sends it to the NDI stack.

Expect this code to be deprecated in future versions and the sending logic to be migrated to the audio\_capture:: class.

**Tally Operation**

\+------------------------------------+---------------------------------+ | File(s) | Description | +====================================+=================================+ | `src/cpp/ndi_encode/tally.*` | tally:: class | +------------------------------------+---------------------------------+

Tally outputs are implemented using the Linux kernel LED class. Tally LED entries are created in the device-tree, and the application uses the resulting sysfs entries to control the LED behavior.

The tally class initializes all LEDs to a known state on construction, then the tally process simply loops, checking for updates from the NDI stack. If new tally data is available, the tally LEDs are updated

#### NDI Decode Operation

**Video Output**

\+----------------------------------------------+----------------------------------+ | File(s) | Description | +==============================================+==================================+ | `src/cpp/ndi_decode/video_playback.*` | video\_capture:: class | +----------------------------------------------+----------------------------------+ | `src/vpga/ip/common/Test_Pattern_Gen_YUV444` | Video timing & pattern generator | +----------------------------------------------+----------------------------------+ | `src/fpga/ip/common/Vid_Out.vhd` | Video output DMA logic | +----------------------------------------------+----------------------------------+ | Filename varies | Low level video outputlogic | +----------------------------------------------+----------------------------------+

Video output timing is driven by the low level video logic. This logic is platform specific:

\+------------+-----+------------------------------------------------------+ | Platform | Dir | Implementation | +============+=====+======================================================+ | a10socdk | Tx | Altera HDMI RxTx example design modified for Tx only | +------------+-----+------------------------------------------------------+ | sockit | Tx | RGB clocked video output stream | +------------+-----+------------------------------------------------------+ | ZCU104 | Tx | Xilinx HDMI Tx example design | +------------+-----+------------------------------------------------------+ | Zybo-Z7-20 | Tx | Open-source DVI Tx logic | +------------+-----+------------------------------------------------------+

The software for controlling the low-level video input logic is currently outside the scope of this example. The Zybo-Z7 HDMI output requires no control software, the Xilinx HDMI control software is running bare-metal on one of the Cortex-R5 cores, and the Altera HDMI control software runs on a dedicated NIOS core (see the README file accompanying the hardware project files for details).

The video output logic generates a pixel stream with programmable timings based on the video clock which is programmable on most platforms to support multiple resolutions. The video pixel data can be a hardware generated test pattern or the video data can be filled in with data from memory for video playback. The vertical sync interrupt from Vid\_Out.vhd is used to drive all output operations, and an NDI FrameSync instance is used to synchronize the received NDI video stream with the hardware output timing.

**Video output process flow:**

\####### video\_playback::pull\_frames thread

* Wait for vertical sync interrupt: video\_playback::pull\_frames
* Pull a frame from the NDI FrameSync Instance: network\_recv::get\_video()
* Copy video frame to reserved memory region visible by hardware: video\_decode::add\_frame()
* Push copied frame onto NDI Decode queue: video\_decode::add\_frame()
* Free video frame acquired from the NDI FrameSync Instance

\####### video\_decode::decode\_frames thread

* Wait for new frame to decode
* Setup hardware to decode NDI frame into raw video data: video\_decode::decode\_frame()
* Start hardware decoding: video\_decode::post\_frame()
* Wait for interrupt
* Queue the raw frame for display: video\_playback::send\_frame()
* The frame will display after the next vertical sync

**Video Decompression**

\+---------------------------------------+-------------------------------------------+ | File(s) | Description | +=======================================+===========================================+ | `src/cpp/ndi_decode/video_decode.*` | video\_decode:: class | +---------------------------------------+-------------------------------------------+ | `src/fpga/NDI_Dec/Decode_x4.vhd` | 4-core NDI Hardware Encoder | +---------------------------------------+-------------------------------------------+ | `src/fpga/NDI_Dec/Decode_Xilinx.vhdp` | Single NDI Hardware Encoder core (Xilinx) | +---------------------------------------+-------------------------------------------+ | `src/fpga/NDI_Dec/Decode_Altera.vhdp` | Single NDI Hardware Encoder core (Xilinx) | +---------------------------------------+-------------------------------------------+

One, two, or four copies of the NDI Decoder core are instantiated in hardware (Decode\_x4.vhd). The cores operate in parallel, with each core operating on one or more quarters of the video frame known as a "slice". This increases the effective throughput of the compression core and lowers system latency.

The video\_decode:: class monitors a work queue filled with NDI video frames by the video\_playback:: class (above). When a new frame is received it is sent to hardware for processing. Once the hardware generates an interrupt indicating processing is finished, the software queus the decoded frame for display via

NDI send threads via video\_playback::send\_frame().

**Audio Output**

\+---------------------------------------+-------------------------------------+ | File(s) | Description | +=======================================+=====================================+ | `src/cpp/ndi_decode/audio_playback.*` | audio\_playback:: class | +---------------------------------------+-------------------------------------+ | `src/fpga/ip/common/Aud_Out.vhd` | Audio output DMA logic | +---------------------------------------+-------------------------------------+

Audio output starts with the low level output input logic. This logic supports standard iis serial audio streams. Support for parallel audio data extracted from the HDMI streams will be supported soon. Once assembled into complete audio samples, the data is queued in a FIFO allowing burst writes to system memory.

The audio\_playback:: class manages the playback of audio data from the reserved memory buffer as well as hardware settings controlling things like the data format (typically left-justified).

The audio\_playback::pull\_frames thread loops through a process very similar to the video\_playback thread. An interrupt is generated when a programmed audio DMA completes which drives the timing of the rest of the audio output operations.

**Audio output process flow**

\####### audio\_playback::pull\_frames thread

* Wait for audio interrupt: audio\_playback::pull\_frames
* Pull a frame from the NDI FrameSync Instance: network\_recv::get\_audio()
* Convert audio frame to 32-bit data and copy to reserved memory region visible by hardware: audio\_playback::queue\_frame()
* Free video frame acquired from the NDI FrameSync Instance
* Enable playabck of audio data: audio\_playback::post\_frame()

### Rebuilding from source

\+--------------------+-------------------------------------------------+ | Directory | Description | +====================+=================================================+ | `src/fpga/` | FPGA Project Files | +--------------------+-------------------------------------------------+ | `linux_kernel/` | Linux kernel, U-Boot, and device-tree | +--------------------+-------------------------------------------------+ | `src/cpp/` | C++ Application Project Files | +--------------------+-------------------------------------------------+ | `os_uSD/` | Debian rootfs and uSD images | +--------------------+-------------------------------------------------+

#### FPGA hardware design

The src/fpga/ directory contains projects to build a bit-file for each supported platform. Refer to the README file in the src/fpga directory for full details on building the hardware projects.

#### Boot loader, Linux kernel, and device-tree

The linux\_kernel/ directory contains projects to build a boot-loader, kernel, and device-tree for the supported platforms. The device-tree customizations required for the NDI FPGA hardware are also included in the example projects. Refer to the README file in the linux\_kernel directory for full details.

**Petalinux root file system**

**NOTE:** For Xilinx platforms, the scripts in linux\_kernel create a Petalinux root file-system in addition to the kernel and boot loader. This is a very minimal embedded system and it is difficult to customize since the entire OS is cross-compiled. This example ignores the Petalinux generated rootfs and instead uses a standard Debian OS install which allows for self-hosted compiles and easy modification of the OS using standard package management tools.

For details on building the Debian root file-system, see the section on creating a bootable image (below).

#### Application software

The src/cpp/ directory contains example encode and decode software applications which use the FPGA based NDI logic to (de)compress live video data, and the Embedded NDI SDK to send and receive that data as an NDI stream. Refer to the README file in the src/cpp directory for details.

#### Bootable image including all of the above

The os\_uSD/ directory contains scripts to create a Debian root filesystem as well as a bootable uSD card image. The resulting images contain all prerequisites needed to perform a self-hosted build of the example NDI application (ie: built on the ARM platform and not cross-compiled). Refer to the README file in the os\_uSD directory for full details.

## NDI FPGA IP Core Example Designs

### Directory structure:

\+------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | Directory | Contents | +==================+===========================================================================================================================================================================================================+ | NDI\_Dec/ | VHDL files for the NDI Decoder core. All files should be compiled into the NDI\_Dec VHDL library rather than the default work library | +------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | NDI\_Enc/ | VHDL files for the NDI Encoder core. All files should be compiled into the NDI\_Enc VHDL library rather than the default work library | +------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | ip/altera | Altera specific implementations of memory and FIFO blocks | +------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | ip/common | Source code common to all example designs | +------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | ip/extern | External (non-NewTek) IP used in the example designs | +------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | ip/packages | VHDL Packages defining various types used in the logic | +------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | ip/xilinx | Xilinx specific implementations of memory and FIFO blocks | +------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | a10-socdk-enc/ | Quartus Pro 22.2 NDI Encode project directory targeting the [Arria-10 SoC DevKit](https://www.intel.com/content/www/us/en/products/details/fpga/development-kits/arria/10-sx.html) | +------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | a10-socdk-dec/ | Quartus Pro 22.2 NDI Decode project directory targeting the [Arria-10 SoC DevKit](https://www.intel.com/content/www/us/en/products/details/fpga/development-kits/arria/10-sx.html) | +------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | SoCKit-Dec/ | Quartus 22.1 NDI Decode project directory targeting the [Terasic SoCKit](https://www.terasic.com.tw/cgi-bin/page/archive.pl?Language=English\&No=816) | +------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | ZCU104/ | Vivado 2019.2 NDI Encode project directory targeting the [Xilinx ZCU104](https://www.xilinx.com/products/boards-and-kits/zcu104.html) | +------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | ZCU104-Dec/ | Vivado 2019.2 NDI Decode project directory targeting the [Xilinx ZCU104](https://www.xilinx.com/products/boards-and-kits/zcu104.html) | +------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | Zybo-Z7-20/ | Vivado 2019.2 NDI Encode project directory targeting the [Digilent Zybo-Z7-20](https://store.digilentinc.com/zybo-z7-zynq-7000-arm-fpga-soc-development-board/) | +------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | Zybo-Z7-20-Dec/ | Vivado 2019.2 NDI Decode project directory targeting the [Digilent Zybo-Z7-20](https://store.digilentinc.com/zybo-z7-zynq-7000-arm-fpga-soc-development-board/) | +------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | Zybo-Z7-20-Lite/ | "Lightweight" Vivado 2019.2 NDI Encode project directory targeting the [Digilent Zybo-Z7-20](https://store.digilentinc.com/zybo-z7-zynq-7000-arm-fpga-soc-development-board/) with 16-bit SDRAM interface | +------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ | Arty-Z7-20-Enc/ | Vivado 2019.2 NDI Encode project directory targeting the [Digilent Arty-Z7-20](https://store.digilentinc.com/arty-z7-zynq-7000-soc-development-board/) | +------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

### Build Dependencies

#### Xilinx

All Xilinx projects require Vivado version 2019.2. Any edition of Vivado 2019.2 will work, including the "WebPACK" edition available via free download from Xilinx.

**ZCU104**

You must have a valid license for the Xilinx HDMI IP core. You may use a full license or a free evaluation license, but the license must be installed **prior to launching Vivado** to build the project: https://www.xilinx.com/products/intellectual-property/hdmi.html

**Zybo-Z7-20 / Zybo-Z7-20-Lite / Arty-Z7-20**

Board files from Digilent must be installed in order to build the Zybo example designs. The board files may be obtained from github: https://github.com/Digilent/vivado-boards/ and installation instructions are on the Digilent website: https://reference.digilentinc.com/reference/software/vivado/board-files

#### Intel/Altera

The contents of the file NDI\_Enc/Encode\_Altera.lic (for Encode) and NDI\_Dec/Decode\_Altera.lic (for Decode) must be appended to your Quartus license file. If you do not have a license file, you must create one.

Arria-10 projects require Quartus Pro 22.2.0.

Cyclone-V projects require Quartus 22.1. The examples were compiled using the free "Lite" version, however the "Standard" edition should work as well.

Follow the Altera instructions for installation, including the required manual installation of WSL if you are using Windows.

If you plan to rebuild the NIOS code which reconfigures the gxb transceivers and plls, you need to manually install Eclipse per the Altera Quartus installation instructions. These instructions can also typically be found in the file: \<Quartus installation directory>/nios2eds/bin/README

### Rebuilding

#### Xilinx

Once you have all required dependencies installed (see above), simply open the desired project and build normally. The contents for the block design elements (hard processor system, PLLs, etc) will be regenerated on the first build.

#### Altera

Before you can build the Altera projects, you must generate the hps platform logic from the qsys file.

**Rebuild the hps platform**

* Launch Platform Designer
* Open ndi\_hps.qsys
* Click "Generate HDL..."
* Select VHDL Synthesis output (optional)
* Click "Generate"
* Once the qsys project has been generated, run a normal Quartus compilation

### Known Issues and Limitations:

The video I/O logic (eg: HDMI to SDRAM and SDRAM to HDMI) is intended as an easy to understand minimally functional example and is not recommended for use in production. In particular, the input logic does not gracefully handle corrupt video data, unexpected cable removal, and similar signal quality issues.

It is expected the user of this SDK will have their own custom video I/O logic specific to their target hardware. If this is not the case and a production capable video I/O solution is desired, there are numerous IP cores available from Xilinx, Altera, and 3rd parties capable of supporting all major video interfaces (SDI, HDMI, Display Port, CSI/DSI, etc.).

### NDI IP Files and Use

#### NDI Encoder

Two VHDL files are required to use the NDI Encoder. Both files should be compiled into the NDI\_Enc library in the following order:

* Xilinx Projects:

`Encode_Xilinx.vhdp`\
`Encode_x4.vhd`

* Altera Projects:

`Encode_Altera.vhd`\
`Encode_x4.vhd`

There is also a component declaration file Enc\_Core\_Comp.vhd for use as a reference when instantiating the encoder core. Each encoder core Enc\_Core\_E includes a register I/O interface, a read-only bus master interface for reading raw video data, and a write-only bus master interface for writing compressed NDI data. Note the raw video memory and the compressed NDI memory do not need to be the same physical bank of memory. Using two memory banks can improve system performance, particularly on lower-end devices such as the Zynq 7000 and Cyclone-V families.

#### NDI Decoder

Two VHDL files are required to use the NDI Decoder. Both files should be compiled into the NDI\_Dec library in the following order:

* Xilinx Projects:

`Decode_Xilinx.vhdp`\
`Decode_x4.vhd`

* Altera Projects:

`Decode_Altera.vhd`\
`Decode_x4.vhd`

There is also a component declaration file Dec\_Core\_Comp.vhd for use as a reference when instantiating the encoder core. Each decoder core Dec\_Core\_E includes a register I/O interface, a read-only bus master interface for reading compressed NDI data, and a write-only bus master interface for writing raw video data. Note the raw video memory and the compressed NDI memory do not need to be the same physical bank of memory. Using two memory banks can improve system performance, particularly on lower-end devices such as the Zynq 7000 and Cyclone-V families.

#### SDRAM Bus Interfaces

The bus-mastering memory interfaces are natively a sub-set of the Altera Avalon interface specification. These buses may be tied directly to an Avalon interface port in an Altera design. For Xilinx designs, clear-text bus shim logic is provided to convert the Avalon bus subset used by the Encoder core into a more standard AXI interface:

\+------------------+------------------------------------------------------+ | File | Function | +==================+======================================================+ | Avl\_Axi\_Rd.vhd | Translates Avalon read bus to AXI read bus | +------------------+------------------------------------------------------+ | Avl\_Axi\_Wr.vhd | Translates Avalon write bus to AXI write bus | +------------------+------------------------------------------------------+ | Avl2\_Axi\_rd.vhd | Merges 2 Avalon read buses into one AXI read bus | +------------------+------------------------------------------------------+ | AXI\_Reg\_Wr.vhd | Interface AXI write bus to simple register write bus | +------------------+------------------------------------------------------+ | AXI\_Reg\_Rd.vhd | Interface AXI read bus to simple register read bus | +------------------+------------------------------------------------------+

Usage examples for these files can be found in the top-level files for the example designs.

**NDI Encode**

**Writes (compressed NDI data)**

There is clear-text logic in the Encode\_x4 file which merges the four lower bandwidth compressed write streams into a single write stream. The raw (uncompressed) audio data is also merged into this stream.

This write bus is connected to a cache coherent port to the hard processor system, eliminating the need for a kernel mode DMA driver.

**Reads (raw video data)**

For maximum performance, the four read streams are all brought out to the top level to provide maximum flexibility in scheduling SDRAM transactions. The NDI Encoder cores will tolerate fairly high latency on raw video reads, but require sufficient overall bandwidth to avoid stalling the NDI cores. Each of the NDI cores can process one pixel element (Y or C) per clock.

**NDI Decode**

**Reads (compressed NDI data)**

There is clear-text logic in the Decode\_x4 file which merges the four lower bandwidth compressed Read streams into a single AXI read stream.

This read bus is connected to a cache coherent port to the hard processor system, eliminating the need for a kernel mode DMA driver.

**Writes (raw video data)**

For maximum performance, the four write streams are all brought out to the top level to provide maximum flexibility in scheduling SDRAM transactions. The NDI Decoder cores will tolerate fairly high latency on raw video writess, but require sufficient overall bandwidth to avoid stalling the NDI cores. Each of the NDI cores can process one pixel element (Y or C) per clock. For maximum performance, the four write streams are all brought out to the top level.

#### Full vs. Lite Encode Designs

The Zybo-Z7-20-Lite and Arty-Z7-20 example designs are intended as a reference for a more limited platform. Only one SDRAM chip is used (16-bit SDRAM interface) and only two encoder cores are instantiated. This implementation is still capable of operation at resolutions up to 1080p60.

All other example designs instantiate a "full" implementation which includes four NDI Encode or Decode cores providing maximum performance and minimum latency.

### HDMI Input Logic

#### Arria-10 SoC DevKit

The a10-socdk-enc design targeting the Arria-10 SoC Development Kit uses a slightly modified version of the Altera Arria10 HDMI Rx-Tx Retransmit HDMI example design. The top-level of this example design is modified to export the recevied HDMI video, audio, and AVI data to the top-level where they are connected to the video and audio input logic.

**Rebuilding the NIOS HDMI software**

* Insure you have manually installed Eclipse per the Altera documentation
* In Quartus, select Tools -> Nios II Software Build Tools for Eclipse
* Use \<NDI SDK Path>/ndi-fpga-refernce/a10-socdk-enc/software as the Workspace directory
* Create a new project and BSP in Eclipse
  * File -> New -> Nios II Application and BSP from Template
  * SOPC Information File name: \<NDI SDK Path>/ndi-fpga-reference/a10-socdk-enc/rtl/nios/nios.sopcinfo
  * Project Name: tx\_control
  * Project location: \<NDI SDK Path>/ndi-fpga-refernce/a10-socdk-enc/software/tx\_control
    * NOTE: Remove rtl from the default path
  * Project Template: Blank Project
  * Click: Finish
* Modify BSP settings
  * Right-click tx\_control\_bsp
  * Nios II -> BSP Editor...
  * In the Main tab, under Settings -> Common
    * Check: enable\_small\_c\_library
    * Check: enable\_reduced\_device\_drivers
    * File -> Save
    * Click: Generate BSP
  * Close BSP Editor
* Using a file browser, navigate to \<NDI SDK Path>/ndi-fpga-refernce/a10-socdk-enc/software/tx\_control\_src
* Select all files, right-click, and select: Copy
* In Eclipse, right-click tx\_control and select: Paste
* Build: Project -> Build All (or press ctrl+B)
* Update MIF file used to build FPGA
  * Right-click tx\_control -> Make Targets -> Build
  * Select mem\_init\_generate
  * Click: Build

#### Zynq 7000 HDMI Input

The Zynq 7000 (Arty and Zybo Z7 boards) designs use the ISERDESE2 and IDELAYE2 primitives to convert the serial HDMI data into parallel video. Two open-source cores are supported, each with it's own advantages and disadvantages. Currently, the Digilent dvi2rgb core is used by default, however this can be changed by modifying the USE\_DVI generic passed to the top-level logic during synthesis:

* Select Project Manager -> Settings
* Select Project Settings -> General
* Click the `...` button next to Generics/Parameters
* Edit the value for the USE\_DVI generic as desired:
  * true = Use the Digilent dvi2rgb core (default)
  * false = Use the Hamsterworks HDMI core

**Digilent dvi2rgb core**

This core only supports DVI inputs, so no HDMI features are supported (eg: Data Island Packets including AVI InfoFrames and embedded audio). The low-level serial-to-parallel conversion works very well, however, and automatically aligns each input lane to the center of the "eye" in the incoming bitstream, then insures each recovered parallel stream is aligned with the other two.

**Hamsterworks HDMI core**

This core supports HDMI features including InfoFrame and Audio Sample Packets, but the low-level serial-to-parallel logic does not align to the center of the "eye" in the incoming bitstream. This results in occasional glitches in the video input stream, so the dvi2rgb core is currently used as the default.

#### ZCU104 HDMI Input

The ZCU104 board uses the Xilinx HDMI 2.0 IP core. A full or evaluation license must be installed for this core prior to building the design with Vivado. This IP core also requires control software to monitor the incoming HDMI signal and make any adjustments required. This software is currently running "bare-metal" on one of the Cortex-R5 cores.

**Rebuilding the Cortex-R5 HDMI Software**

* Compile the ZCU104 FPGA design
* Export the hardware to the SDK
  * Select File -> Export -> Export Hardware...
  * Select \<Local to Project> and click OK
* Launch the SDK File -> Launch SDK
  * Exported location: \<Local to Project>
  * Workspace: \<Local to Project>
  * Click OK
* Disable automatic builds while we are creating a new BSP and Project
  * Insure Project -> Build Automatically is not checked
* Create a new BSP File -> New -> Board Support Package
  * Set Project name to standalone\_bsp\_R5\_0
  * Leave Use default location set
  * Leave Hardware Platform set to ZCU104\_NDI\_HDMI\_hw\_platform\_0
  * Set CPU to psu\_cortexr5\_0
  * Select standalone Board Support Package OS
  * Click Finish
  * The Board Support Package Settings window should open
  * Select Overview if not already selected
  * Select the libmetal and openamp libraries (used to communicate with Linux), leave all other libraries unselected
  * Switch the console to UART1
    * Select Standalone
    * Set the Value for stdin to psu\_uart\_1
    * Set the Value for stdout to psu\_uart\_1
  * Click OK
  * Be patient, the BSP generation can take a while.
* Import the HDMI Rx example project
  * Open the system.mss file if it is not already open:
    * `Project Exporer -> standalone_bsp_R5_0 -> system.mss`
  * Scroll down to v\_hdmi\_rx\_ss under Peripherial Drivers (it is near the bottom)
  * Select Import Examples
  * Select RxOnly\_R5
  * Click OK
  * Be patient!
* Rename the project (Optional)
  * Right-click standalone\_bsp\_R5\_0\_RxOnly\_R5\_1
  * Select Rename
  * Change name to NewTek\_HDMI\_RxOnly\_R5
  * All file paths below are relative to this directory!
* Update the DDR memory region in the linker script
  * Open the file src/lscript.ld
  * Set the psu\_r5\_ddr\_0\_MEM\_0 region details to match the rproc\_0\_reserved reserved memory region in the device-tree and save the file Base Address : 0x3ed00000 Size : 0x80000
* Change HDMI code to use UART1
  * Open src/xhdmi\_example.h
  * At line 109, change the UART\_BASEADDRESS to use UART1 and save the file #define UART\_BASEADDR XPAR\_XUARTPS\_1\_BASEADDR
* Update EDID Data (Optional)
  * Open src/xhdmi\_edid.h
  * Edit constant Edid\[] as desired
* Build the project Project -> Build All
* Re-enable automatic builds if desired: Project -> Build Automatically
* Copy the elf file to the ZCU104
  * The elf file can be found at the following path
  * `<sdkdir>/<projdir>/Debug/<projname>.elf`
  * Copy this file to `/lib/firmware/` on the ZCU104
  * Make sure you reference this filename when launching the R5 via remoteproc!

### HDMI Output Logic

#### Arria-10 SoC DevKit

The a10-socdk-dec design targeting the Arria-10 SoC Development Kit uses a modified version of the Altera Arria10 HDMI Rx-Tx Retransmit HDMI example design. All Rx logic and the RxTx loopback logic is removed and the HDMI Tx fpll and iopll blocks use the REFCLK\_SDI signal as a clock reference instead of the HDMI Rx clock. The tx\_control NIOS code has been updated to reconfigure the output divisor for the fpll and iopll instance to switch between video modes.

**Rebuilding the NIOS HDMI software**

* Insure you have manually installed Eclipse per the Altera documentation
* In Quartus, select Tools -> Nios II Software Build Tools for Eclipse
* Use \<NDI SDK Path>/ndi-fpga-refernce/a10-socdk-dec/software as the Workspace directory
* Create a new project and BSP in Eclipse
  * File -> New -> Nios II Application and BSP from Template
  * SOPC Information File name: \<NDI SDK Path>/ndi-fpga-reference/a10-socdk-dec/rtl/nios/nios.sopcinfo
  * Project Name: tx\_control
  * Project location: \<NDI SDK Path>/ndi-fpga-refernce/a10-socdk-dec/software/tx\_control
    * NOTE: Remove rtl from the default path
  * Project Template: Blank Project
  * Click: Finish
* Modify BSP settings
  * Right-click tx\_control\_bsp
  * Nios II -> BSP Editor...
  * In the Main tab, under Settings -> Common
    * Check: enable\_small\_c\_library
    * Check: enable\_reduced\_device\_drivers
    * File -> Save
    * Click: Generate BSP
  * Close BSP Editor
* Using a file browser, navigate to \<NDI SDK Path>/ndi-fpga-refernce/a10-socdk-dec/software/tx\_control\_src
* Select all files, right-click, and select: Copy
* In Eclipse, right-click tx\_control and select: Paste
* Build: Project -> Build All (or press ctrl+B)
* Update MIF file used to build FPGA
  * Right-click tx\_control -> Make Targets -> Build
  * Select mem\_init\_generate
  * Click: Build

#### Zybo-Z7-20 HDMI Output

The Zybo-Z7 board uses the DVI output logic from the "Hamsterworks" HDMI project (ip/extern/HDMI/src/dvid\_output.vhd). This is a simple DVI output and no HDMI features (e.g., emedded audio) are currently supported. Both 74.25 MHz (720p) and 148.5 MHz (1080p60) pixel rates are supported.

#### ZCU104 HDMI Output

The ZCU104 board uses the Xilinx HDMI 2.0 IP core. A full or evaluation license must be installed for this core prior to building the design with Vivado. This IP core also requires control software to configure the HDMI IP core. This software is currently running "bare-metal" on one of the Cortex-R5 cores.

**Rebuilding the Cortex-R5 HDMI Software**

* Compile the ZCU104 FPGA design
* Export the hardware to the SDK
  * Select File -> Export -> Export Hardware...
  * Select \<Local to Project> and click OK
* Launch the SDK File -> Launch SDK
  * Exported location: \<Local to Project>
  * Workspace: \<Local to Project>
  * Click OK
* Disable automatic builds while we are creating a new BSP and Project
  * Insure Project -> Build Automatically is not checked
* Create a new BSP File -> New -> Board Support Package
  * Set Project name to standalone\_bsp\_R5\_0
  * Leave Use default location set
  * Leave Hardware Platform set to ZCU104\_Dec\_hw\_platform\_0
  * Set CPU to psu\_cortexr5\_0
  * Select `standalone` Board Support Package OS
  * Click Finish
  * The Board Support Package Settings window should open
  * Select `Overview` if not already selected
  * Select the libmetal and openamp libraries (used to communicate with Linux), leave all other libraries unselected
  * Switch the console to UART1
    * Select Standalone
    * Set the Value for stdin to psu\_uart\_1
    * Set the Value for stdout to psu\_uart\_1
  * Click OK
  * Be patient, the BSP generation can take a while.
* Import the HDMI Tx example project
  * Open the system.mss file if it is not already open:
    * `Project Exporer -> standalone_bsp_R5_0 -> system.mss`
  * Scroll down to v\_hdmi\_tx\_ss under Peripherial Drivers (it is near the bottom)
  * Select Import Examples
  * Select TxOnly\_R5
  * Click OK
  * Be patient!
* Rename the project (Optional)
  * Right-click standalone\_bsp\_R5\_0\_RxOnly\_R5\_1
  * Select Rename
  * Change name to NewTek\_HDMI\_TxOnly\_R5\_1080p60
  * All file paths below are relative to this directory!
* Update the DDR memory region in the linker script
  * Open the file src/lscript.ld
  * Set the psu\_r5\_ddr\_0\_MEM\_0 region details to match the rproc\_0\_reserved reserved memory region in the device-tree and save the file Base Address : 0x3ed00000 Size : 0x80000
* Change HDMI code to use UART1
  * Open src/xhdmi\_example.h
  * At line 119, change the UART\_BASEADDRESS to use UART1 and save the file #define UART\_BASEADDR XPAR\_XUARTPS\_1\_BASEADDR
* Update the default video mode
  * Open src/xhdmi\_example.c
  * Edit the default video resolution on line 2159 (optional)
  * Change the colorspace on line 2160 to XVIDC\_CSF\_YCRCB\_420 for 4Kp60 and XVIDC\_CSF\_YCRCB\_422 for other modes
  * Possible values are listed in the `XVidC_VideoMode` enum in the file standalone\_bsp\_R5\_0/psu\_cortexr5\_0/include/xvidc.h
* Fix build errors related to not having a test pattern generator:
  * Copy the tpg header files xv\_tpg.h and xv\_tpg\_hw.h from the Xilinx SDK install directory into the BSP include folder
    * `C:\Xilinx\SDK\2018.1\data\embeddedsw\XilinxProcessorIPLib\drivers\v_tpg_v8_0\src\*.h`
    * `standalone_bsp_R5_0/psu_cortexr5_0/include/`
  * Edit xhdmi\_example.c
    * Comment the contents of the XV\_ConfigTpg function (lines 311-353)
    * Comment the contents of the ResetTpg function (lines 358-363)
    * Comment the assignment to `Pattern` (line 1176)
    * Comment the call to XV\_ConfigTpg (line 1179)
  * Edit xhdmi\_menu.c
    * Comment the assignment to Pattern (line 1526)
    * Comment the call to XV\_ConfigTpg (line 1529)
* Build the project Project -> Build All
* Re-enable automatic builds if desired: Project -> Build Automatically
* Copy the elf file to the ZCU104
  * The elf file can be found at the following path
  * `<sdkdir>/<projdir>/Debug/<projname>.elf`
  * Copy this file to /lib/firmware/ on the ZCU104
  * Make sure you reference this filename when launching the R5 via remoteproc!

#### SoCKit-Dec

The SoCKit-Dec design targeting the Cyclone-V SoCKit Development board video output implementation (DVI\_TX.vhd) converts the YUV video data from the NDI decoder into RGB and drives both the on-board VGA output and the (optional) Terasic DVH-HSMC daughtercard via the HSMC connector.

This is a simple DVI output and no HDMI features (e.g., emedded audio) are currently supported. Both 74.25 MHz (720p) and 148.5 MHz (1080p60) pixel rates are supported.

## NDI Embedded Application Examples

This is a set of example applications illustrating the use of the FPGA based NDI encoder with the Embedded NDI SDK. For full details and a theory of operation covering the entire system (software, hardware, and OS), refer to the top-level README file.

Source code for the examples is organized into the following subdirectories:

\+--------------+------------------------------------------------------------+ | Directory | Contents | +==============+============================================================+ | ndi\_common/ | Files common to all NDI applications | +--------------+------------------------------------------------------------+ | ndi\_decode/ | Example NDI Decoder application (eg: TV or Monitor) | +--------------+------------------------------------------------------------+ | ndi\_encode/ | Example NDI Encoder application (eg: Camera or NDI Source) | +--------------+------------------------------------------------------------+ | ndi\_util/ | Debug and utility applications | +--------------+------------------------------------------------------------+

### Getting Started

#### Prerequisites

The following prerequisites must be installed before you can build the example applications.

**libuio**

* https://github.com/Linutronix/libuio

libuio is used to access the FPGA hardware via entries in the device-tree

The available uSD card images have libuio packages compiled from source (see \~/libuio) and the debian packages installed. For other platforms, libuio will need to be compiled and installed manually.

**NDI Embedded SDK Library**

The NDI Embedded SDK allows the sending and receiving of compressed frames, which is required to use the FPGA encoder. To install the library, simply copy the files to an appropriate location for your system. The example uSD card images have the NDI library files already installed in /usr/local/lib and /usr/local/include.

#### Building and Installing

Once the prerequisites have been installed, building the applications is straight-forward:

`cd <NDI_SDK>/fpga_reference_design/src/cpp`\
`make`\
`make install`

The applications will be installed with setuid privileges to the /usr/local/bin/ directory.

### Usage

See the README.md files in each subdirectory for usage details specific to each available application.

## Altera Bootloader and Linux Kernel

The U-Boot bootloader and Linux kernel are built from git repositories provided by altera-opensource on github. See RocketBoards.org for full details.

### Directory Structure:

\+-------------------+--------------------------------------------------+ | Directory | Target Development Board | +===================+==================================================+ | a10-socdk | Arria-10 SoC Development Kit | +-------------------+--------------------------------------------------+ | SoCKit | Arrow/Terasic SoCKit | +-------------------+--------------------------------------------------+

### Building an Existing Project

The build instructions from Altera:

https://www.rocketboards.org/foswiki/Documentation/BuildingBootloaderCycloneVAndArria10

...involve quite a few manual steps. These steps have mostly been automated via a Makefile, with some additional changes to customize the kernel as needed for the NDI Advanced SDK (enable UIO devices and create custom device-tree nodes).

The Makefile will checkout the source directories, patch and configure as required, then build the files required to boot and create a uSD image.

The configuration snippet uio.fragment enables the required uio kernel modules.

The patch file 0001-NewTek-Altera-NDI-device-tree-entries.patch adds the required device-tree nodes for logic in the FPGA fabric. See the instructions from Altera for full details:

https://www.rocketboards.org/foswiki/Documentation/HOWTOCreateADeviceTree

#### Note:

If you follow the RocketBoards instructions for generating the Cyclone-V boot files, the FPGA will not be programmed by default. The provided Makefile builds a u-boot.scr file which is executed by the default Altera U-Boot bootcmd. This script programs the FPGA prior to loading the Linux kernel and device-tree.

## PetaLinux Projects for NDI Xilinx Hardware Platforms

All steps below require you have an appropriate version of the Xilinx PetaLinux tools installed on your development machine. Refer to Xilinx UG1144 "PetaLinux tools Documentation: Reference Guide" for detailed instructions on installing PetaLinux and the required prerequisites.

### Directory Structure:

\+------------------------+------------------------------------------------------------------------+ | Directory | Contents | +========================+========================================================================+ | Arty-Z7-20.2019.2 | PetaLinux 2019.2 project for the Digilent Arty-Z7-20 | +------------------------+------------------------------------------------------------------------+ | ZCU104-Dec.2019.2 | PetaLinux 2019.2 project for the Xilinx ZCU104 (Decode) | +------------------------+------------------------------------------------------------------------+ | ZCU104-Enc.2019.2 | PetaLinux 2019.2 project for the Xilinx ZCU104 (Encode) | +------------------------+------------------------------------------------------------------------+ | Zybo-Z7-20.2019.2 | PetaLinux 2019.2 project for the Digilent Zybo-Z7-20 | +------------------------+------------------------------------------------------------------------+ | Zybo-Z7-20-Lite.2019.2 | PetaLinux 2019.2 project for the Digilent Zybo-Z7-20 with 16-bit SDRAM | +------------------------+------------------------------------------------------------------------+

#### Notes

The Ultrascale+ designs have different FPGA fabric hardware for HDMI Rx vs Tx and thus have separate directories for NDI Encode vs. Decode. The other platforms can use the same kernel, u-boot, and device-tree for both encode and decode.

### Building an Existing Project

To build one of the existing PetaLinux projects provided for the supported development boards, use the following general procedure:

`# Setup the PetaLinux environment`\
`source /path/to/petalinux/version/settings.sh`\
\
`# Change to the appropriate project directory`\
`cd projectdir/`\
\
`# Run petalinux-configure to populate the project structure`\
`petalinux-configure`\
\
`# Build the project`\
`petalinux-build`\
\
`# Update the boot files`\
`./boot.sh`

#### Notes

* The petalinux-build step will occasionally fail when performing a full build from scratch. Simply re-run the petalinux-build command and the build will continue. Sometimes it is necessary to re-run the petalinux-build step several times.
* If the petalinux-build step repeatedly fails in the same place, examine the log files as there may be missing dependencies.
* Output files will be in the projectdir/images/linux/ directory.
* projectdir naming convention is \<platform>.\<petalinux version>

### Building a New Project From Scratch

If you are not using one of the supported development boards you may need to build a PetaLinux project from scratch. Full details for working with PetaLinux can be found in Xilinx UG1144 "PetaLinux Tools Documentation: Reference Guide". A brief summary of one possible work flow is provided below.

* Build the hardware project

&#x20;

* Create a Vivado project as required for your system and build a bit file.

&#x20;

* Export the hardware to the SDK

&#x20;

* In Vivado, select the File -> Export -> Export Hardware menu option

&#x20;

* Create a new PetaLinux project

`# For Zynq-7000`\
`petalinux-create -t project -n <projectname> --template zynq`\
\
`# For Zynq Ultrascale`\
`petalinux-create -t project -n <projectname> --template zynqMP`

* Import the hardware definitions

`cd <projectname>`\
`petalinux-config --get-hw-description=/path/to/vivadoproject/project.sdk`

* Customize the PetaLinux project (optional)

`# Configure petalinux settings (eg: boot loader options, kernel command line, etc)`\
`petalinux-config`\
\
`# Configure the Linux kernel`\
`petalinux-config -c kernel`

* Customize system-user.dtsi to reflect the FPGA hardware

&#x20;

* Xilinx IP cores should be automatically added to the device tree, but details for any custom FPGA logic must be added manually. Details will depend on the specifics of your FPGA project. Refer to the existing projects for examples:

`projectdir/project-spec/meta-user/recipes-bsp/device-tree/files/system-user.dtsi`

* Build the PetaLinux project

`petalinux-build`

* Note: This command will occasionally fail with the error "timeout while establishing a connection with SDK". If this happens simply run the build command again.

&#x20;

* Create boot files

&#x20;

*   The generated boot loader files and possibly the FPGA bit file need to be packaged into files suitable for copying to your boot media. The specific command options needed will depend on the configuration options chosen for your system. The following example is for a uSD image with the boot loader programming the FPGA prior to booting the Linux kernel:

    ```
    petalinux-package --boot --fpga /path/to/fpga.bit --uboot

    # If you have previously generated the boot files and want to rebuild them,
    # you need to add the --force flag
    petalinux-package --boot --fpga /path/to/fpga.bit --uboot --force
    ```

### Rebuilding After Changes

#### Xilinx FPGA IP Added/Removed

If you add or remove any Xilinx IP blocks with Linux kernel drivers, the PetaLinux project needs to be updated to incorporate the changes. The easiest way to do this is simply follow the directions for "Building a new project from scratch", above, skipping the petalinux-create command. This will complete much faster than the initial build, since most of the build artifacts are cached and can be reused.

The minimum process to update a project with new hardware definitions is:

* Export the hardware to the SDK

&#x20;

* In Vivado, select the File -> Export -> Export Hardware menu option

&#x20;

* Import the updated hardware definitions

`cd <projectname>`\
`petalinux-config --get-hw-description=/path/to/vivadoproject/project.sdk`

#### FPGA Bit File Updated

If you update the FPGA bit file with hardware changes that do not alter the automatically generated device-tree, the update process is much simpler. Simply rebuild the boot files using the updated bit file:

`# Rebuild the boot files`\
`petalinux-package --boot --fpga /path/to/fpga.bit --uboot --force`

If you are using the default directory structure, you can simply run one of the provided boot scripts:

`cd <projectname>`\
`./boot.enc.sh # For NDI Encode`\
`./boot.dec.sh # For NDI Decode`

The generated boot files will be in the image/linux directory, and will also be copied to a subdirectory based on the FPGA bitfile chosen (Encode/Decode & Full/Lite).

#### Device Tree Changes

If you update the system-user.dtsi file, the system device tree needs to be updated and the changes need to be incorporated into the boot loader.

`# Rebuild just the boot loader`\
`petalinux-build -c u-boot`\
\
`# Rebuild the boot files`\
`petalinux-package --boot --fpga /path/to/fpga.bit --uboot --force`

## uSD Image Builder

Scripts to build uSD images for NDI demo platforms based on Debian.

\+----------------------+--------------------------------------------------------+ | File | Description | +======================+========================================================+ | README.md | This file | +----------------------+--------------------------------------------------------+ | CHANGELOG.md | List of notable changes | +----------------------+--------------------------------------------------------+ | a10-socdk.conf | Config file for Arria-10 SoC Development Kit | +----------------------+--------------------------------------------------------+ | composite.armhf.conf | Config file for combined Zybo-Z7 & SoCKit image | +----------------------+--------------------------------------------------------+ | zcu104.conf | Config file for ZCU104 | +----------------------+--------------------------------------------------------+ | zybo\_z7-20.conf | Config file for Zybo-Z7-20 | +----------------------+--------------------------------------------------------+ | build.sh | Main image build script | +----------------------+--------------------------------------------------------+ | common.sh | Functions common to several scripts | +----------------------+--------------------------------------------------------+ | boot.sh | Extracts boot files from a PetaLinux project | +----------------------+--------------------------------------------------------+ | debootstrap.sh | Creates a Debian root filesystem from scratch | +----------------------+--------------------------------------------------------+ | second-stage.sh | The debootstrap second-stage and rootfs customizations | +----------------------+--------------------------------------------------------+ | mk.uSD.sh | Creates a uSD image from boot files and a rootfs | +----------------------+--------------------------------------------------------+ | part.txt | Partion table for the uSD | +----------------------+--------------------------------------------------------+ | part.a2.txt | Partion table for uSD images with Altera A2 partition | +----------------------+--------------------------------------------------------+ | update.tgz.sh | Example to generate optional tgz files | +----------------------+--------------------------------------------------------+

The build scripts are based on a config script which provides details needed to create an image. The main details required are a pointer to the boot files and various architecture specific information (eg: the Debian architecture name and the qemu static binary to use for an emulated chroot environment). Various other settings may be tweaked as well, including the uSD target size, system host name, etc. Refer to the existing \*.conf files for details.

### Build Dependencies

* `debootstrap`

&#x20;

* Used to create a root filesystem image. Currently using version 1.0.123

&#x20;

* `qemu-static`

&#x20;

* Required to run debootstrap second-stage. Currently using version 5.2.0

### Build an image

To build an image from scratch, simply run the build.sh script with the desired configuration specified:

`./build.sh zcu104.conf`

### Rebuild an image

It is often not necessary to run all steps from scratch. Once an initial build has created the required bootfs and rootfs directories, each component of the uSD image may be updated independently. The components that makeup the uSD image are:

* `bootfs.<board>/`

&#x20;

* This directory contains the Xilinx boot files and linux kernel modules extracted from the PetaLinux project. This directory is created and updated by running the `boot.sh` script.

&#x20;

* `../linux_kernel/<board>/boot`

&#x20;

* This directory contains the Altera boot files and linux kernel modules. This directory is created and updated by running make in the `../linux_kernel/<board>` directory.

&#x20;

* `root.<deb_arch>.tgz`

&#x20;

*   This file is manually generated and if present will be extracted to the root directory of the target file-system by the root user. This is currently used to install pre-compiled NDI example binaries to /usr/local/bin and the required shared libraries to /usr/local/lib.

    This file is processed by the `second-stage.sh` script, so any changes require re-running `debootstrap.sh` or manually applying the changes to the rootfs directory.

    Note this file should include full paths preceded by a leading ./ eg: `./usr/local/lib/file.so`. Below is one example of how to properly create this file:

`# Start at the root directory`\
`cd /`\
\
`# Create a tar archive with a leading ./ and put it in /tmp`\
`tar -czvf /tmp/root.armhf.tgz ./usr/local/bin/*`

* `user.<deb_arch>.tgz`

&#x20;

*   This file is similar to the `root.<deb_arch>.tgz` file, above, except it is extracted into the default user's home directory by the default user. This file is used to install the HDMI start and stop scripts for the ZCU104, and is unused by the Zybo-Z7-20 example. Place any files that need to be owned by the default user and thus cannot be placed in the root archive (since the default username, uid/gid, and home directory may change) into this archive.

    This file is processed by the `second-stage.sh` script, so any changes require re-running `debootstrap.sh` or manually applying the changes to the rootfs directory.

&#x20;

* `rootfs.<deb_arch>/`

&#x20;

*   This directory contains a stock Debian install generated by debootstrap along with some modifications required to make a usable image (eg: create /etc/fstab and enable networking) and tweaks required for this example (eg: build and install the libuio package). This directory is generated by running the `debootstrap.sh` script, while most of the customizations to the rootfs occur in the `second-stage.sh` script.

    Running debootstrap.sh deletes any existing rootfs and recreates it from scratch, which is a lengthy process and does not allow for modifications other than editing the `second-stage.sh` script. Once created, however, the rootfs directory may be edited manually, just be careful with file-system permissions and user ids. It is also possible to chroot to the rootfs and execute native commands, eg:

`# Setup shell variables for our configuration`\
`source zcu104.conf`\
\
`# Open a native shell in the rootfs`\
`sudo LANG=C.UTF-8 chroot $ROOTFS $QEMU /bin/bash`

Once any desired changes have been made to the above components, a new image can be created by running:

`sudo ./mk.uSD.sh <config>.conf`

## uSD Images for NDI Demo Platforms

### Login Details

```
Default username: debian
Default password: temppwd

sudo is enabled without password for root access
root login is disabled
```

### Obtaining the uSD Image Files

The uSD images may be downloaded via the following URLs:

\+------------------+---------------+-----------------------------------+ | Manufacturer | Board | uSD Image Download URL | +==================+===============+===================================+ | Altera | A10-SoCDK | http://ndi.link/NDIUSDA10 | +------------------+---------------+-----------------------------------+ | Digilent | Zybo-Z7-20 | http://ndi.link/NDIUSDZYBO | +------------------+---------------+-----------------------------------+ | Digilent | Arty-Z7-20 | http://ndi.link/NDIUSDZYBO | +------------------+---------------+-----------------------------------+ | Terasic | SoCKit | http://ndi.link/NDIUSDZYBO | +------------------+---------------+-----------------------------------+ | Xilinx | ZCU104 | http://ndi.link/NDIUSDZCU | +------------------+---------------+-----------------------------------+

**NOTE:** The same uSD image is used for the SoCKit board (Intel FPGA), the Zybo-Z7-20 board (Xilinx FPGA), and the Arty-Z7-20 (Xilinx FPGA). The uSD image contains boot files for all three systems and may be used without modification with either the Zybo-Z7-20 or the A10-SoCDK. To use the image with the Arty-Z7-20, you must update the boot files on the FAT partition prior to booting on the Arty development board.

Each uSD image is a zip archive of three files:

\+-----------+--------------------------------------------------------------+ | Extension | Description | +===========+==============================================================+ | .img.xz | The image file (xz compressed) | +-----------+--------------------------------------------------------------+ | .bmap | A block map file for use with bmaptools (highly recommended) | +-----------+--------------------------------------------------------------+ | .md5 | Checksum of the above two files | +-----------+--------------------------------------------------------------+

Most users will likely only need to use the img.xz file.

### Writing the Image to a uSD Card

Use of bmaptool at a Linux command line is the most efficient way to write the image file to a uSD card. If this is not an option or is considered too complex, any uSD imaging tool which supports xz compression can be used. [Etcher](https://etcher.io/) is a good choice if you do not already have a preference.

### Using the Image

Insert the programmed uSD card into the development board and boot as usual. It is recommended you connect to the serial terminal (via USB). All images use the following console settings:

`115200`` baud, 8 data bits, 1 stop bit, no parity`

The system will boot and display the IP address, as well as the default username and password at the serial console login prompt. You may login either via the serial console or remotely via ssh.

#### First Boot

The initial boot will take somewhat longer than normal as a new set of ssh keys are generated and the root partition is expanded to fill the uSD card.

### Running the Example NDI Encode Application

An example NDI source application is provided. This application is launched automatically at boot and may also be run from the command line. The utility is named ndi\_encode and is capable of using a generated video pattern or live HDMI input as the video source. The application includes various command-line options to control the initial video mode as well as a command interpreter that can be used to change the operating mode at run-time. To launch the demo from the command line, simply run one of the following commands:

`# Live HDMI input:`\
`ndi_encode`\
\
`# Pattern generator:`\
`ndi_encode`` -P`

Once running, the operating mode can be changed by typing commands at the console. Type "help" to get a list of commands.

If running with HDMI input on the ZCU104, the second virtual serial port is used as a console for the Cortex-R5 running the HDMI monitoring software. Connection details are the same as the Linux console:

`115200`` baud, 8 data bits, 1 stop bit, no parity`

The R5 will report changes in the incoming HDMI stream on this port.

#### Known Issues and Limitations

While the NDI Encoder core is fully functional the example software and demo platforms currently have some limitations:

* This version of the NDI Embedded SDK is designed for development use and will run on a stream for 30 minutes. For a commercial use license, please email [licensing@ndi.com](file:///C:/Users/hky/Desktop/licensing@ndi.com).
* Due to the asynchronous nature of the startup scripts, the system clock may be adjusted after the ndi\_encode example is launched. If this happens, it may trigger the 30 minute timeout even though the application may not have been running for 30 minutes of "wall clock" time. Kill and re-start the demo application to resume NDI streaming.
* The HDMI input hardware for the ZCU104 uses an evaluation license, so HDMI video will stop being received and the image will turn "green" after apx. 1 hour of operation. Reprogram the FPGA (eg: reboot) to restore normal operation.
* While HDMI input on the ZCU104 is functional, the HDMI management software (running on one of the R5 cores) does not communicate with the ndi\_encode application. The video format tracking hardware is used to detect video format changes, however more details regarding the HDMI signal are available via the HDMI management software.

### Running the Example NDI Decode Application

An example NDI receive application is provided. This application is launched manually by executing `ndi_decode`. When launched with no options the utility will attempt to locate NDI sources on the local network and will automatically connect to the first source found. A specific NDI source may be specified using the -s parameter:

`ndi_decode`` -s ``"NDI Device (Chan 1)"`

NOTE: The default boot files on the uSD image must be switched to the NDI Decode version prior to running the ndi\_decode application. See the "Available Examples" section for your specific board for details.

#### Known issues and limitations

* "Scaling" is performed by manipulating line stride when the NDI stream and video output resolutions do not match
* Format conversion is not performed between 4:2:2 and 4:2:0 video sources. If the output video format does not match the NDI stream format, a warning is displayed on the Linux console.

### Board Details

Connections and settings required to boot using the uSD images. Refer to the vendor documentation for full details.

#### Altera Arria-10 SoC Development Kit

**Available Examples**

Two FPGA examples are available for the ZCU104 board:

* Decode example with 4 NDI Decode cores
* Encode example with 4 NDI Encode cores

The default uSD image uses the Enc files, however files for all examples are included on the uSD card. To change the example design, simply update the FPGA programming files in the `/boot` directory and **cycle power**:

`# Use the Decode example`\
`sudo`` cp /boot/fit_spl_fpga_dec.itb /boot/fit_spl_fpga.itb`\
\
`# Use the Encode example`\
`sudo`` cp /boot/fit_spl_fpga_enc.itb /boot/fit_spl_fpga.itb`\
\
`# You MUST cycle power to reprogram the FPGA!!!`

**Jumpers and Switches**

All jumpers and switches are set per the defaults listed in section 3 of the Arria 10 SoC Development Kit User Guide.

\+------------+---------------------------+-----------------------------+ | Ref | Function | Setting | +============+===========================+=============================+ | SW1 | Boot Mode | All off | +------------+---------------------------+-----------------------------+ | SW2 | User Switches | All off | +------------+---------------------------+-----------------------------+ | SW3.1 | A10 JTAG | Off | +------------+---------------------------+-----------------------------+ | SW3.2 | Max V JTAG | Off | +------------+---------------------------+-----------------------------+ | SW3.3 | FMCA JTAG | On | +------------+---------------------------+-----------------------------+ | SW3.4 | FMCB JTAG | On | +------------+---------------------------+-----------------------------+ | SW3.5 | PCIe JTAG | On | +------------+---------------------------+-----------------------------+ | SW3.6 | MSTR0 JTAG | Off | +------------+---------------------------+-----------------------------+ | SW3.7 | MSTR1 JTAG | Off | +------------+---------------------------+-----------------------------+ | SW3.8 | MSTR2 JTAG | Off | +------------+---------------------------+-----------------------------+ | SW4 | MSEL | All off | +------------+---------------------------+-----------------------------+ | J16 | OSC2 Clk Sel | Short | +------------+---------------------------+-----------------------------+ | J17 | OSC2 Clk Sel | Short | +------------+---------------------------+-----------------------------+ | J30 | HPS Core Voltage | Short | +------------+---------------------------+-----------------------------+ | J32 | FMCBVADJ | Short 9 and 10 | +------------+---------------------------+-----------------------------+ | J42 | FMCAVADJ | Short 9 and 10 | +------------+---------------------------+-----------------------------+ | J3 | BSEL0 | Short left 2 pins | +------------+---------------------------+-----------------------------+ | J4 | BSEL1 | Short upper 2 pins | +------------+---------------------------+-----------------------------+ | J5 | BSEL2 | Short upper 2 pins | +------------+---------------------------+-----------------------------+

**Connections**

\+-----+-------------------+-------------------------------------------+ | Ref | Function / Label | Connect to | +=====+===================+===========================================+ | J19 | FMC Port B | Bitec FMC HDMI Daughter Card Rev. 11 | +-----+-------------------+-------------------------------------------+ | J5 | HPS Ethernet | Local Ethernet network | +-----+-------------------+-------------------------------------------+ | J10 | PROG/UART | Host system USB port | +-----+-------------------+-------------------------------------------+ | J36 | Power | Power Supply Module | +-----+-------------------+-------------------------------------------+ | RX | Bitec HDMI Rx | HDMI video source (NDI Encode) | +-----+-------------------+-------------------------------------------+ | TX | Bitec HDMI Tx | HDMI video output (NDI Decode) | +-----+-------------------+-------------------------------------------+

**LEDs**

\+--------------------------+-------------------------------------------+ | LED | Function | +==========================+===========================================+ | FPGA\_LED0 | Tally (Main or Preview) | +--------------------------+-------------------------------------------+ | FPGA\_LED1 | Tally (Main) | +--------------------------+-------------------------------------------+ | FPGA\_LED2 | Tally (Preview) | +--------------------------+-------------------------------------------+ | FPGA\_LED3 | CPU load | +--------------------------+-------------------------------------------+

#### Digilent Zybo-Z7-20

**Available Examples**

Three FPGA examples are available for the Zybo-Z7 board:

* Decode.Xil: 4-core NDI Decoder with 1 GB of 32-bit SDRAM
* Encode.Xil: 4-core NDI Encoder with 1 GB of 32-bit SDRAM
* Enc-Lite.Xil: 2-core NDI Encoder with 512 MB of 16-bit SDRAM (1 SDRAM chip is unused)

The default uSD image uses the 4-core Encoder files, however files for all three examples are included on the uSD card. To change the example design, simply update the files in the `/boot` directory and reboot:

`# Use the Decode example with 4 cores and 1 GB of 32-bit SDRAM`\
`sudo`` cp /boot/Decode.Xil/* /boot/`\
\
`# Use the Encode example with 4 cores and 1 GB of 32-bit SDRAM`\
`sudo`` cp /boot/Encode.Xil/* /boot/`\
\
`# Use the "Lite" Encode example with 2 cores and 512 MB of 16-bit SDRAM`\
`sudo`` cp /boot/Enc-Lite.Xil/* /boot/`

The "Enc-Lite" Encode example is intended to allow performance evaluation of a typical "minimal" Zynq platform that uses only a single SDRAM memory chip with a 16-bit memory bus. This design is still capable of processing 1080p60 video.

The "Full" Encode example will have lower latency at all resolutions and can in theory compress up to 4Kp60 4:2:0 video (with enough available SDRAM bandwidth). A source of 4K video would also be required (eg: Analog Devices ADV7619, ITE IT68059) since the HDMI I/O on the Zybo-Z7 does not support resolutions beyond 1080p60.

**Jumpers and Switches**

\+-------------+--------------------------------+----------------------+ | Ref | Function | Setting | +=============+================================+======================+ | JP5 | Boot Mode | SD | +-------------+--------------------------------+----------------------+ | JP6 | Power | WALL | +-------------+--------------------------------+----------------------+

**Connections**

\+------+---------------------+-----------------------------------------+ | Ref | Function / Label | Connect to | +======+=====================+=========================================+ | J9 | HDMI Rx | HDMI video source (NDI Encode) | +------+---------------------+-----------------------------------------+ | J8 | HDMI Tx | HDMI video output (NDI Decode) | +------+---------------------+-----------------------------------------+ | J7 | Line In | Analog audio in | +------+---------------------+-----------------------------------------+ | J5 | Headphone | Analog audio out | +------+---------------------+-----------------------------------------+ | J3 | Ethernet | Local Ethernet network | +------+---------------------+-----------------------------------------+ | J17 | Power | Appropriate 5V power supply | +------+---------------------+-----------------------------------------+ | J12 | PROG/UART | Host system USB port | +------+---------------------+-----------------------------------------+

**LEDs**

\+-----------+----------------------------------------------------------+ | LED | Function | +===========+==========================================================+ | LD0 | Tally (Main) | +-----------+----------------------------------------------------------+ | LD1 | Tally (Preview) | +-----------+----------------------------------------------------------+ | LD2 | CPU load | +-----------+----------------------------------------------------------+ | LD3 | uSD activity | +-----------+----------------------------------------------------------+ | LD4 | Tally (either Main or Preview) | +-----------+----------------------------------------------------------+

#### Digilent Arty-Z7-20

**Available Examples**

One FPGA example is available for the Arty-Z7 board:

* Arty-Enc.Xil: NDI Encoder based on Zybo "Lite" design

The default uSD image uses the Zybo boot files, however the Arty boot files are included. To modify the uSD image to boot on the Arty, simply update the files in the `/boot` directory before booting the Arty board:

`# Use the Arty version of the "Lite" Encode example`\
`sudo`` cp /boot/Arty-Enc.Xil/* /boot/`

**Jumpers and Switches**

\+--------------+-------------------------------+-----------------------+ | Ref | Function | Setting | +==============+===============================+=======================+ | JP4 | Boot Mode | SD | +--------------+-------------------------------+-----------------------+ | JP5 | Power | USB | +--------------+-------------------------------+-----------------------+

**Connections**

\+------+---------------------+-----------------------------------------+ | Ref | Function / Label | Connect to | +======+=====================+=========================================+ | J10 | HDMI Rx | HDMI video source (NDI Encode) | +------+---------------------+-----------------------------------------+ | J11 | HDMI Tx | HDMI video output (NDI Decode) | +------+---------------------+-----------------------------------------+ | J8 | Ethernet | Local Ethernet network | +------+---------------------+-----------------------------------------+ | J14 | PROG/UART | Host system USB port | +------+---------------------+-----------------------------------------+

**LEDs**

\+------------------+---------------------------------------------------+ | LED | Function | +==================+===================================================+ | LD0 | Tally (Main) | +------------------+---------------------------------------------------+ | LD1 | Tally (Preview) | +------------------+---------------------------------------------------+ | LD2 | CPU load | +------------------+---------------------------------------------------+ | LD3 | uSD activity | +------------------+---------------------------------------------------+

#### Xilinx ZCU104

**Available Examples**

Two FPGA examples are available for the ZCU104 board:

* Dec: Decode example with 4 NDI Decode cores
* Enc: Encode example with 4 NDI Encode cores

The default uSD image uses the Enc files, however files for all examples are included on the uSD card. To change the example design, simply update the files in the `/boot` directory and reboot:

`# Use the Decode example`\
`sudo`` cp /boot/Dec/* /boot/`\
\
`# Use the Encode example`\
`sudo`` cp /boot/Enc/* /boot/`

**Jumpers and Switches**

\+--------------+-------------------------+-----------------------------+ | Ref | Function | Setting | +==============+=========================+=============================+ | J85 | POR\_OVERRIDE | 2-3 (default) | +--------------+-------------------------+-----------------------------+ | J12 | SYSMON I2C addr | 1-2 (default) | +--------------+-------------------------+-----------------------------+ | J13 | SYSMON I2C addr | 1-2 (default) | +--------------+-------------------------+-----------------------------+ | J20 | PS\_POR\_B | 1-2 (default) | +--------------+-------------------------+-----------------------------+ | J21 | PS\_SRST\_B | 1-2 (default) | +--------------+-------------------------+-----------------------------+ | J22 | Reset Seq | open (default) | +--------------+-------------------------+-----------------------------+ | SW6 1 | PS\_MODE | On | +--------------+-------------------------+-----------------------------+ | SW6 2 | PS\_MODE | Off | +--------------+-------------------------+-----------------------------+ | SW6 3 | PS\_MODE | Off | +--------------+-------------------------+-----------------------------+ | SW6 4 | PS\_MODE | Off | +--------------+-------------------------+-----------------------------+

**Connections**

\+---------+---------------------------+--------------------------------+ | Ref | Function / Label | Connect to | +=========+===========================+================================+ | P7 | HDMI Rx (bottom) | HDMI video source | +---------+---------------------------+--------------------------------+ | J52 | Power | Power supply | +---------+---------------------------+--------------------------------+ | P12 | Ethernet | Ethernet network | +---------+---------------------------+--------------------------------+ | J164 | JTAG/UART | Host system USB port | +---------+---------------------------+--------------------------------+

**LEDs**

\+------------+---------------------------------------------------------+ | LED | Function | +============+=========================================================+ | LED0 | Tally (Main or Preview) | +------------+---------------------------------------------------------+ | LED1 | Tally (Main) | +------------+---------------------------------------------------------+ | LED2 | Tally (Preview) | +------------+---------------------------------------------------------+ | LED3 | CPU load | +------------+---------------------------------------------------------+

#### Terasic SoCKit

The NDI Decode demo uses the on-board VGA output, but also drives an optional [DVI-HSMC](https://www.terasic.com.tw/cgi-bin/page/archive.pl?Language=English\&CategoryNo=68\&No=359) daughter card which is capable of supporting higher resolutions. The DVI-HSMC daughter card will be required for the upcoming NDI Encode examples, as the SoCKit has no on-board video input.

**Available Examples**

Currently, only the NDI Decode example is supported on the SoCKit board. An Encode example will be provided in a later release.

**Jumpers and Switches**

\+------------+---------------------------+-----------------------------+ | Ref | Function | Setting | +============+===========================+=============================+ | SW4.1 | JTAG\_HPS\_EN | Off (default) | +------------+---------------------------+-----------------------------+ | SW4.2 | JTAG\_HSMC\_EN | On (default) | +------------+---------------------------+-----------------------------+ | SW6.1 | MSEL0 | Off | +------------+---------------------------+-----------------------------+ | SW6.2 | MSEL1 | On | +------------+---------------------------+-----------------------------+ | SW6.3 | MSEL2 | On | +------------+---------------------------+-----------------------------+ | SW6.4 | MSEL3 | On | +------------+---------------------------+-----------------------------+ | SW6.5 | MSEL4 | On | +------------+---------------------------+-----------------------------+ | SW6.6 | N/A | On | +------------+---------------------------+-----------------------------+ | J17 | BOOTSEL0 | 1-2 (default) | +------------+---------------------------+-----------------------------+ | J19 | BOOTSEL1 | 2-3 (default) | +------------+---------------------------+-----------------------------+ | J18 | BOOTSEL2 | 1-2 (default) | +------------+---------------------------+-----------------------------+ | J15 | CLKSEL0 | 2-3 (default) | +------------+---------------------------+-----------------------------+ | J16 | CLKSEL1 | 2-3 (default) | +------------+---------------------------+-----------------------------+ | JP2 | HSMC VCCIO | 5-6 (default) | +------------+---------------------------+-----------------------------+

**Connections**

\+--------+---------------------------+---------------------------------+ | Ref | Function / Label | Connect to | +========+===========================+=================================+ | J6 | Line In (Blue) | Analog audio in | +--------+---------------------------+---------------------------------+ | J7 | Line Out (Green) | Analog audio out | +--------+---------------------------+---------------------------------+ | J10 | VGA Connector | Analog video out | +--------+---------------------------+---------------------------------+ | J14 | HSMC Connector | HSMC-DVI (optional) | +--------+---------------------------+---------------------------------+

[^1]: As noted in the AAC support section of this document, this would almost always be two bytes.
