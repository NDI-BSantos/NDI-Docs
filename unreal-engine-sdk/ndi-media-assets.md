# NDI Media Assets

NDI streams are represented as assets in the [Unreal Engine](https://www.unrealengine.com/). This allows the same stream to be used by multiple [NDI Receiver Actors](ndi-receiver-actor.md). Furthermore, if a sending stream were to be a component or actor instead, problems could occur due to Unreal Engine creating multiple copies during gameplay.

### NDI Media Sender Asset

A media sender is an NDI stream going out of the Unreal Engine. An NDI Broadcast Actor typically uses it to send out rendered views. Only one broadcast actor should be using a given NDI Media Sender asset.

#### Broadcast Settings

The `Source Name` is the name of the stream. It should be unique to avoid confusion.&#x20;

The `Frame Size` is the desired width and height in pixels of the stream. If the stream is supplied by (for example) an [NDI Broadcast Actor](ndi-broadcast-actor.md) with frames of a size different than this, the frames will be scaled (preserving aspect ratio) to fit the specified size.&#x20;

The `Frame Rate` specifies the desired framerate of the stream. Due to the nature of real-time rendering with Unreal Engine, it is not guaranteed that frames will be rendered and broadcast at this rate. Generally, it should be set to no more than what Unreal Engine can manage for the scene.&#x20;

The stream can optionally include an alpha channel. If the stream outputs alpha values, then **Alpha Remap** can control how the alpha values in the Unreal Engine-generated frame are translated to alpha values in the NDI stream. The min value is the Unreal Engine rendered alpha value mapped to an NDI stream alpha value of `0`, while the max is the value mapped to a stream alpha of `1`. All other alpha values are linearly interpolated from those. The min value can be set larger than the max value.&#x20;

PTZ control through the stream can be enabled and disabled for the NDI Media Sender Asset. This has a slightly different effect than enabling or disabling the [NDI PTZ Controller](ndi-broadcast-actor.md#ndi-ptz-controller) on an [NDI Broadcast Actor](ndi-broadcast-actor.md). The toggle in the NDI Media Asset tells the NDI receivers of the stream whether or not PTZ is enabled (which in turn, for example, will cause [Studio Monitor](https://app.gitbook.com/s/RNgRFpW0QELCFWgOTQfu/ndi-tools-for-windows/studio-monitor) to display PTZ controls). The toggle in the broadcast actor PTZ controller component determines if the actor should use the PTZ commands.

#### Content

The **Render Target** contains the texture that will be encoded and sent over NDI. Typically it will be provided by the viewport capture component of the N[DI Broadcast Actor](ndi-broadcast-actor.md) using this sender, so there is no need to set it manually.

### NDI Media Receiver Asset

A media receiver asset is a stream coming into Unreal Engine. It is used by [NDI Media Receiver Actors](ndi-receiver-actor.md) to receive decoded frames from an NDI stream. Multiple receiver actors can use the receiver asset.

#### Settings

The connection information specifies how to connect to an NDI stream. The stream's name can be specified either as a Source Name (using the “Machine Name (Stream Name)” format) or as an NDI URL.&#x20;

The Bandwidth controls what components of the stream the receiver requests. Audio and video can be (temporarily) muted.&#x20;

When muted, the data is still received but not processed by the receiver asset.

#### Content

The `Video Texture` is the texture to which a received frame is decoded. It is optional. If not set, then an internally generated video texture is used.

### NDI Media Receiver and Media IO Framework

NDI Media Receivers can be used with the Unreal Engine Media IO Framework. They can be added to **Media Bundles**, and the Media Player can play NDI® streams.&#x20;

The URL for playing an NDI stream in the Media Player looks like this: `ndiio://MachineName (StreamName)`&#x20;

Existing NDI Media Receiver assets can also be used in the **Media Player** and added to playlists.&#x20;

NDI Media Receivers can also be created in **Media Bundles**, which can then be used to create media bundle actors in the level. This provides an alternative to adding NDI Receiver Actors directly to the level.

### NDI Timecode Provider

The **NDI IO Plugin** contains a **timecode** provider class, which can present the timecodes of an NDI stream to Unreal. The **NDI Timecode Provider** uses an NDI Media Receiver asset as the source of the timecodes. In order to work, the receiver's video stream must contain timecodes. NDI does not guarantee that all video streams will have valid timecode values. The timecodes produced by the provider are those of the most recently obtained frame.
