# OPUS Support

Starting in version 5 of the NDI SDK one may transmit Opus audio through NDI. All audio is in the multi-channel Opus format, allowing up to 255 channels of audio within a packet. The FourCC used should be be `NDIlib_FourCC_audio_type_ex_OPUS`,

The properties provided within `NDIlib_audio_frame_v3_t` must reflect all legal values for the Opus codec, meaning that all bitrates, channel counts and valid packet sizes are supported. All audio within NDI uses the floating-point decompression functions, meaning that clipping would not occur if full range audio streams and it is recommended that you use the floating-point versions of the compression. The Opus audio level matches that of the NDI SDK, meaning that a floating-point sine-wave value of 1.0 represents a +4 dBU equivalent audio level.

Unlike other compressed formats, the raw audio data is provided within the `NDIlib_audio_frame_v3_t` structure data files and a `NDIlib_compressed_packet_t` is not used. This distinction is made to reduce the bandwidth required for Opus audio over WAN networks by removing the need for extra PTS, DTS values which are not required for the decompression of the Opus bit-stream.
