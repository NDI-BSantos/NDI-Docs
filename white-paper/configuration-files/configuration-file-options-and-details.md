# Configuration file options and details

```
{ 
  "ndi": { 
    "machinename": "Hello World",
```

This option lets you change how your machine is identified on the network using NDI, overriding its local name. Use this with caution, as duplicate machine names on the network can disrupt mDNS and cause other sources to malfunction. It's crucial to ensure all machine names on the network are unique.&#x20;

***

```
    "send": { 
      "metadata": "<My very cool source/>" 
    }, 
```

In version 5 of NDI, it is possible to specify metadata for the current source when creating an NDI sender with the Advanced SDK. When using the NDI finder, it is then possible to receive this metadata to receive any number of properties about the source. Please note that you should always make your metadata in XML format and be careful to correctly escape it when adding it into the JSON configuration file.

***

```
"networks": { 
      "ips": "192.168.86.32,", 
      "discovery": "" 
    }, 
```

These IP addresses are for machines from which you want to discover NDI sources. Each machine runs a service on port 5960, which the configured machine connects to, allowing source discovery on those IPs without relying on mDNS. This setting is useful for pulling NDI streams from transmitters located on a different subnet. In Windows and macOS, you can configure this in NDI Access Manager under "External Source."

***

```
 "networks": { 
   "ips": "192.168.86.32,", 
   "discovery": "127.0.0.1,127.0.0.1" 
 }, 
```

Starting with NDI version 5, you can use a comma-separated list of discovery servers to enable full redundancy. For more details, check the discovery server section of this white paper. You can also include port numbers in the list if you don't want to use the default port 5959.\
\
When a Discovery server is used, receivers merge the list of sources from the server with those found via mDNS. Senders, on the other hand, will skip mDNS when a Discovery server is configured, allowing you to operate without network multicast if preferred.

***

```
"groups": { 
      "send": "Public", 
      "recv": "Public" 
    }, 
```

Here is the list of groups that senders on this system will automatically join by default. If no groups are specified, senders will be included in the Public group by default.

***

```
" sourcefilter": { 
      "regex": "MACHINE .*" 
    }, 
```

Starting with NDI version 5, you can specify a regular expression to further refine the set of sources visible to NDI finders. This advanced option gives you precise control over which sources will be accessible to the local machine. If the regular expression you provide is invalid, it will not be applied.

***

```
"adapters": { 
 "allowed": ["10.28.2.10","10.28.2.11"] 
}, 
```

Starting with NDI version 5, you can now specify which network adapters will be used for network transmission. Multiple NICs (Network Interface Cards) can be selected for transmitting and receiving video and audio data. This feature allows you to dedicate specific NICs to different types of data, such as keeping NDI video on one set of adapters while using a separate NIC for audio.

While NDI can automatically choose the optimal network adapters and intelligently manage bandwidth across them, it's generally recommended to allow NDI to make these selections. In some modes, NDI can balance bandwidth across multiple NICs, but using NIC teaming at the system level typically provides superior performance compared to what software can achieve alone.

If this setting is incorrectly configured to include NICs that don’t exist, NDI may not function properly. Additionally, keep in mind that systems operating on entirely different networks with distinct IP address ranges may not be robustly supported by the operating system, which could affect NDI's functionality in such configurations.

***

```
"rudp": { 
      "send": { 
        "enable": true 
      }, 
      "recv": { 
        "enable": true 
      },
```

Connection options are available in NDI version 5, allowing you to forcefully enable or disable the reliable UDP mode, which is the default connection type from NDI version 5 onwards.&#x20;

Separate settings are available for both sending and receiving, and both the source and receiver must permit this mode for it to be applied. By default, reliable UDP mode is enabled on both ends.

As the default connection type, reliable UDP mode is preferred for most network configurations, and we recommend using it whenever possible.

***

```
"multicast": { 
      "send": { 
        "ttl": 1, 
        "enable": false, 
        "netmask": "255.255.0.0", 
        "netprefix": "239.255.0.0" 
      }, 
      "recv": { 
        "enable": true 
      } 
    }, 
```

These settings control the use of multicast for receiving NDI streams. If multicast is explicitly disabled on a machine, even if the sender is configured to use multicast, the transmission will default to unicast. When multicast receiving is enabled, and the sender is on the same local network, the receiver can negotiate a multicast stream. However, if the sender is on a different network, this negotiation won't occur, as it could result in a multicast stream that cannot reach the receiver.

In cases where you have a properly configured network and can guarantee that a multicast stream can be reliably routed from a different network to the receiver’s local network, you can specify the sender’s subnet in the “subnets” setting to enable multicast negotiation.

```
"multicast": {
  "recv": {
   "enable": true,
   "subnets": [ "10.28.5.0/24", "10.28.4.0/24" ]
  }
},
```

These settings also pertain to multicast NDI configurations on this machine. The first setting controls whether multicast sending is enabled, with multicast sending being disabled by default. The next settings involve the IP address prefix and mask, which, in this example, define the multicast IP range as 239.255.0.0 to 239.255.255.255. NDI will use different multicast addresses within this range to help ensure efficient stream filtering by the network adapter. A range of multicast addresses must be available for NDI senders. The TTL value controls how many “hops” the multicast traffic can take, allowing it to traverse beyond the local network.

There are separate settings for sending and receiving, and both sides must allow this mode to be applied; by default, multicast mode is enabled for both sources and receivers.

***

```
"tcp": { 
      "send": { 
        "enable": false 
      }, 
      "recv": { 
        "enable": false 
              } 
    }, 
```

These settings control the enabling or disabling of multi-TCP for sending and receiving. If multi-TCP is turned off, unicast UDP will be used instead. Should unicast UDP also be disabled, the system will default to using a standard TCP connection. Sending and receiving each have their own separate settings. For this mode to function, it must be enabled on both the sending and receiving ends, though it is enabled by default for sources and receivers. In NDI version 5, multi-TCP is not the default mode, as Reliable UDP generally offers better performance.

***

```
    "unicast": { 
      "send": { 
        "enable": false 
      }, 
      "recv": { 
        "enable": false 
      } 
    }, 
```

These settings control the enabling or disabling of unicast UDP for sending or receiving data. If unicast UDP is disabled, the system will default to using the base TCP connection. Unicast settings dictate whether UDP with forward-error correction is used for transmission. While this can be configured, we strongly recommend keeping it enabled by default. Our extensive testing has shown that our UDP implementation handles poor network conditions and packet loss more effectively than TCP/IP, which may experience timeout issues when acknowledgment packets are occasionally dropped (a rare but possible occurrence over extended periods).

The UDP implementation also includes features like paced network sending with zero-copy scatter-gather lists and jittered timing, which help minimize packet loss on networks with multiple synchronized video streams. By default, 4Kb UDP packets are used, though enabling jumbo packets on the network is unnecessary.

All NDI versions will fall back to TCP/IP if a specific protocol isn't supported by both the sender and receiver. It's important to note that a sender can simultaneously transmit in multiple modes, depending on the requirements of the receivers.

There are distinct UDP unicast settings for sending and receiving. Both the sender and receiver must allow UDP mode for it to be used, and this is enabled by default on both sides. However, in NDI version 5, unicast UDP is not the default mode, as better performance is achieved with Reliable UDP.

***

```
"codec": { 
      "shq": { 
        "quality": 100, 
        "mode": "auto" 
      } 
    }, 
  } 
} 
```

These settings are exclusive to devices or software applications developed with the NDI Advanced SDK, providing the ability to override the default codec quality settings of NDI. The “quality” setting operates on a percentage scale, controlling the bit-rate— for example, a value of 200 would instruct NDI to target a bitrate twice the default. It's important to be cautious when selecting high bitrates, as this can significantly increase CPU usage for compression and decompression, as well as place additional strain on the network and the operating system's networking stack. Once the bitrate reaches the maximum level for a specific media type (e.g., when the codec’s quality value hits its limit), further increases may have no effect.

The “mode” option allows you to force NDI into a specific color mode. By default, the mode is set to “auto,” which uses heuristics to optimally allocate bits between luminance and chroma fields. However, you can specify “4:2:2” or “4:2:0” to enforce a particular chroma subsampling mode. Be aware that forcing a specific mode can sometimes result in lower quality than allowing the codec to dynamically allocate bits for the best possible PSNR (Peak Signal-to-Noise Ratio).
