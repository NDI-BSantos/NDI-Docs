# Sending Audio Frames

To submit audio frames, start by building a structure of type `NDIlib_compressed_packet_t` that has the compressed AAC audio data within it.

The following example creates a compressed audio frame when you have compressed audio data of size `audio_data_size`, at pointer `p_audio_data` with [`audio_extra_data_size`](#user-content-fn-1)[^1] at pointer `p_audio_extra_data`:

```
// See notes above
uint8_t* p_audio_data; 
uint32_t audio_data_size;

// See notes above
uint8_t* p_audio_extra_data;
uint32_t audio_extra_data_size;

// Compute the total size of the structure
uint32_t packet_size = sizeof(NDIlib_compressed_packet_t) + audio_data_size +
                                                            audio_extra_data_size;

// Allocate the structure
NDIlib_compressed_packet_t* p_packet = (NDIlib_compressed_packet_t*)malloc(packet_size);

// Fill in the settings
p_packet->version = NDIlib_compressed_packet_t::version_0;
p_packet->fourCC = NDIlib_FourCC_type_AAC;
p_packet->pts = 0; // These should be filled in correctly if possible.
p_packet->dts = 0;
```

As noted in the <mark style="color:red;">AAC</mark> support section of this document, this would almost always be two bytes.

```
p_packet->flags = NDIlib_compressed_packet_t::flags_keyframe; // All AAC packets are a keyframe
p_packet->data_size = audio_data_size;
p_packet->extra_data_size = audio_extra_data_size;

// Compute the pointer to the compressed audio data, then copy the memory into place.
uint8_t* p_dst_audio_data = (uint8_t*)(1 + p_packet);
memcpy(p_dst_audio_data, p_audio_data, audio_data_size);

// Compute the pointer to the ancillary extra data
uint8_t* p_dst_extra_audio_data = p_dst_audio_data + audio_data_size;
memcpy(p_dst_extra_audio_data, p_audio_extra_data, audio_extra_data_size);
```

Once you have the compressed data structure that describes the frames, then you need simply create a regular `NDIlib_audio_frame_v3_t` to pass to NDI SDK functions as shown in the following example:

```
// Create a regular NDI audio frame, but of compressed format
NDIlib_audio_frame_v3_t audio_frame;
audio_frame.sample_rate = 48000;		// This must match AAC format
audio_frame.no_channels = 2;			// This must match AAC format
audio_frame.no_samples = 1024;			// This must match AAC format
audio_frame.timecode = p_packet->pts;		// Might be a good value. Read docs!
audio_frame.FourCC = (NDIlib_FourCC_audio_type_e)NDIlib_FourCC_audio_type_ex_AAC;
audio_frame.p_data = (uint8_t*)p_packet;
audio_frame.data_size_in_bytes = packet_size;
audio_frame.p_metadata = "<something/>";	// Per frame metadata is fully supported.

// Transmit the AAC audio data to NDI
NDIlib_send_send_audio_v3(pSender, &audio_frame);
```

Once this is sent the audio data will be transmitted, and you may free or re-use any audio data pointers that you allocated to represent the data. It is obviously possible to use a pool of memory to build audio packets without per-packet memory allocations.

The send audio function is thread-safe and may be called on a separate thread from compression transmission.

[^1]: As noted in the AAC support section of this document, this would almost always be two bytes.
