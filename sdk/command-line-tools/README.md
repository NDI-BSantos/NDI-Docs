# Command Line Tools

### Recording

A full cross-platform native NDI recording is provided in the SDK. This is provided as a command-line application in order to allow it to be integrated into both end-user applications and scripted environments. All input and output from this application is provided over `stdin` and `stdout`, allowing you to read and/or write to these in order to control the recorder.

The NDI recording application implements most of the complex components of file recording and may be included in your applications under the SDK license. The functionality provided by the NDI recorder is as follows:

* **Record any NDI source.** For full-bandwidth NDI sources, no video recompression is performed. The stream is taken from the network and simply stored on disk, meaning that a single machine will take almost no CPU usage in order to record streams. File writing uses asynchronous block file writing, which should mean that the only limitation on the number of recorded channels is the bandwidth of your disk sub-system and the efficiency of the system network and disk device drivers.
* **All sources are synchronized.** The recorder will time-base correct all recordings to lock to the current system clock. This is designed so that if you are recording a large number of NDI sources, the resulting files are entirely synchronized with each other. Because the files are written with timecode, they may then be used in a nonlinear editor without any additional work required for multi-angle or multi-source synchronization.
* Still better, if you lock the clock between multiple computer systems using NTP, recordings done independently on all computer systems will always be automatically synchronized.
* **The complexities of discontinuous and unlocked sources are handled correctly.** The recorder will handle cases in which audio and/or video are discontinuous or not on the same clock. It should correctly provide audio and video synchronization in these cases and adapt correctly even when poor input signals are used.
* **High Performance.** Using asynchronous block-based disk writing without any video compression in most cases means that the number of streams written to disk is largely limited only by available network bandwidth and the [speed of your drives](#user-content-fn-1)[^1]. On a fast system, even a large number of 4K streams may be recorded to disk!
* And much more... Having worked with a large number of companies wanting recording capabilities, we realized that providing a reference implementation that handles a lot of the edge cases and problems of recording would be hugely beneficial. Allowing all sources to be synchronized makes NDI a fundamentally more powerful and useful tool for video in all cases.
* The implementation provided is cross-platform and may be used under the SDK license in commercial and free applications. Note that audio is recorded in floating-point and so is never subject to audio clipping at record time.

Recording is implemented as a stand-alone executable, which allows it either to be used in your own scripting environments (both locally and remotely), or called from an application. The application is designed to take commands in a structured form from `stdin` and put feedback out onto `stdout`.

#### Command Line Arguments

The primary use of the application would be to run it and specify the NDI source name and the destination file- name. For instance, if you wished to record a source called `My Machine (Source 1)` into a file `C:\Temp\A.mov`. The command line to record this would be:

> `"NDI Record.exe" –I "My Machine (Source 1)" –o "C:\Temp\A.mov"`

This would then start recording when this source has first provided audio and video (both are required in order to determine the format needed in the file). **Additional command line options are listed below**:

<table data-full-width="true"><thead><tr><th width="238">Command Line Option</th><th>Description</th></tr></thead><tbody><tr><td><strong>-i "source-name"</strong></td><td><strong>Required option.</strong><br>The NDI source name to record.</td></tr><tr><td><strong>-o "file-name"</strong></td><td><strong>Required option.</strong><br>The filename you wish to record into. Please note that if the filename already exists, a number will be appended to it to ensure that it is unique.</td></tr><tr><td><strong>-u "url"</strong></td><td><strong>Optional</strong>.<br>This is the URL of the NDI source if you wish to have recording start slightly quicker or if the source is not currently visible in the current group or network.</td></tr><tr><td><strong>-nothumbnail</strong></td><td><strong>Optional</strong>. <br>Specify whether a proxy file should be written. By default, this option is enabled.</td></tr><tr><td><strong>-noautochop</strong></td><td><strong>Optional.</strong><br>When specified, this specifies that if the video properties change (resolution, framerate, aspect ratio), the existing file is chopped, and a new one starts with a number appended. When false, it will simply exit when the video properties change, allowing you to start it again with a new file name should you want. By default, if the video format changes, it will open a new file in that format without dropping any frames.</td></tr><tr><td><strong>-noautostart</strong></td><td><strong>Optional.</strong><br>This command may be used to achieve frame-accurate recording as needed. When specified, the record application will run and connect to the remote source; however, it will not immediately start recording. It will then start immediately when you send a &#x3C;start/> message to stdin.</td></tr></tbody></table>

Once running, the application can be interacted with by taking input on `stdin`, and will provide response onto `stdout`. These are outlined below.

If you wish to quit the application, the preferred mechanism is described in the input settings section. However, one may also press `CTRL+C` to signal an exit, and the file will be correctly closed. If you kill the recorder process while it is running, the resulting file will be invalid since QuickTime files require an index at the end of the file. The Windows version of the application will also monitor its launching parent process; if that should exit, it will correctly close the file and exit.

#### Input Settings

While this application is running, a number of commands can be sent to `stdin`. These are all in XML format and can control the current recording settings. **These are outlined as follows**.

<table data-full-width="true"><thead><tr><th width="378">Command Line Option</th><th>Description</th></tr></thead><tbody><tr><td><strong>&#x3C;start/></strong></td><td>Start recording at this moment; this is used in conjuction with the “-noautostart” command line.</td></tr><tr><td><strong>&#x3C;exit/></strong> or <strong>&#x3C;quit/></strong></td><td>This will cancel recording and exit the moment that the file is completely on disk.</td></tr><tr><td><strong>&#x3C;record_level gain="1.2"/></strong></td><td>This allows you to control the current recorded audio levels in decibels. 1.2 would apply 1.2 dB of gain to the audio signal while recording to disk.</td></tr><tr><td><strong>&#x3C;record_agc enabled="true"/></strong></td><td>Enable (or disable) automatic gain control for audio, which will use an expander/compressor to normalize the audio while it is being recorded.</td></tr><tr><td><strong>&#x3C;record_chop/></strong></td><td>Immediately stop recording, then restart another file without dropping frames.</td></tr><tr><td><strong>&#x3C;record_chop filename="another.mov"/></strong></td><td>Immediately stop recording, and start recording another file in potentially a different location without dropping frames. This allows a recording location to be changed on the fly, allowing you to span recordings across multiple drives or locations.</td></tr></tbody></table>

#### Output Settings

Output from NDI recording is provided onto `stdout`. The application will place all non-output settings onto `stderr` , allowing a listening application to distinguish between feedback and notification messages. For example, in the run log below, different colors are used to highlight what is placed on `stderr` (<mark style="color:blue;">blue</mark>) and `stdout` (<mark style="color:green;">green</mark>).

{% code fullWidth="true" %}
```json
NDI Stream Record v1.00
Copyright (C) 2023 Vizrt NDI AB. All rights reserved.

[14:20:24.138]: <record_started filename="e:\Temp 2.mov" filename_pvw="e:\Temp 2.mov.preview" frame_rate_n="60000" frame_rate_d="1001"/>
[14:20:24.178]: <recording no_frames="0" timecode="732241356791" vu_dB="-23.999269" start_timecode="732241356791"/>
[14:20:24.209]: <recording no_frames="0" timecode="732241690457" vu_dB="-26.976938"/> 
[14:20:24.244]: <recording no_frames="2" timecode="732242024123" vu_dB="-20.638922"/> 
[14:20:24.277]: <recording no_frames="4" timecode="732242357789" vu_dB="-20.638922"/> 
[14:20:24.309]: <recording no_frames="7" timecode="732242691455" vu_dB="-17.237122"/> 
[14:20:24.344]: <recording no_frames="9" timecode="732243025121" vu_dB="-19.268487"/> 
...
[14:20:27.696]: <record_stopped no_frames="229" last_timecode="732273722393"/>
```
{% endcode %}

Once recording starts, it will put out an XML message specifying the filename for the recording and provide you with the framerate.

It then gives you the timecode for each recorded frame and the current audio level in decibels (if the audio is silent, then the dB level will be -inf). If a recording stops, it will give you the final timecode written into the file. Timecodes are specified as UTC time since the Unix Epoch (1/1/1970 00:00) with 100 ns precision.

#### Error Handling

A number of different events can cause recording errors. The most common is when the drive system that you are recording to is too slow to record the video data being stored on it, or the seek times to write multiple streams end up dominating the performance (note that we do use block writers to avoid this as much as possible).

The recorder is designed to never drop frames in a file; however, when it cannot write to disk sufficiently fast, it will internally “buffer” the compressed video until it has fallen about two seconds behind what can be written to disk. Thus, temporary disk or connection performance issues do not damage the recording.

Once a true error is detected, it will issue a record-error command as follows:

> `[`<mark style="color:blue;">`14:20:24.344`</mark>`]: <record_error error="The error message goes here."/>`

If the option for autochop is enabled, the recorder will start attempting to write a new file. This process ensures that each file always has all frames without drops, but if data needs to be dropped because of insufficient disk performance, that data will be missing between files.

### NDI Discovery Service

The NDI discovery service is designed to allow you to replace the automatic discovery NDI uses with a server that operates as a centralized registry of NDI sources.

This can be very helpful for installations where you wish to avoid having significant mDNS traffic for a large number of sources or in which [multicast is not possible](#user-content-fn-2)[^2] or desirable. When using the discovery server, NDI is able to operate entirely in unicast mode and thus in almost any installation.

The discovery server supports all NDI functionalities, including NDI groups.

#### Server

Using a discovery server is as simple as running the application in `Bin\Utilities\x64\NDI Discovery Service.exe`. This application will then run a server on your local machine that accepts incoming connections with senders, finders, and receivers and coordinates amongst them all to ensure they are all visible to each other.

If you wish to bind the discovery server to a single NIC, then you can run it with a command line that specifies the NIC to be used. For instance:

> `"NDI Discovery Service.exe" -bind 196.168.1.100`

{% hint style="warning" %}
If you are installing this on a separate machine from the SDK, you should ensure that the Visual Studio 2019 C runtime is installed on that machine and that the NDI licensing requirements are met.
{% endhint %}

32-bit and 64-bit versions of the discovery service are available, although the 64-bit version is recommended. The server will use very little CPU usage, although when there are a very large number of sources and connections, it might require RAM and some network traffic between all sources to coordinate source lists.

{% hint style="info" %}
It is, of course, recommended that you have a static IP address so that any clients configured to access it will not lose connections if the IP is dynamically re-assigned.
{% endhint %}

#### Clients

Clients should be configured to connect with the discovery server instead of using mDNS to locate sources. When there is a discovery server, the SDK will use both mDNS and the discovery server for _finding and receiving_ so as to locate sources on the local network that are not on machines configured to use discovery.

For _senders_, if a discovery service is specified, mDNS will not be used; these sources will _only_ be visible to other finders and receivers that are configured to use the discovery server.

#### Configuration

In order to configure the discovery server for NDI clients, you may use the **Access Manager** Tool (included in the [**NDI Tools Core Suite**](https://ndi.video/tools/ndi-core-suite/)) to enter the IP address of the discovery server machine.

To configure the discovery server for NDI clients, you may use Access Manager (included in the NDI Tools bundle) to enter the IP address of the discovery server machine.

It is possible to run the Discovery server with a command line option that specifies which NIC it is operating from with:

> `DiscoveryServer.exe -bind 192.168.1.100`

This will ask the discovery server to only advertise on the IP address specified. Likewise, it is possible to specify a port number that will be used for the discovery server using:

> `DiscoverytServer.exe -port5400`DiscoveryServer.exe -port 5400     &#x20;

This allows you to work on a non-default port number or run multiple discovery servers for multiple groups of sources on a single machine. If a port of 0 is specified, then a port number is selected by the operating system and will be displayed at run-time.

#### Redundancy and Multiple Servers

Within NDI 5, there is full support for redundant NDI discovery servers. When one configures a discovery server, it is possible to specify a comma-delimited list of servers (e.g. “192.168.10.10, 192.168.10.12”), and then they will all be used simultaneously. If one of these servers goes down, as long as one remains active, then all sources will remain visible at all times. As long as at least one server remains active, then no matter what the others do, all sources can be seen.

This multiple-server capability can also be used to ensure entirely separate servers to allow sources to be broken into separate groups, which can serve many workflow or security needs.

### NDI Benchmark

To help gauge your machine performance for NDI, a tool is provided that will initiate one NDI stream per core on your system and measure how many 1080p streams can be decoded in real-time. Note that this number reflects the best-case performance and is designed to exclude any impact of networking and only gauge the system CPU performance.

This can be used to compare performance across machines, and because NDI is highly optimized on all platforms, it is a good measure of the total CPU performance that is possible when all reasonable opportunity is taken to achieve high performance on a typical system. For instance, on Windows, NDI will use all extended vector instructions (SSSE3 and up, including VEX instructions), while on ARM, it will use NEON instructions when possible.

[^1]: Note that in practice, the performance of the device drivers for the disk and network sub-systems quickly becomes an issue as well. Ensure that you are using well-designed machines if you wish to work with large channel counts.

[^2]: It is very common that cloud computing services do not allow multicast traffic.
