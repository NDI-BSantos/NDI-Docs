# Windows DirectShow Filter

The Windows version of the NDI SDK includes a DirectShow audio and video filter. This is particularly useful for people wishing to build simple tools and integrate NDI video into WPF applications.

Both x86 and x64 versions of this filter are included in the SDK. If you wish to use them, you must first register those filters using regsvr32. The SDK install will register these filters for you. The redistributable NDI installer will also install and register these filters, which can be downloaded by users [**here**](http://ndi.link/NDIRedistV5). You may, of course, include the filters in your own application installers under the terms of the NDI license agreement.

Once the filter is registered, you can instantiate it by using the GUID:

{% code fullWidth="false" %}
```json
DEFINE_GUID(CLSID_NdiSourceFilter, 0x90f86efc, 0x87cf, 0x4097, 
            0x9f, 0xce, 0xc, 0x11, 0xd5, 0x73, 0xff, 0x8f);
```
{% endcode %}

The filter name is “NDI Source”. The filter presents audio and video pins you may connect to. Audio is supported in floating-point and 16-bit, and video is supported in UYVY and BGRA.

The filter can be added to a graph and will respond to the IFileSourceFilter interface. This takes “filenames” in the form `ndi://computername/source`. This will connect to the “source” on a particular “computer name“. For instance, to connect to an NDI source called “MyComputer (Video 1)”, you must escape the characters and use the following URL: `ndi://MyComputer/Video+1`

To receive just the video stream, use the audio=false option as follows:

> `NDI://computername/source?audio=false`

Use the video=false option to receive just the audio stream, as in the example below:

> `NDI://computername/source?video=false`

Additional options may be specified using the standard method to add to URLs, for example:

> `NDI://computername/source?low_quality=true` \
> \
> `NDI://computername/source?audio=false&low_quality=true&force_aspect=1.33333&rgb=true`

