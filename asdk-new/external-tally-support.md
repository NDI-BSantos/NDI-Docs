# External Tally Support

Some sending might have knowledge of whether they are on output that might come from another non-NDI device. In effect, the sender side wants to indicate how it is known to be used and have that be reflected both to the local device but also down-stream in the "echoed tally messages". To achieve this there is a function on the sender side that updates the tally state.

```
bool NDIlib_send_set_tally(
    NDIlib_send_instance_t p_instance, const NDIlib_tally_t* p_tally
);
```

This function will "reference" count the local tally and the remote tally to give an indicator both on the local device and on the remote connections on the combined NDI and local tally values.
