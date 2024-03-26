# Release Notes

{% hint style="info" %}
**These release notes refer to changes in our complete technology, including the SDKs, NDI Tools, and any other items. Take your time to comb through the documentation and decide what is more relevant for your specific context.**
{% endhint %}

### NDI 5.6.1

#### **SDK**

* Bug fix in NDI genlock, where the genlock connect API call could get blocked.
* Updated the NDI library for Android to use an embedded version of mDNS for compatibility with newer Android OS.
* Fixed a potential crash in the NDI library when destroying the NDI sender instance on Windows.

#### **Updates specific to NDI Advanced**

* Xilinx projects migrated to 2022.1 tools.
* Added reference design for Kria KV260 development board by AMD.
* Buf fix in the clear-text FPGA logic.
* Zynq 7000 HDMI Rx logic improved.
* uSD images migrated to Debian 12 (Bookworm).

#### NDI Tools

* Bug fix in NDI Test Patterns for Windows where the NDI send pattern would appear green when changing the frame rate of imported stills.
* Bug fix in NDI Screen Capture for Windows where options were not correctly restored on system reboot/restart.
* Bug fix in NDI Bridge when displaying calculated bandwidth in the Ul.
* Fixed an issue in NDI DirectShow filter to preserve the aspect ratio.

## NDI 5.6.0

#### **NDI Bridge Enhancements**

* Various minor improvements.
* A new configurable buffer setting improves stability, smooths out network jitter, and reduces stuttering on the incoming remote NDI Bridge sources.
* Updated encryption model with TLS 1.3 (Note: for compatibility, this requires all NDI Bridge nodes to use version 5.6).
* A new option for multi-GPU systems allows transcoding to be assigned to specific hardware.

#### **Other Improvements**

* Windows versions of NDI Tools now restore system tray icons after Windows Explorer restarts.
* Newer PTZ cameras are recognized within NDI Studio Monitor for registration purposes.
* Improved forward error correction in multicast and unicast UDP sending.
* Improved detection of clients that suddenly disappear within the NDI Discovery server.
* Improvements to FPGA reference design. Please refer to the ChangeLog files for more details.
* The NDI Webcam tool will no longer show an NDI logo when connecting to sources. This makes the tool better for people who are using it in production.

### NDI 5.5.4

* Fixed detection of Adobe applications on Windows to install the NDI plugins appropriately.
* Fixed potential crashes when using the NDI Transmit plugin on macOS.
* Fixed previews not rendering in NDI Webcam and NDI Router on a system with dual GPUs.
* Fixed potential hang in NDI video encoder.
* Fixed compatibility issue with NDI Remote while working on AWS.
* Fixed handling of audio devices in NDI Studio Monitor when a device is added or removed.
* Fixed potential hang in NDI Launcher after choosing to run certain NDI applications.
* Added support for longer NDI source names.
* Increased reliability when using NVIDIA GPU for hardware-accelerated decoding.
* Increased reliability of the embedded web server within NDI Studio Monitor.
* Updated URLs for NDI short links.

### NDI 5.5.3

* Improved performance of audio resampling used in NDI frame sync.
* Improved handling of misleading "extra data" within NDI|HX video streams.
* Updated NDI Launcher on macOS to look for 2023 versions of Adobe applications.
* Improved handling of encryption key changes in NDI Bridge.

### NDI 5.5.2

* Bug fix on older versions of Windows 10 where system audio was not captured within NDI Screen Capture.
* Fixed crash when handling SpeedHQ video within the NDI library on Android.
* Bug fix where streams caused reconnections to occur when using RUDP on Android.
* Bug fix for an issue when initializing RUDP streams on platforms where RUDP is not supported.
* Made improvements in mDNS handling on Windows.
* Improve responsiveness when establishing new connections in the NDI receiver.
* Fixed crash in NDI Discovery Server if it was started on a system with no available NICs.
* Fixed potential freeze in Adobe Premiere while attempting to export and the NDI Transmit plugin was active.
* Fixed the “Launch at System Startup” feature in NDI Launcher for non-admin user accounts on Windows.
* Fixed a bug where multiple TAB button clicks caused the NDI Launcher window to become blank.
* Fixed a bug in the NDI recorder when recording an audio stream that was not 48 kHz.
* Fixed a bug where the session settings from the older version of NDI Launcher would not transfer to the newly installed version.

### NDI 5.5.1

* Improvements in startup time for NDI Launcher caused by low internet bandwidth.&#x20;
* Bug fix for NDI tutorial video overlapping caused by low internet bandwidth.
* Fixed potential instability issues when using a newer version of the NDI|HX driver with older versions of NDI software.

## NDI 5.5.0

* Registration and Launcher streamlined. Autorun selected NDI apps on the system start.
* All new matrix Router for NDI streams.
* NDI Remote enhancements, including Talkback.
* Multi-cam support for NDI Webcam.
* NDI Audio Direct (VST plugins for your DAW).

### NDI 5.1.4

* Bug fix in NDI Screen Capture on Windows incorrectly enabling pointer trails for mouse capture.
* Bug fix in NDI Studio Monitor after activation of certain NDI|HX cameras.
* Bug fix for issue when initializing RUDP streams on platforms where RUDP is not supported.
* Bug fix when processing certain PA16 video frames.
* Bug fix in NDI Scan Converter on macOS not capturing and sending video.

### NDI 5.1.3

* We fixed a bug in the NDI SDK that could cause a crash when decoding 10-bit HEVC on an NVIDIA video card.
* Bug fix for NDI Screen Capture on Windows 11 to lead to webcam devices being unusable.
* Bug fix for Premiere plugin where audio channels were mapped incorrectly for NDI output.
* Addressed stability issues in NDI VST output plugin.

### NDI 5.1.2

* Bug fix for NDI Bridge using high bandwidth for H.264 and HEVC streams.
* Bug fix in the NDI SDK where a reported tally change was not notified properly in the NDI sender.&#x20;
* Bug fix where the bandwidth for interlaced video was double the intended bandwidth.
* Bug fix in NDI Bridge for HEVC video not being generated appropriately when running on an Intel-only system.
* Bug fix in Premiere plugin that would lead to unstableness when reconnecting audio devices while the NDI output was enabled.

### NDI 5.1.1

* Fixed a problem in which HX sources might not have been decoded correctly on NVIDIA hardware when asking for RGB output data.

## NDI 5.1.0

* Bug fix in Premiere plugin that solved A/V sync problems when the NDI side-car index files have been deleted. In recent versions of NDI we moved our recording to use 64-bit MOV indexing in the headers which allows for much longer files to be recorded, this change needed to be reflected in the Premiere plugin.
* Discovery servers may now be specified by a DNS name and not just as an IP address.
* The NDI SDK uses typed values for returning “instances” instead of relying on void\*. For most code this will be a change that does not require any updates, however if your code assumes that these types are void\* then you will need to update it. Most importantly, however, this avoids the potential for mixing up the types passed into NDI functions, which reduces the chances of potential bugs in code that uses the NDI SDK.
* Fix memory leak when using “chop” on NDI recording code.
* The NDI installer and uninstallers will silently shut down the tools launcher and no longer give you a warning when installing.
* The NDI SDK will validate XML messages being sent before they are transmitted; this ensures that there is no vulnerability in which a remote application might be able to crash by sending data that is not XML. The only SDK assumption, of course, is that downstream NDI receivers correctly parse XML being received, which is something that the SDK is unable to control directly.
* NDI Studio Monitor has much lower GPU usage when running in Low Latency mode.
* NDI Bridge will transcode into full-bandwidth with increased compatibility for hardware high-bandwidth decoders that do not have full support for the entire set of NDI codec capabilities. This should significantly improve compatibility with hardware decoders.
* NDI Bridge includes an option that allows it to avoid automatic detection of the external IP and port number (and detection of whether port forwarding is enabled). This allows you to use NDI Bridge when you are not on a public network at all.
* Important changes to the performance of RUDP on Linux platforms enable Generic Segmentation Offload (GSO) for UDP sending, which might also be referred to as UDP\_SEGMENT, which was made available in Linux Kernel 4.18. This results in significantly reduced CPU overhead.
* Removed limitation on how many NDI senders can be created on a single machine.

### NDI 5.0.11

* Bug fix for the NDI Transmit plug-in for Adobe CC applications.
* Version number update to match the public launch of NDI Tools that includes NDI Bridge.
* MOV files recorded by the NDI SDK or Tools were technically limited to about 14 hours long because the indexes for this file format have a size limitation of 32-bits in “number of clock samples”. By using the extended version of these headers there is no longer any reasonable restriction on the length of recordings.

### NDI 5.0.10

* The first version that includes a full NDI Bridge. The NDI Bridge executable supports three command line options, the first is “/join” which will start the application in the Join mode. “/local” and “/host” start the application in Local and Host mode respectively.
* Support for hardware rendering of the mouse cursor in Screen Capture without “trails” and without any performance overhead.
* KVM support for applications has been added to the Advanced SDK.\
  Support for the latest version of the Adobe CC applications that have been updated.
* Reliable UDP sending is likely to use lower CPU usage than previous versions. NDI will take advantage of computer systems with NICs that support hardware-accelerated UDP segmentation offload. This can be used to offload much of the CPU burden of high-bandwidth UDP sending. Better reliable UDP spreading of sending and receiving across multiple CPU cores.
* If you are connected to an NDI discovery server and the network connection is physically lost (i.e., the application or device on one side does not gracefully close the connection), this will now be detected and handled much more quickly than in previous versions.
* Lower CPU usage in many situations through more efficient use of queues without CPU locking.
* The NDI installer on Windows will check whether “Media Foundation” is available on your operating system and notify you if it needs to be installed. There are some local versions of Windows (e.g., Windows 10 Pro N) that might not include this as a default option.
* The NDI installer will back up your NDI configuration when it is being installed so that each update to Tools does not wipe out your configuration. The configuration should be completely removed when you uninstall NDI Tools completely.
* There was a fix on Apple SDKs for cases in which extra data was incorrect in HX2 streams; we now handle this condition gracefully.
* Apple SDKs with Free NDI SDK now combine all Apple platforms into a single installer, which, unfortunately, significantly increases the installer size. However, it does mean that one SDK covers Mac, iPad, iOS, and tvOS.
* Android SDK is available for free for use in mobile applications.

### NDI 5.0.9

* Launching an NDI application will no longer try to configure the firewall for you unless you have administration privileges. This avoids the potential for the annoying user-access-control dialog to come up each time you launch an NDI application – you are likely still then to see a firewall warning, which you will have to click on “Allow” access for, or NDI will not be able to access your network correctly.
* The NDI screensaver is now aware of multi-DPI settings when multiple monitors with different DPI settings are present on the system.
* The NDI Webcam tool will no longer show an NDI logo when connecting to sources. This makes the tool better for people who are using it in production.
* When auto-focus is switched off for a PTZ Monitor in Studio Monitor, we do not resend the previous focus distance, which makes the operation “feel better” because the focus stays set on the last value it was seen when auto-focus was used.
* Reliable UDP sending is likely to use lower CPU usage than previous versions. NDI will take advantage of computer systems with NICs that support hardware-accelerated UDP segmentation offload. This can be used to offload much of the CPU burden of high-bandwidth UDP sending.

### NDI 5.0.8

* Very significant changes have been made to RUDP sending to allow for much better network utilization on networks with high round-trip time or packet loss. In addition, RUDP support has significant updates with improved performance on WAN and MTU discovery.&#x20;
* Significant improvements to macOS, iOS, and Android have been made.&#x20;
* Some important notes have been added to the Advanced SDK documentation regarding how to efficiently send compressed frames to the network under poor conditions with the SDK.
* NDI SDK documentation has been updated with comments on performance, particularly related to some ARM platforms.

### NDI 5.0.7

* The NDI redistributable that was included with the SDK was not digitally signed; this has been corrected.
* Fix for NDI Screen Capture HX in which a lockup was possible if a device (e.g., blue-tooth keyboard) was removed at almost the same time that an NDI connection was closed.
* Fix a problem with Screen Capture in which we might not send the first frame if the current screen was not changing at the time the NDI connection was made.
* A condition was fixed in which Panasonic NDI cameras might not be detected on the network if they were added after NDI started and had not been running for more than a few minutes.

### NDI 5.0.6

* Change to the RUDP receiver (and somewhat the sender) to have higher performance on multi-core machines that are under load or running many threads. When possible, we now keep a thread in a wake-able state on each CPU core so that upon receiving data, we always know that it can be processed if there is any CPU time on the machine.
* In Screen Capture, it was possible that if a screen was not currently being updated, new NDI connections that occur do not receive a frame. The primary problem occurred when there was more than one connection at a time.
* The NDI Launcher application will notify you when a newer version is available, so you do not always need to monitor our website for changes!
* The NDI launcher will give you access to the NDI changes list so you can quickly see what has been updated in this version of NDI.
* A crash that could occur based on timing when applications connected and disconnected from the “Web Cam input” on Windows has been fixed.
* The discovery server allows you to specify a port number using the flag “-port 5959”. You can now choose any port you want, and in Access Manager or the configuration files, you can specify a discovery server address that includes a port number. This allows you, for instance, to run multiple discovery servers on a single machine that controls different sets of NDI sources.

### NDI 5.0.5

* Using NDI | HX v1 cameras with the Bridge Local mode (our transcoder) could cause video artifacts under certain network conditions due to the potential for compressed frames to be missed. This is now no longer possible.
* There was an issue where if a configuration file had a NIC filter set up that specified an adapter that did not exist on your machine, then we would bind to a NIC that did not exist, which would fail but might slow down the creation of connections or other possible symptoms. We now validate the configuration files against the NICs that are on the machine and check that the IP addresses are truly valid before attempting to bind to them.
* SDK sample code that shows how to display the average frame time and jitter in the advanced SDK.

### NDI 5.0.4

* NDI Remote had an issue with some cameras when moving between the high-quality and proxy streams. This has been resolved.
* Reliable UDP sending was always asynchronous in all sending, however it still needed to use a single thread per sending destination to ensure that individual connections could not interfere with other ones (which might have stalled). This is no longer the case, and RUDP can now send across all destinations with all buffers in flight without needing any additional threads to service the sending.

### NDI 5.0.3

* Improvements to Windows-based audio and video drivers will make them much more robust to applications that are less tolerant of “strange clocking”. In general, this makes things far more robust and should result in much better results. A placeholder image is shown when you are not connected to a source so that the application receives video instead of simply receiving no data.
* In the NDI Tools on Windows, clicking on the “Balloon help popups” in Screen Capture and Webcam input no longer opens the help but instead opens the context menu for that tool. This is very convenient and much less annoying! (Help is still available on the context menus).
* The Webcam tool on Windows will connect to audio much faster. Previously, when you started a call (e.g., with Skype, Teams, or Zoom), it took about half a second for the audio to start, which made communication difficult. This should no longer happen.
* Importantly, if you are using Webcam plugins with applications like Skype, Teams, or Zoom and your audio content is not just speech, you must disable noise suppression and echo cancellation. These effects are designed to make speech more audible against background sounds; however, they can also create significant audio artifacts on music or other non-speech content.
* Audio support on some much older NDI HX v1 converter devices was not working; this has now been solved.

### NDI 5.0.2

* The NDI discovery server has an optional parameter that allows you to bind it to a single NIC on your machine for improved security.
* The NDI discovery server displays output that is designed to be read by another application that might want to provide a UI on top of it.

### NDI 5.0.1

* Example code for showing how to connect to a device (as a receiver) and recover the tally state. This makes the implementation of external tally devices very simple. This uses the tally\_echo functionality.
* New example code included in NDI Advanced SDK.
* RUDP is available as a specific connection type setting on macOS, previously “auto” selected it because it is now the default but there should be a specific setting that enables it.
* The “preferred NIC” is not a guarantee that a NIC will be used on all protocols since there are some cases where it might not be possible; this is particularly true for the older transfer protocols. We have extended the support for NIC selection to include more modes and increase both sides of the connection (both sending and receiving).
* This is a bug fix for a case in which connection sharing might not have always worked, particularly when using loop-back on the local machine. This did not actually cause any visible problem but caused NDI to work a little less efficiently than it should have.
* Studio Monitor PTZ keyboard shortcuts handled “left” at a slightly different speed from “right”. These now match exactly.
* Changes to HX drivers on iOS that make them more compliant with app store rules. Bug fix for crash using NDI Audio Direct with Adobe Audition.

## NDI 5.0.0

* The first launch of NDI 5!
