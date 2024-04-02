# Startup and Shutdown

The functions `NDIlib_initialize` and `NDIlib_destroy` can be called to initialize or de-initialize the library. Although never absolutely required, it is recommended that you call these (Internally, all objects are reference-counted; the libraries are initialized on the first object creation and destroyed on the last, so these calls are invoked implicitly.)

In the latest version of NDI on platforms where the application has permissions, it will attempt to configure the firewall settings for the application to allow it to perform video communication over NDI.

The only negative side-effect of this behavior is that more work will be done than is required if you repeatedly create and destroy a single object. These calls allow that to be avoided. There is no scenario under which these calls can cause a problem, even if you call `NDIlib_destroy` while you still have active objects. `NDIlib_initialize` will return false on an unsupported CPU.
