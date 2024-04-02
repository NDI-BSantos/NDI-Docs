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

### NDI Bridge Utility for Hardware

This command-line utility (codenamed **Embedded Bridge**), designed for Linux platforms, enables users to effectively advertise and transmit local NDI sources to remote endpoints via WAN connections. It functions as a Bridge client application that operates in Join mode.

Access to remote endpoints is provided by connecting the NDI Bridge Utility to an independent external NDI Bridge Host. Once the connection is successfully established, users can view and request streaming of remote sources over the WAN facilitated by tools like NDI Studio Monitor.

The NDI Bridge Utility is primarily meant for deployment on embedded devices (such as PTZ cameras) that can harness it to transmit NDI output to remote receivers across WAN connections and, conversely, to receive metadata or control instructions from a remote sender.

It's important to note that the NDI Bridge Utility does not support transcoding. Instead, it provides a straightforward passthrough over the WAN to the remote endpoint and seamless forwarding of all metadata from NDI clients, such as Studio Monitor. For instance, when users avail themselves of Studio Monitor’s integrated PTZ controls, NDI Embedded Bridge delivers those PTZ commands across the WAN to the connected camera. (Note that some metadata functionality may depend on capabilities granted by the vendor ID.)

Lastly, it's worth mentioning that all communication over NDI Bridge is encrypted for enhanced security.

#### Minimum requirements

* A minimum Linux kernel version of 4.18 or higher is required.
* The Linux kernel must have support for dual-stack IPv4 and IPv6.
* To achieve optimal performance, support for Generic Segmentation Offload (GSO) for UDP is essential. This requirement aligns with the UDP\_SEGMENT socket option. Without GSO support, UDP transmission may lead to a notable increase in CPU usage.

#### Command Line Arguments

This process operates as a command-line application and offers support for command-line options. Additionally, it is compatible with a JSON configuration file, where each individual entry aligns with a specific command-line option. Users of this application are required to provide a vendor ID as input; failure to do so will result in the application halting after the specified timeout period.

#### Command Line Options

**USAGE**

`./ndi-embedded-bridge [OPTIONS] [JSON CONFIG FILENAME]`

<table data-full-width="true"><thead><tr><th width="276">Command Line Option</th><th>Description</th></tr></thead><tbody><tr><td><strong>-help</strong></td><td>Displays available command line arguments with short descriptions.</td></tr><tr><td><strong>-help COMMAND</strong></td><td>Displays extended help for the given command line argument (e.g. -help address).</td></tr><tr><td><strong>-address "[ID@]ADDRESS[:PORT]"</strong></td><td>Target Host for the Join instance. (Default: "127.0.0.1:5990").</td></tr><tr><td><strong>-diag "[info|warn|error]"</strong></td><td>Set the diagnostic level of log output. (Default: "warn").</td></tr><tr><td><strong>-id "ID"</strong></td><td>The identity of the Bridge process. (Default: "MACHINE_NAME.PROCESS_ID"). If an identifier is not explicitly provided (either here or optionally through the address) then one will be automatically generated which will be the name of the machine on which the application is run.</td></tr><tr><td><strong>-logfile "FILE"</strong></td><td>Capture log output to a file (highly recommended in 'quiet' mode!).</td></tr><tr><td><strong>-machine "ID"</strong></td><td>Accept NDI sources only from the specified machine ID. (default: "").</td></tr><tr><td><strong>-port PORT</strong></td><td>Port number to use with the Host. (Default: 5990).</td></tr><tr><td><strong>-reportstats MILLISECONDS</strong></td><td>Enable reporting connection statistics every given millisecond. (Default: 500). The bandwidth data measured is in bytes per second (Bps).</td></tr><tr><td><strong>-retry SECONDS</strong></td><td>Amount of time to wait between attempts to connect to the Host. (Default: 5).</td></tr><tr><td><strong>-secure "KEY"</strong></td><td>Secure communications using the provided key.</td></tr><tr><td><strong>-source "NAME"</strong></td><td>Limit NDI discovery to just the NDI source name specified. The source name should match the device name and channel name on that device in the form of DEVICE_NAME (CHANNEL_NAME).</td></tr><tr><td><strong>-srcgroups "GROUP[,GROUP]"</strong></td><td>Limit the Join's NDI source discovery to just the specified groups.</td></tr><tr><td><strong>-vendor_id "ID"</strong></td><td>NDI Vendor ID. (default: "").</td></tr><tr><td><strong>-vendor_name "NAME"</strong></td><td>NDI Vendor name. (Default: "").</td></tr><tr><td><strong>-nolocal</strong></td><td>Exclude NDI sources local to the Join. (Default: false).</td></tr><tr><td><strong>-overwrite</strong></td><td>When enabled, any existing log output file will be overwritten. (Default: false).</td></tr><tr><td><strong>-quiet</strong></td><td>Suppress all console output and hide the console window. (Default: false).</td></tr><tr><td><strong>-bridge_name "ID"</strong></td><td>Alias for the "-id" option.</td></tr><tr><td><strong>-source_name "NAME"</strong></td><td>Alias for the "-source" option.</td></tr><tr><td><strong>-reportbps "MILLISECONDS"</strong></td><td>Alias for the "-reportstats" option.</td></tr></tbody></table>

**EXAMPLES**

{% code overflow="wrap" fullWidth="true" %}
```json
./ndi-embedded-bridge -address "NiceCamera@192.168.0.42:5990"

./ndi-embedded-bridge -bridge_name "NiceCamera" -address "192.168.0.42" -port 5990

./ndi-embedded-bridge -address "NiceCamera@192.168.0.42:5990" -source_name "PTZCamera (Channel 1)"

./ndi-embedded-bridge -address "NiceCamera@192.168.0.42:5990" -secure "123abc" -source "PTZCamera (Channel 1)"

./ndi-embedded-bridge -address "NiceCamera@192.168.0.42:5990" -secure "123abc" -source "PTZCamera (Channel 1)" -vendor_name "XXX" -vendor_id "YYY"
```
{% endcode %}

#### JSON Configuration Options

The application can be started by supplying the filename of a JSON file containing the settings.\
The filename must have a (`.json`) extension.&#x20;

The top-level object can contain two members:

* "ndi" object containing NDI configuration settings.
* "bridge" object whose members are equivalent to the command line arguments.

**EXAMPLES**

{% code overflow="wrap" fullWidth="true" %}
```json
./ndi-bridge "/etc/config/bridge.json"

{
	"ndi": {
		"machinename": " Hello World",
		"groups": {
			"recv": "Public"
		},
		"networks": {
			"discovery": "192.168.20.2:5959"
		},
		"vendor": {
			"name": "XXX",
			"id": "YYY"
		}
	},

	"bridge": {
		"address": "192.168.0.42",
		"port": 5990,
		"secure": "123abc",
		"diag": "info",
		"id": "NiceCamera"
	}
}
```
{% endcode %}

To gracefully exit the application, use the `CTRL+C` key combination. This will terminate the current bridge process and close the application.

#### Output Statistics

To assist you in identifying connectivity and network performance problems, the NDI Bridge Utility provides detailed statistics for connections and streams. These statistics are available using the '-reportstats' flag with a non-zero time.

The statics available focus on networking at the packet level rather than the NDI frame level. NDI frames are often split into multiple packets and multiple streams sharing a connection; thus, issues cannot typically be attributed to specific NDI frames.

The output from the application employs standard logging levels to categorize the severity or urgency of each message. Messages at the INFO level are marked with \[ I ]. WARN-level messages are marked with \[ W ]. And ERROR-level messages are marked with \[ E ].

**EXAMPLE**

The run log below illustrates various statistics and their respective parameters and supplies a detailed description of each.

{% code overflow="wrap" fullWidth="true" %}
```json
NDI Bridge Embedded v6.0.0.0
Copyright (C) 2023-2024 Vizrt NDI AB. All rights reserved.

[ W ][14:04:40.882] [Application.NDI.Bridge] Bridge id: NiceCamera
[ I ][14:04:40.882] [Application.NDI.Bridge] Initializing PersistentBuffer
[ I ][14:04:40.882] [Application.NDI.Bridge] Initializing NDI Library
[ I ][14:04:40.882] [Application.NDI.Bridge] Initializing rUDP
[ W ][14:04:41.019] [Join] NICECAMERA: Attempting to connect to Host at address 104.56.9.100:5990
[ I ][14:04:41.020] [Join] NICECAMERA: Bandwidth usage: Send=0Bps. Receive=0Bps. SendFail=0Bps.
[ W ][14:04:41.069] [Join] NICECAMERA: Connected to Host at 192.168.0.42:5990
[ I ][14:04:46.033] [Join] NICECAMERA: Bandwidth usage: Send=4756085Bps. Receive=78Bps. SendFail=0Bps.
[ I ][14:04:51.047] [Join] Connection statistics: Rtt=24517us. MinRtt=20728us. MaxRtt=46994us. SendLostPackets=107 SendCongestionCount=8 SendCongestionWindow=111770 RecvDroppedPackets=0 RecvReorderedPackets=0
[ I ][14:04:51.047] [Join] Control statistics: Scheduling=336us. Pacing=0us. AmplificationProt=0us. CongestionControl=1764614us. FlowControl=0us.
…
[ I ][14:05:36.178] [Join] Stream statistics "WORK-PC (VLC)" video_highq: IdFlowControl=0us. FlowControl=0us. App=3264084us.
```
{% endcode %}

#### Connection Statistics

<table data-full-width="true"><thead><tr><th width="249">Name</th><th>Description</th></tr></thead><tbody><tr><td><strong>Rtt</strong></td><td>Round-Trip Time is the duration in microseconds for a packet to travel from the sender to the receiver and back again. This metric is crucial for assessing network latency and performance.</td></tr><tr><td><strong>MinRtt, MaxRtt</strong></td><td>Minimum and Maximum Round-trip-time in microseconds represent the shortest and longest round-trip times observed during the connection's lifespan. These metrics offer insights into the range of network latency experienced by the connection.</td></tr><tr><td><strong>SendLostPackets</strong></td><td>The Number of packets that the sender believes are permanently lost, as no acknowledgment (ACK) of their transmission was received . This metric helps gauge the sender's perception of packet loss within the network.</td></tr><tr><td><strong>SendCongestionCount</strong> </td><td>Number of congestion events experienced by the sender, signaling instances where the congestion window is full and required bandwidth is unavailable, thus impacting connection performance.</td></tr><tr><td><strong>SendCongestionWindow</strong></td><td>Size of the sender's congestion window in bytes, which limits the data transmission rate to prevent network overload. This window adjusts dynamically based on network conditions to ensure the sender operates within the available bandwidth.</td></tr><tr><td><strong>RecvDroppedPackets</strong></td><td>Number of packets that were locally dropped on receive, indicating the count of packets not successfully received by the application on the receiving end (possibly due to exceeding a receiver's buffer capacity or receiving invalid packets).</td></tr><tr><td><strong>RecvReorderedPackets</strong></td><td>Number of packets received out of order (which can happen due to network congestion or routing changes).</td></tr></tbody></table>

#### Control Statistics

<table data-full-width="true"><thead><tr><th width="249">Name</th><th>Description</th></tr></thead><tbody><tr><td><strong>Scheduling</strong></td><td>Total time in microseconds that the connection was blocked due to packet transmission scheduling issues such as delays caused by congestion control algorithms or prioritization mechanisms.</td></tr><tr><td><strong>Pacing</strong></td><td>Total time in microseconds that the connection was blocked by pacing, which regulates the packet transmission rate to avoid congestion.</td></tr><tr><td><strong>AmplificationProt</strong></td><td>Total time in microseconds that the connection was blocked by amplification protection mechanisms, which guard against potential amplification attacks by limiting or blocking certain types of traffic.</td></tr><tr><td><strong>CongestionControl</strong></td><td>Total time in microseconds that the connection was unable to send data at its desired rate due to network congestion.</td></tr><tr><td><strong>FlowControl</strong></td><td>Total time in microseconds that the connection was blocked by flow control, which limits the amount of data a sender can transmit before receiving acknowledgment from the receiver.</td></tr></tbody></table>

#### Stream Statistics

<table data-full-width="true"><thead><tr><th width="249">Name</th><th>Description</th></tr></thead><tbody><tr><td><strong>IdFlowControl</strong></td><td>Total time in microseconds that a specific stream was blocked by its own flow control limit, which is separate from the overall connection-level flow control limit.</td></tr><tr><td><strong>FlowControl</strong></td><td>Total time in microseconds that a stream was blocked by flow control, indicating the cumulative duration during which the stream was unable to send data because it had reached the flow control limit set by the receiver.</td></tr><tr><td><strong>App</strong></td><td>Total time in microseconds that a stream was blocked by the application, indicating the cumulative duration during which the application prevented the stream from sending data, potentially due to application-specific reasons such as flow control or buffer constraints.</td></tr></tbody></table>

[^1]: Note that in practice, the performance of the device drivers for the disk and network sub-systems quickly becomes an issue as well. Ensure that you are using well-designed machines if you wish to work with large channel counts.

[^2]: It is very common that cloud computing services do not allow multicast traffic.
