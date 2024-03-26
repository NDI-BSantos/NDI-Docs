# Configuration Files

The NDI configuration settings are stored in JSON files. The location of these files varies per platform and is described in the next section of the manual.

{% hint style="info" %}
Please note that when using the NDI Advanced SDK, all settings are per instance and so entirely separate settings may be used for every finder, sender, and receiver; this is incredibly powerful by allowing you to specify per sender or receiver which NICs are used, formats are used, what the machine name is, etc...
{% endhint %}

Please pay extra attention to the value types, as it is important that these match what is listed here (e.g., true rather than “true”). Also, please note that all these parameters have default values that are the recommended best configuration for most machines; we only suggest you change these values if there is a very specific need for your installation.



<table data-header-hidden data-full-width="true"><thead><tr><th width="435"></th><th></th></tr></thead><tbody><tr><td><pre class="language-json" data-line-numbers data-full-width="false"><code class="lang-json">{
   "ndi": {
      "machinename": "Hello World",
</code></pre></td><td>This is an option that allows you to change how your machine is identified on the network using NDI, overriding the local name of the machine. This option should be used with great care since a clash of machine names on the network is incompatible with mDNS and can cause all other sources not to work correctly. When using this, it is essential to ensure that all machine names on the network are unique. We recommend avoiding using this parameter when at all possible.</td></tr><tr><td><pre class="language-json" data-line-numbers><code class="lang-json">"send": {
  "metadata": "&#x3C;My very cool source/>"
},
</code></pre></td><td>In NDI 5, it is possible to specify metadata for the current source when creating an NDI sender with the Advanced SDK. When using the NDI finder, it is then possible to receive this metadata in order to receive any number of properties about the source. Please note that you should always make your meta-data in XML format and be careful to escape it correctly when adding it to the JSON configuration file.</td></tr><tr><td><p></p><pre class="language-json" data-line-numbers><code class="lang-json">"networks": {
  "ips": "192.168.86.32,",
  "discovery": "127.0.0.1,127.0.0.1"
},
</code></pre></td><td><p>These are extra IP addresses for machines from which you wish to discover NDI sources. Each local machine runs a service on port 5960, which is then connected to the machine this configuration is run on. This allows sources to be discovered on those IP addresses without needing mDNS to discover it.<br><br><mark style="background-color:yellow;">Hint: When a Discovery server is used, receivers combine the list of sources found on the discovery server with those discovered via mDNS. Senders, however, will avoid using mDNS when a discovery server is configured, allowing you to run entirely without network multicast if you desire.</mark><br><br>Starting in NDI 5, it is possible to use a comma-delimited list of discovery servers for full support for redundancy. For more information, please review the section of the manual regarding the discovery server.</p><p></p><p>The discovery server list may include port numbers. If you do not list the port numbers, the default port of 5959 will be used.</p></td></tr><tr><td><p></p><pre class="language-json" data-line-numbers><code class="lang-json">"groups": {
  "send": "Public",
  "recv": "Public"
},
</code></pre></td><td>This is the list of groups that the senders on this system are going to be part of by default. If groups are not specified, senders will be part of the public group by default.</td></tr><tr><td><pre class="language-json" data-line-numbers><code class="lang-json">"sourcefilter": {
  "regex": "MACHINE .*"
},
</code></pre></td><td>In NDI 5, there is the ability to specify a regular expression that will be used to filter further the set of sources that will be visible to NDI finders. This is an advanced option that allows you to specify exactly which sources are going to be visible to the local machine. If your regular expression is not valid, then it will not be applied.</td></tr><tr><td><p></p><pre class="language-json" data-line-numbers><code class="lang-json">"adapters": {
 "allowed": ["10.28.2.10","10.28.2.11"]
},
</code></pre><p></p></td><td><p>Starting in NDI version 5, this lists all the network adapters that will be used for network transmission. One or more NICs can be used for the transmission and receiving of video and audio data. This capability can be used to ensure that the NDI primary stream data remains on a group of network adapters, allowing you to ensure that dedicated audio is on a separate network card from the NDI video.<br></p><p>It is generally preferred that you let NDI select the network adapters automatically, as it can smartly select which to use and how to choose the ones that result in the best bandwidth. While in some modes, NDI can automatically balance bandwidth across multiple NICs, it is normally better for you to use NIC teaming at a machine configuration level which can result in much better performance than what is possible in software.<br></p><p>If this setting is configured incorrectly to specify NICs that might not exist, then NDI might fail to function correctly. Also please note that the operation of computer systems that are separately on entirely different networks with different IP address ranges is often not handled robustly by the operating system, and NDI might not fully function in these configurations.</p></td></tr><tr><td><p></p><pre class="language-json" data-line-numbers><code class="lang-json">"rudp": {
  "send": {
    "enable": true
  }, 
  "recv": {
    "enable": true
  }
},
</code></pre></td><td><p>The following connections are available in NDI version 5 and allow the force enabling and disabling of the reliable UDP mode, which is the default connection type on NDI version 5 and later. The full details of this connection type are described in the section for “Performance and Implementations” section of this manual.</p><p></p><p>There are separate settings for sending and receiving. Both sides need to allow this mode to be applied; sources and receivers have it enabled by default.</p><p></p><p>This is the default connection type and represents the preferred type for most network configurations, and we recommend its use where possible.</p></td></tr><tr><td><p></p><pre class="language-json" data-line-numbers><code class="lang-json">"multicast": {
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
</code></pre></td><td><p>These settings enable or disable the use of multicast for receiving. If you explicitly disable it on a machine then, even if the sender is configured for multicast, it will use unicast. When multicast receiving is enabled, and a sender is available in the same local network, the receiver can negotiate for a multicast stream to be sent. If the sender is not on the same local network, this negotiation does not occur (since it could lead to a multicast stream being sent but never able to arrive at the receiver). If you have a correctly configured network and can ensure a multicast stream can route reliably from a different network to the receiver’s local network, you can specify the sender’s subnet in the “subnets” setting to allow multicast negotiation to occur.<br><br>These settings pertain to the multicast NDI setting on this machine. The first setting determines whether multicast sending is enabled or not. By default, multicast sending is disabled. Next is the IP address prefix and mask. In this example, multicast IP addresses will be chosen in the range 239.255.0.0 - 239.255.255.255. NDI will attempt to use different multicast addresses to ensure that the streams can be filtered efficiently by the network adapter. NDI senders need a range of multicast addresses available. The TTL value controls how many “hops” the multicast sending traffic will take, allowing it to move outside of the local network.<br><br>There are separate settings for sending and receiving. Both sides need to allow this mode to be applied; sources and receivers have it enabled by default.<br></p><p>We generally discourage the use of Multicast since configuration and ensuring that high performance is achieved is very difficult at a network level; in most cases, the default protocols (particularly reliable UDP) perform much better.</p></td></tr><tr><td><p></p><pre class="language-json" data-line-numbers><code class="lang-json">"tcp": {
  "send": {
    "enable": false
  }, 
  "recv": {
    "enable": false
  }
},
</code></pre></td><td><p>These settings enable or disable multi-TCP sending or receiving. If multi-TCP is disabled, then unicast UDP will be used. If unicast UDP is also disabled, then the base TCP connection will be used.</p><p></p><p>There are separate settings for sending and receiving. Both sides need to allow this mode to be applied; sources and receivers have it enabled by default.</p><p></p><p>Multi-TCP is not the default mode in NDI 5, which performs better with Reliable UDP.</p></td></tr><tr><td><p></p><pre class="language-json" data-line-numbers><code class="lang-json">"unicast": {
  "send": {
    "enable": false
  },
  "recv": {
    "enable": false
  }
},
</code></pre></td><td><p>These settings enable or disable unicast UDP sending or receiving. If unicast UDP is disabled, then the base TCP connection will be used.</p><p></p><p>Unicast settings determine whether UDP with forwards- error correction is used for sending. While configurable, we recommended that this be enabled by default and not changed. Our experience has been that our UDP implementation handles poor networks and packet loss more robustly than TCP/IP, which can encounter timeout problems when acknowledgment packets are dropped (while rare, this can happen over a period of hours).</p><p></p><p>The UDP implementation also fully implements paced network sending with zero memory copy scatter-gather lists and jittered timing to reduce the chance of packet loss on networks with many synchronized video streams. By default, 4Kb UDP packets are used, although jumbo packets do not need to be enabled on the network.</p><p></p><p>All versions of NDI fall to TCP/IP if a particular protocol is not supported by both sides. Again, note that that a sender implementation can simultaneously send internally in multiple modes based on what receivers require.</p><p></p><p>There are separate UDP unicast settings for sending and receiving. Both sides need to allow this for UDP mode to be applied; sources and receivers have it enabled by default.</p><p></p><p>Unicast UDP is not the default mode in NDI 5, which performs better with Reliable UDP.</p></td></tr><tr><td><p></p><pre class="language-json" data-line-numbers><code class="lang-json">"codec": {
      "shq": {
        "quality": 100,
        "mode": "auto"
      }
    }, 
  }
}
</code></pre></td><td><em>These settings are only available in the NDI Advanced SDK</em> and allow you to override the default codec quality settings of NDI. The “quality” setting is a percentage scale to apply to the bit-rate control; for instance, a value of 200 would mean that NDI targets a bitrate that is double the NDI default. Be careful when specifying high bitrates because the CPU usage required for compression and decompression might increase, and the strain on the network and the OS networking stack is correspondingly increased. Once the bitrate hits a maximum level for a particular media type (e.g., the codec q value becomes the maximum), then increasing it further might have no impact.<br><br>The “mode” allows you to force NDI into a particular color mode. The default is “auto,” which uses heuristics to best allocate bits between the luminance and chroma fields. You may specify “4:2:2” or “4:2:0” here to force the codec into a particular chroma-subsampling mode. Please note that often forcing it into a particular mode will cause the codec to be less high quality than letting the codec choose the bit allocation that results in the best PSNR.</td></tr><tr><td></td><td></td></tr></tbody></table>
