# Simple Setup of Broadcast and Receivers

There are a number of ways to add NDI broadcast of views of the level, and receiving of NDI streams into the level. The easiest way is to add an [NDI Broadcast Actor](ndi-broadcast-actor.md) and [NDI Receive Actor](ndi-receiver-actor.md) to the level.

### Simple Broadcast Actor

In `Place Actors` search for `NDI Broadcast Actor`. Drag and drop it into the level scene. A camera will appear on the level, and an `NDIBroadcastActor` is added to the `World Outliner`. The broadcast actor needs an`NDI Media Source` asset that contains details of the stream, such as the stream name and frame size.&#x20;

Select the broadcast actor in the `World Outliner,` and look at its `Details`. In the **NDI IO** section, there is a reference to the NDI Media Source. Initially, it is set to “None”. Use the dropdown menu next to it to create a new `NDI Media Sender`asset.&#x20;

Play the level, and the broadcast actor will begin streaming its view. By default, the stream's name is **`Unreal Engine Output`**, and it is displayed at a resolution of 1920 x 1080 at 60fps. Open, for example, [Studio Monitor](https://app.gitbook.com/s/RNgRFpW0QELCFWgOTQfu/ndi-tools-for-windows/studio-monitor) to view the stream.&#x20;

The details of the stream can be changed in the settings for the NDI Media Sender asset

### Simple Receiver Actor

Setting up a simple **NDI stream receiver actor** is similar to setting up a broadcast actor. From `Place Actors,`drag and drop an `NDI Receive Actor` into the level. It will be shown on the level as a black screen, and an `NDIReceiveActor` will appear in the `World Outliner`.&#x20;

The receiver actor must be associated with an `NDI Media Receiver` asset containing the details of the stream to receive. Click on the added `NDIReceiveActor` in the `World Outliner`, and in the `Details` look for the `NDI IO` section. In the `Media` sub-section there is a reference to an `NDI Media Source` asset. Use the dropdown menu next to it to create a new `NDI Media Receiver` asset.&#x20;

Open the settings for the new `NDI Media Receiver` asset and fill in the source's name in the `Source Name` field. Play the level, and after a few seconds, the stream should appear on the receiver actor.

{% hint style="warning" %}
Make sure the actor is in view.
{% endhint %}

### Active Viewport Broadcasting

The plugin has the ability to broadcast the active viewport. The settings for this are on the **NDI page** of the **Plugins section of the Project Settings**. By default the broadcast of the active viewport is started through a blueprint function. It is also possible to automatically start the broadcast by enabling `Begin Broadcast On Play` in the plugin's settings.
