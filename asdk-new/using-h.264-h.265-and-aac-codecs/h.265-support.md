# H.265 Support

## Supported Formats

NDI assumes that all H.265 data is as specified in Annex B of ITU-T H.265 | ISO/IEC 23008-2. The data must include the start codes. We support 4:2:0 in Main, Main Still Picture, and Main10 profiles in resolutions up to 4096x2304 pixels if it is supported by decoding hardware. We recommend that an I-Frame interval of between one and two seconds; it is however very important that you recall that the SDK will inform you when I-Frame insertions are required. The recommended bitrate of the stream is provided by `NDIlib_send_get_target_bit_rate`; for HX it is considered reasonable that you allow the user to select whether you wish a bit rate in the range of 0.1 -- 2.0x the bitrate provided by this function.

The recommended rules for the H.265 frame types are the same as those for the H.264 frame types described in the above section.

## Extra Data

The extra ancillary data required to configure an H.265 codec must correctly be provided. This must exist for all I-frames. The structure of this data should contain concatenated NAL units in Annex B format, along with their start codes. For H.265, they are the VPS, SPS, and PPS NAL units.

## PTS and DTS

The PTS and DTS are not used directly by the NDI SDK, however they are important to ensure that frames may be decoded and displayed in correct order. While we recommend that you use 100 ns intervals for these values, any time-base is technically supported if the ordering of integers is correct.
