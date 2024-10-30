# Creating and Destroying Devices

### Creating and destroying devices

It is very simple to create and destroy a device. Simply create an NDI receiver that one wishes to receive audio from and pass it into `NDIlib_avsync_create`, this will return an instance of type `NDIlib_avsync_instance_t` which may be used until you no longer need it. Once you no longer need it you should call `NDIlib_avsync_destroy`. Please note that you should destroy a device before the corresponding `NDIlib_recv_instance_t` is called since an internal reference to this object is kept within the `NDIlib_avsync_instance_t`.
