# AAC Support

## Supported Formats

AAC audio submitted to the NDI SDK should be in the raw AAC format.

All reasonable AAC bitrates are supported, although it is recommended that you use 192 kbps as a reasonable compromise. Any valid AAC channel count and sample-rate should be supported.

## Extra Data

The extra ancillary data required to configure an AAC codec must correctly be provided. This must exist for all AAC frames. The structure of this data should be the `AudioSpecificConfig` structure followed by `GASpecificConfig` as specified from ISO/IEC 14496-03. In practice, this means the extra-data section should be two bytes for each audio frame that describes the sample rate, channel count, etc. The information set in this structure should correctly match the audio settings header on the same frame.

## Audio Levels

The AAC audio level matches that of the NDI SDK, meaning that a floating-point sine-wave value of 1.0 represents a +4 dBU equivalent audio level.
