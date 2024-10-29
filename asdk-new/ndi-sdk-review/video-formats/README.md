# Video Formats

Decoding is more complex than encoding, because you do not get to specify what format you are sent. It is recommended that you support as many possible video formats as you wish, although if this is not possible (e.g., non-video resolutions) you can either scale, or simply provide a place-holder image. It is important that you support the three possible video formats (4:2:0, 4:2:2, 4:2:2:4) since these are in common use. If you cannot process the alpha channel, it is recommended that you multiply the image against black.

It is also important to understand that it is the NDI sender that determines the video and audio clock rates. A simple framebuffer is not sufficient to smoothly display audio and video without glitches.
