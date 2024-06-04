# NDI Receiver Actor

An NDI Receiver Actor acts as a screen on which an NDI stream is played. The stream to be played is specified using an **NDI Media Receiver** asset.&#x20;

By default, the receiver actor does not have a media receiver asset set as a media source. In the **NDI IO section** of the receiver actor's settings, an existing NDI Media Receiver asset can be used, or a new one can be created. Multiple receiver actors can share the same receiver asset.

#### NDI IO Settings

The actor's media source references an **NDI Media Receiver** asset. Multiple NDI Receiver Actors can share the same media receiver asset.&#x20;

The Frame Width and Frame Height scale the display. By default, it is set up to fit a `16:9` aspect ratio. If the frames in an NDI stream have a different aspect ratio, the video will be stretched or squashed (the aspect ratio of the video will not be preserved).&#x20;

Enabling or disabling audio, color, and alpha channels can control which components of the stream to use for this actor. **These settings only affect this actor, not any other actor sharing the same media source**.
