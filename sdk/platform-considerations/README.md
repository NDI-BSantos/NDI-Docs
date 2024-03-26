# Platform Considerations

Of course, all platforms are slightly different, and the location of configuration files and the settings can differ slightly between platforms. On all platforms, if there is an environment variable `NDI_CONFIG_DIR`set before initializing the SDK, then we will load the ndi-config.v1.json from this folder when the library is used.

### Windows

The Windows platform is fully supported and provides high performance in all paths of NDI. As with all operating systems, the x64 version provides the best performance. All modern CPU feature sets are supported. _Please note that the next major version of NDI will deprecate support for 32-bit Windows platforms._

We have found that on some computer systems, if you install "WireShark" to monitor network traffic, a virtual device driver called "NPCap Loopback Driver" installs can interfere with NDI, potentially causing it to fail to communicate. This is also a potential performance problem for networking since it is designed to intercept network traffic. This driver is not required or used by modern versions of Wireshark. If you find it is installed on your system, we recommend that you go to your network settings and use the context menu on the adapter to disable it.

The NDI Tools bundle for Windows includes “Access Manager”, a user interface for configuring most of the settings outlined above. These settings are also stored in `C:\ProgramData\NDI\ndi-config.v1.json`.

### Windows UWP

Unfortunately, the Universal Windows Platform imposes significant restrictions that can negatively affect NDI, and about which you need to be aware. These are listed below.

* The UWP platform does not allow the receiving of network traffic from Localhost. This means that any sources on your local machine will not be able to be received by a UWP NDI receiver.\
  \
  [https://docs.microsoft.com/en-us/windows/iot-core/develop-your-app/loopback](https://docs.microsoft.com/en-us/windows/iot-core/develop-your-app/loopback)
* The current Windows 10 UWP mDNS discovery library has a bug that will not correctly remove an advertisement from the network after the source is no longer available; this source will eventually “time out” on other finders; however, this might take a minute or two.
* Due to sandboxing, UWP applications cannot load external DLLs. This makes it unlikely that NDI|HX will work correctly.
* When you create a new UWP project, you must ensure you have all the correct capabilities specified in the manifest for NDI to operate. Specifically, at time of writing, you need:
  * Internet (Client & Server)
  * Internet (Client)
  * Private Networks (Client & Server)

### MacOS

The Mac platform is fully supported and provides high performance in all paths of NDI. As with all operating systems, the x64 version provides the best performance. Reliable UDP support requires macOS 10.14 or later, which enables IPv6 socket properties; NDI will work on earlier versions, but Reliable UDP will not be used.

Because of recent changes made within macOS in regard to the signing of libraries, if you wish **NDI HX** **version 1** to work within your applications, you should locate the options in XCode under “_Targets->Signing & Capabilities_” and ensure that the option “_Hardened Runtime -> Disable Library Validation_” is checked.

In iOS 14 and XCode 12, a new setting was introduced to enable support for mDNS and Bonjour. Under “_Bonjour Services_”, one should assign “_Item 0_” as being “_\_ndi.\_tcp._” in order for NDI discovery to operate correctly.

The configuration settings are stored in `$HOME/.ndi/ndi-config.v1.json`.

### Android

Because Android handles discovery differently than other NDI platforms, some additional work is needed. The NDI library requires the use of the “_NsdManager_” from Android, and there is no way for a third-party library to do this on its own. As long as an NDI sender, finder, or receiver is instantiated, an instance of the NsdManager will need to exist to ensure that Android’s Network Service Discovery Manager is running and available to NDI.

This is normally done by adding the following code to the beginning of your long-running activities:

> `private NsdManager m_nsdManager;`

At some point before creating an NDI sender, finder, or receiver, instantiate the NsdManager:

> `m_nsdManager = (NsdManager)getSystemService(Context.NSD_SERVICE);`

You will also need to ensure that your application is configured to have the correct privileges required for this functionality to operate.

### iOS

iOS supports NDI finding, sending, and receiving.

In iOS 14 and XCode 12, a new setting was introduced to enable support for mDNS and Bonjour. Under “_Bonjour Services_”, one should assign “_Item 0_” as being “_\_ndi.\_tcp._” In order for NDI discovery to operate correctly. It is also required to enable networking in the App sandbox settings.

The configuration settings are stored in `$HOME/.ndi/ndi-config.v1.json`.

### Linux

The Linux version is fully supported and provides high performance in all paths of NDI. The NDI library on Linux depends on two 3rd party libraries:&#x20;

> `libavahi-common.so.3`
>
> `libavahi-client.so.3`

The usage of these libraries depends on the `avahi-daemon` service to be installed and running.

The configuration settings are stored in `$HOME/.ndi/ndi-config.v1.json`.

{% hint style="warning" %}
Please take careful note of the comments under Linux in the Reliable UDP in the [Performance and Implementation](../performance-and-implementation/) section.
{% endhint %}
