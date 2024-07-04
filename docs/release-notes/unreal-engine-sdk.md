---
description: SDK VERSION 3.6 FOR USE WITH UNREAL ENGINE® 5.0, 5.1, 5.2, 5.3, AND 5.4
---

# Unreal Engine SDK

#### **Version 3.6**&#x20;

* Compatibility fixes up to and including Unreal Engine 5.4

#### **Version 3.5**&#x20;

* Compatibility fixes up to and including UE5.3.2&#x20;
* Removed support for UE4. The differences between UE4 and UE5 are becoming too great.&#x20;
* Updated to NDI 6.0.1

#### **Version 3.4**&#x20;

* Compatibility fixes up to and including UE5.3.

#### Version 3.3

* Implemented sending audio out of Unreal over NDI.&#x20;
* Improved receiving audio from NDI into Unreal, including control over the number of channels.&#x20;
* Fixed shader script compatibility with Shader Model 6.&#x20;
* Fixed crashes when resizing the window while using active viewport streaming.&#x20;
* Fixed crash when receiving stream switched from not having an alpha channel to having an alpha channel.&#x20;
* Fixed compatibility issue with UE5.1 when the NDI Media Texture is opened in the editor.&#x20;
* Fixed changing the name of a source in the receiver's properties panel not registering as a change.

#### Version 3.2

* Fixed UE5.0 compatibility for NDI Media Texture2D. Some missing functions were causing errors when attempting to open the texture's properties panel. &#x20;
* Fixed initialization of NDI Media Receiver internal video texture potentially happening on the wrong thread, particularly in shipping builds.&#x20;
* Added missing dependencies to example build script.&#x20;
* Added support for UE5.1.

#### Version 3.1&#x20;

* Changed BGRA to UYVY conversion to work around missing texture readback functionality in the UE5 D3D12 renderer.&#x20;
* Fixed receiver connection editor settings not updating properly when part of the stream name specification is changed. &#x20;
* Updated NDI to latest 5.5.1 release.

#### Version 3.0

* Support UE4.26, UE4.27, and UE5.0. Dropped support for UE4.25 due to too many incompatibilities. Added support for Linux Arm64.&#x20;
* Implemented support for Unreal Media IO Framework. &#x20;
* Alpha channels work again, both receiving and sending.&#x20;
* Added ability to mute audio and/or video on a receiving connection.&#x20;
* Added support for disabling color channels (and keeping alpha).&#x20;
* Viewport capture now uses the more capable scene capture component instead of the cinematic camera. &#x20;
* Added PTZ support. &#x20;
* Added support for sending, receiving, and parsing NDI metadata (including through blueprints). &#x20;
* Added support for controlling UE properties from TriCaster macros (still somewhat experimental). &#x20;
* More blueprint support (including NDI metadata handling, delegates for notifying when an audio/video/metadata frame has been sent or is being received, starting and stopping connections, more access to sender and receiver states). &#x20;
* Added support for receivers to supply timecodes to a UE Timecode Provider. &#x20;
* NDI source can now be selected through a pulldown menu in the connection settings. &#x20;
* Received audio is now mixed to mono instead of dropping channels.
