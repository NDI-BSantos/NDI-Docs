# H.264 Support

## Supported Formats

NDI assumes that all H.264 data is as specified in Annex B of ITU-T H.264 | ISO/IEC 14496-10. The data must include the start codes. We support 4:2:0 in Baseline, Main, and High profiles up to level 5.1. We recommend that an I-Frame interval of between one and two seconds; it is however very important that you recall that the SDK will inform you when I-Frame insertions are required. The recommended bitrate of the stream is provided by `NDIlib_send_get_target_bit_rate`; for HX it is considered reasonable that you allow the user to select whether you wish a bit rate in the range of 0.1 -- 2.0x the bitrate provided by this function.

The current NDI implementation supports H.264 decoding without any installed plugins across the Windows and macOS targets. If you require Linux support, please contact NDI SDK support.

Full hardware acceleration is provided on these platforms if the relevant hardware acceleration recommendations are provided by an end user application as described in the NDI Advanced SDK documentation. On Windows 7, the maximum decoder resolution is 1920x1080 due to OS limitations. On other platforms the maximum resolution is currently 4096x2304.

While you may use all H.264 frame types, we recommend considering the use of B frames with caution, since these will cause increased decoder latency in some configurations. Because NDI is a low latency, real-time scheme, it is crucial that you focus your great effort on achieving low latency and high quality.

It is crucial that your codec provide reasonable accurately-timed frames; a common problem that we have observed is that -- with some bitrate control algorithms -- that the I-Frames are sufficiently large that subsequent P frames are delayed far more than the Nyquist sampling limit, which tends to cause jittery video transmission.

## Extra Data

The extra ancillary data required to configure an H.264 codec must correctly be provided. This must exist for all I-frames. The structure of this data should contain concatenated NAL units in Annex B format, along with their start codes. For H.264, they are the SPS and PPS NAL units.

## PTS and DTS

The PTS and DTS are not used directly by the NDI SDK, however they are important to ensure that frames may be decoded and displayed in correct order. While we recommend that you use 100 ns intervals for these values, any time-base is technically supported if the ordering of integers is correct.
