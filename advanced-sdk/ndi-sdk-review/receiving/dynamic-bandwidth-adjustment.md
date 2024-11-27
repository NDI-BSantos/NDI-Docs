# Dynamic Bandwidth Adjustment

{% hint style="danger" %}
For access to to this feature please contact [support@ndi.video)](mailto:support@ndi.video)&#x20;
{% endhint %}

Beginning with NDI 6.1, the Advanced SDK includes a new API that lets you change the receive bandwidth of a video stream. Previously, switching between ‘program quality’ and ‘preview quality’ video streams required you to create two separate receiver instances using the `NDIlib_recv_bandwidth_highest` and `NDIlib_recv_bandwidth_lowest` fields, respectively.

With the new API, you can use a single receiver instance to toggle between program and preview quality streams, provided that the receiver was created with the video stream enabled. Eliminating the need for two separate receiver instances reduces bandwidth usage, and enables seamless switching between program and preview quality streams.

{% hint style="info" %}
This functionality has no effect on audio or metadata-only receivers.
{% endhint %}

Please be aware that you require a special Vendor ID to enable and use this API (without this the API will fail). If you have not already done so, please contact the NDI support team to learn how to obtain the special Vendor ID.

### API Function

To trigger a bandwidth change on the video stream, use the following API:

`bool NDIlib_recv_set_bandwidth(NDIlib_recv_instance_t p_instance, NDIlib_recv_bandwidth_e bandwidth);`

The return value indicates whether the action was successfully performed. Keep in mind that the bandwidth change may not be instantaneous, and it could take a few frames for the change to be reflected in the video stream if you are actively capturing video using the NDIlib\_recv\_capture\_v3 function.

#### Example

We recommend reviewing the **NDI Advanced SDK example code** `(NDIlib_Recv_Change_Bandwidth)`, which provides a simple illustration of how to implement this feature.
