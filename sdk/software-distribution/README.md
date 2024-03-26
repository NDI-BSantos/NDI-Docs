# Software Distribution

To clarify which files may be distributed with your applications, the following are the files and the distribution terms under which they may be used for the SDK.

{% hint style="info" %}
Note that open-source projects have the right to include the header files within their distributions, which may then be used with dynamic loading of the NDI libraries.
{% endhint %}

### Header Files (NDI\_SDK\_DIR\INCLUDE\\\*.H)

These files may be distributed with open-source projects under the terms of the MIT license and may be included in open-source projects (see “Dynamic Loading” section for preferred mechanism). However, the requirements of these projects in terms of visual identification of NDI shall be as outlined within the License section above.

### Binary Files (NDI\_SDK\_DIR\BIN\\\*.\*)

You may distribute these files within your application if your EULA terms cover the specific requirements of the NDI SDK EULA, and your application covers the terms of the License section above.

### Redistributables (NDI\_SDK\_DIR\REDIST\\\*.EXE)

You may distribute the NDI redistributable and install them within your own installer. However, you must make all reasonable efforts to keep the versions you distribute up to date. You may use the command line with `/verysilent` to install without any user intervention, but if you do, then you must ensure that the terms of the NDI license agreement are fully covered elsewhere in your application.

An alternative is to provide a user link to the NDI-provided download of this application at [ndi.link/NDIRedistV5](http://ndi.link/NDIRedistV5). At runtime, the location of the NDI runtime DLLs can be determined from the environment variable `NDI_RUNTIME_DIR_V5`.

### Content Files (NDI\_SDK\_DIR\LOGOS\\\*.\*)

You may distribute all files in this folder as you need and use them in any marketing, product, or web material. Please refer to the [NDI Brand Guidelines](https://app.gitbook.com/o/a38gd62OPjDFyE3J0iLo/s/oUOsQw9oPpQ8LFlxOpLI/).

### Libraries

Components included in the SDK provide support for finding (find), receiving (recv), and sending (send). These share common structures and conventions to facilitate development and may all be used together.

#### [NDI-SEND](./#ndi-send)

This is used to send video, audio, and metadata over the network. You establish yourself as a named source on the network, and then anyone may see and use the media that you are providing.

Video can be sent at any resolution and framerate in RGB(+A) and YCbCr color spaces, and any number of receivers can connect to an individual NDI-Send.

#### [NDI-FIND](./#ndi-find)

This is used to locate all of the sources on the local network that are serving media capabilities for use with NDI.

#### NDI-RECEIVE

This allows you to take sources on the network and receive them. The SDK internally includes all the requisite codecs and handles all the complexities of reliably receiving high-performance network video.

### Utilities

To make the library easy to use, the SDK includes several utilities that can be used to convert between common formats. For instance, conversion between different audio formats is provided as a service.

### Command Line Tools

There are also several important command line tools within the SDK, including a discovery server implementation and a command line application that can be used for recording.
