# HDR

NDI SDK v6 supports 16-bit data formats and standardizes metadata elements to specify wide gamut color primaries and HDR transfer functions, including HLG and PQ. This enables broadcasting high bit depth, wide color gamut, and high dynamic range video, with brightness up to 10000 nits when using the PQ transfer function.

{% hint style="info" %}
For in-depth information about HDR production, please consult the [**report ITU-R BT.2408**](https://www.itu.int/pub/R-REP-BT.2408), "Guidance for operational practices in HDR television production".
{% endhint %}

The specific HDR capabilities available to your system will vary depending on your license and the SDK you use, as described next.

***

### Differences between the NDI SDK and the NDI Advanced SDK

Systems licensed to integrate the NDI Advanced SDK:

* Can receive and process HDR content
* Can send HDR content

{% hint style="warning" %}
When using the **trial** version of the NDI Advanced SDK, sending HDR content is limited to 30 minutes (the stream will stop automatically afterward).
{% endhint %}

Systems that integrate the NDI SDK:

\- Can receive and process HDR content

\- Cannot send content with HDR metadata\


{% hint style="danger" %}
Any systems integrating **NDI 5** **or older** versions of both the NDI SDK and the NDI Advanced SDK **cannot** send or receive HDR content (in this case, a placeholder frame will be displayed).
{% endhint %}

### HDR Metadata

Video frames can be annotated with HDR metadata by writing an XML string into the `p_metadata` member of the `NDIlib_video_frame_v2_t` struct.

```json
// // HDR require metadata information in video frame
// note 'transfer' have two possible values for HDR: bt_2100_pq or bt_2100_hlg
NDI_video_frame_16bit.p_metadata =
    "<ndi_color_info "
    "   transfer=\"bt_2100_hlg\" "
    "   matrix=\"bt_2020\" "
    "   primaries=\"bt_2020\" "
    "/> ";

```

The receiver can use this metadata to interpret the received frames' content correctly. The NDI SDK will not alter the content of the frame in any way.

The HDR metadata format consists of an `ndi_color_info` XML element containing the following three attributes:

* **"primaries" -** specifies the chromaticity coordinates of the color primaries of the video frame.
* **"transfer"** - specifies the brightness transfer characteristic.
* **"matrix"** - Specifies the matrix coefficients used in deriving luma and chroma from RGB or XYZ primaries.

The following values are defined for these attributes:

<table data-full-width="true"><thead><tr><th>"primaries"</th><th>"transfer"</th><th>“matrix”</th></tr></thead><tbody><tr><td>“bt_601”</td><td> “bt_601”</td><td>“bt_601”</td></tr><tr><td>“bt_709”</td><td>“bt_709”</td><td>“bt_709”</td></tr><tr><td>“bt_2020”</td><td>“bt_2020”</td><td>“bt_2020”</td></tr><tr><td>“bt_2100”</td><td>“bt_2100_hlg”</td><td>“bt_2100”</td></tr><tr><td></td><td>“bt_2100_pq”</td><td></td></tr></tbody></table>

{% hint style="info" %}
For a detailed description of these video format properties please consult [**ITU-T H.273**](https://www.itu.int/rec/T-REC-H.273).
{% endhint %}

Example:

{% code overflow="wrap" %}
```json
<ndi_color_info primaries="bt_2020" transfer="bt_2100_hlg" matrix="bt_2020" />
```
{% endcode %}

If multiple XML elements must be added to the same frame, use an `ndi_metadata_group` XML element as root node.

Example:

```json
<ndi_metadata_group>
    <ndi_color_info primaries="bt_2020" transfer="bt_2100_hlg" matrix="bt_2020" />
</ndi_metadata_group>
```

{% hint style="info" %}
For in-depth information about the individual options please consult [**ITU-T Rec. H.273**](https://www.itu.int/rec/T-REC-H.273), "Coding-independent code points for video signal type identification".
{% endhint %}

### Sending Compressed HDR Frames

Sending compressed 10-bit or 12-bit H.264 and HEVC frames is much like non-HDR frames, but with the addition of frame annotations supplying HDR metadata as described above.

### Sending Uncompressed HDR Frames

HDR transfer functions are defined in ITU BT.2100 and specify 10 and 12- bit integer representations for both PQ and HLG variants.

To send uncompressed 10-bit and 12-bit HDR frames via SHQ streams, first convert the video data to 16-bit, limited range, and write it into a P216 buffer (or a PA16 buffer if an alpha channel is required).

For limited range 10-bit and 12-bit HDR formats, simply padding the lower bits with zeroes is sufficient to result in a valid 16-bit limited range value.

Please note that full-range signals are not supported by NDI Tools at this point. To convert full-range signals to 16-bit limited range signals, normalize the full-range signal to the 0.0...1.0 floating point range, and convert to limited range 16-bit integers using the quantization equations specified in the ITU BT.2100 recommendation.

### Receiving Compressed HDR Frames

The `p_metadata` member of the `NDIlib_video_frame_v2_t` data structure may contain the HDR metadata described above. (Please note that the `ndi_color_info` may be nested under a `ndi_metadata_group` XML root element.)

NDI receivers receiving HDR frames using NDI SDK versions older than v6 will send a placeholder image showing the incompatibility of the stream.

### Receiving Uncompressed HDR Frames

The NDI SDK can uncompress high-bit depth frames to P216 or PA16 buffer formats if required.

When a high bit depth stream is supplied, configure the NDI receiver to use the "best" color format to receive high bit depth frames:

{% code overflow="wrap" %}
```json
NDIlib_recv_create_v3_t NDI_recv_create_desc;
NDI_recv_create_desc.source_to_connect_to = *p_hdr_source;
NDI_recv_create_desc.color_format = NDIlib_recv_color_format_best;
 
// Create the receiver
NDIlib_recv_instance_t pNDI_recv = NDIlib_recv_create_v3(&NDI_recv_create_desc);
If the incoming NDI stream is high bit depth then the video frame received via NDIlib_recv_capture_v2 will have a fourCC of either NDIlib_FourCC_video_type_P216 or NDIlib_FourCC_video_type_PA16.
```
{% endcode %}

{% hint style="warning" %}
Note that if the video frame is requested in 8-bit format, the NDI SDK does not attempt down-mapping from HDR to SDR.
{% endhint %}
