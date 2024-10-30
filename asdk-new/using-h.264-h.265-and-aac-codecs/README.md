# Using H.264, H.265, and AAC Codecs

We recommend that you start by reviewing the sending of frames over the network using the NDI® SDK. The Advanced SDK is designed to operate almost identically, although you can send compressed data streams directly.

Currently the Advanced SDK supports H.264 or H.265 compression at a very wide variety of bitrates, resolutions, and framerates; and it would be common for you to use hardware-assisted compression to generate the compressed video stream on Advanced SDK devices, then send this onto the network using the NDI Advanced SDK. AAC audio is supported for audio transmission.

To send a compressed video frame, you should use the Advanced SDK structure NDIlib\_compressed\_packet\_t to pack your data. You should allocate memory for your packet so that the following data will be in a single block:

<table><thead><tr><th width="224">Size and Type</th><th width="163">Name</th><th>Details</th></tr></thead><tbody><tr><td><strong>uint32_t, 4 bytes</strong></td><td>version</td><td>This represents the current version number of the structure.  This should be set to <code>NDIlib_compressed_packet_t::version_0</code>, which has a value of 44 currently (representing the structure size).</td></tr><tr><td><strong>uint32_t, 4 bytes</strong></td><td>fourCC</td><td><p>This is the FourCC for the current compression format.  Currently H.264 is supported, although other formats might be available in the future.</p><p> </p><ul><li>H.264 should be specified using <code>NDIlib_FourCC_type_H264</code>.</li></ul><ul><li>H.265 should be specified using <code>NDIlib_FourCC_type_HEVC</code>.</li></ul><ul><li>AAC audio should be submitted using <code>NDIlib_FourCC_type_AAC</code>.</li></ul></td></tr><tr><td><strong>int64_t, 8 bytes</strong></td><td>pts</td><td>The stream presentation time stamp. See notes in the next section.</td></tr><tr><td><strong>int64_t, 8 bytes</strong></td><td>dts</td><td>The stream display time stamp. See notes in the next section.</td></tr><tr><td><strong>int64_t, 8 bytes</strong></td><td>reserved</td><td>This is currently a reserved field and will not be propagated by the SDK.</td></tr><tr><td><strong>uint32_t, 4 bytes</strong></td><td>flags</td><td><p>The flags that apply to this frame.  Currently there are two supported values for this setting:</p><p> </p><ul><li><code>NDIlib_compressed_packet_t::flags_none</code>. Nothing.</li></ul><ul><li><code>NDIlib_compressed_packet_t::flags_keyframe</code>. This is a frame that can be decoded without dependence on other stream data. This normally means that this is considered an I-Frame.  For H.264 and H.265, key-frames must have extra data. This should always be set for AAC audio.</li></ul></td></tr><tr><td><strong>uint32_t, 4 bytes</strong></td><td>data_size</td><td>The size of the compressed video frame.</td></tr><tr><td><strong>uint32_t, 4 bytes</strong></td><td>extra_data_size</td><td>The size of the ancillary extra data for the current codec settings. This is required for key-frames in most compressed video formats.</td></tr><tr><td><strong>data_size bytes</strong></td><td>[data]</td><td>This is the compressed video frame data in byte format.</td></tr><tr><td><strong>extra_data_size bytes</strong></td><td>[extra_data]</td><td>This is the compressed ancillary data if <code>extra_data_size</code> is not zero.</td></tr></tbody></table>