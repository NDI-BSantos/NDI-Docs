# NDI Broadcast Actor

A broadcast actor creates an NDI stream of the view, as captured by the actor. It is represented in the level by a camera and is independent of the viewport. Multiple broadcast actors can be used, each with its own stream, to stream multiple views of the scene at the same time.&#x20;

By default, a newly created broadcast actor does not have a media source set. The media source contains information about the stream, such as its name and resolution. The media source for a broadcast actor is an `NDI Media Sender` asset type. When setting up a broadcast actor, a new NDI Media Sender asset can be created and set, or an existing one can be selected.&#x20;

The broadcast actor also comes with two components attached: the `ViewportCaptureComponent` containing the setting of how to capture the view, and a `PTZController` component that can be used to control the camera as a virtual PTZ device.

### NDI Viewport Capture Component

The viewport capture component is responsible for capturing the view. It is derived from the standard `USceneCaptureComponent2D`, and has many settings to control how the view is captured. We shall go over a few that are particularly relevant for NDI use.

#### Capture Settings

When overriding the broadcast settings, the view will be rendered at the given size, but scaled to fit the stream's resolution (as set in the actor's media source).&#x20;

If the stream outputs alpha values (the actor's media source has Output Alpha turned on), then Alpha Remap can be used to control how the alpha values in the view are translated to alpha values in the NDI stream. The min value is the Unreal Engine rendered alpha value mapped to an NDI stream alpha value of `0`, while the max is the value mapped to a stream alpha of `1`.&#x20;

All other alpha values are linearly interpolated from those. The min value can be set larger than the max value. The alpha remap values specified in the viewport capture component will overwrite the alpha remap values in the actor's NDI Media Sender asset.

#### Scene Capture

The **Texture Target** is optional. If not set (i.e. it is set to `None`), a default transient texture target will be created internally. It should suffice for most uses.&#x20;

The **Capture Source** sets how to capture the view. Unreal Engine renders views in a number of passes and can apply various post-processing steps for various effects. The capture source determines during which pass the view is captured. Depending on the capture source, some information may be unavailable. For example, in **Final Color modes,** alpha values are not available. While in **Scene Color modes,** post-processing effects such as depth of field is not present.&#x20;

When using a capture source with alpha (whether obtained from opacity or scene depth), the broadcast actor's media source should have **Output Alpha** turned on for NDI to stream out an alpha channel. Care should also be taken when setting the **Alpha Remap Min and Max values** in the `Capture Settings` section. Notice that some **Capture Source** modes produce an inverted alpha channel. For those, Alpha Remap Min should be set to `1` and Alpha Remap Max to `0` to undo the inversion.&#x20;

When using a **Capture Source** mode with scene depth in the alpha channel, set Alpha Remap Max to the largest distance you want to be represented as `1` in the stream's alpha channel. The Alpha Remap Min can be set to `0` to capture the entire depth range. If you want to use the scene depth as an alpha mask, remember that an alpha value of `0` is fully transparent (and `1` fully opaque). In such cases set the Alpha Remap Min to the maximum depth (far away is transparent), and Alpha Remap Max to `0` (near is opaque). A hard distance cut-off can be achieved by setting Alpha Remap Max to slightly less than the Min.

### NDI PTZ Controller

The **PTZ Controller** component allows the virtual camera to be oriented, zoomed, and focused through the NDI PTZ interface. PTZ control commands are received from the associated broadcast actor's media sender asset and applied to the viewport capture component.&#x20;

**PTZ control is enabled by default. The toggle in the PTZ Controller settings can disable or enable it for an actor**. There is also an Enable PTZ toggle in the NDI media sender, which can disable PTZ control for all actors using that sender.&#x20;

Minimum and maximum limits can be enabled and set for the pan, tilt, and field of view (zoom) if the minimum and maximum values are the same, which effectively locks that movement. The directions in which pan and tilt move can also be inverted.&#x20;

For focus control to have a visible effect, the Capture Source for the viewport capture component should be set to a **Final Color** mode (the application of depth of field is a post-process). Enabling auto-focus sets the depth of field to zero, which Unreal Engine interprets as the depth of field being disabled.&#x20;

The PTZ controller can have several **Presets**. Keep in mind that PTZ controls such as those exposed through the NDI® Studio Monitor application are limited to nine presets. Presets can be pre-defined or set and modified by stream receivers through PTZ controls. &#x20;

When switching between presets, the PTZ controller can be set to smoothly move the camera **between the current state and other presets**. The **Preset Recall Easing** determines how many seconds it takes to complete the transition. If set to zero, the switch is done instantaneously. When used with an NDI Broadcast Actor, the PTZ Controller usually uses the actor's media sender asset to get PTZ commands. This can be overridden by explicitly specifying an NDI Media Sender asset in the PTZ Controller.

#### NDI PTZ Control on other actors

The NDI PTZ Controller component can be attached to actors other than the NDI Broadcast Actor. When doing so, a media sender asset should be specified in the PTZ component. Otherwise, the controller will not know where to receive PTZ commands. Only pan and tilt transformations will be applied to the actor. Zoom and focus will be ignored.
