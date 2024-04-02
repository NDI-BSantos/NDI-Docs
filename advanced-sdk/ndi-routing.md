# NDI Routing

Using NDI routing, you can create an output on a machine that looks just like a ‘real’ video source to all remote systems. However, rather than producing actual video frames, it directs sources watching this output to receive video from a different location.

For instance: if you have two NDI video sources - “Video Source 1” and “Video Source 2” – you can create an NDI\_router called “Video Routing 1” and direct it at “Video Source 1”. “Video Routing 1” will be visible to any NDI receivers on the network as an available video source. When receivers connect to “Video Routing 1”, the data they receive will actually be from “Video Source 1”.

NDI routing does not actually transfer any data through the computer hosting the routing source; it merely instructs receivers to look at another location when they wish to receive data from the router. Thus, a computer can act as a router exposing potentially hundreds of routing sources to the network – without any bandwidth overhead. This facility can be used for large-scale dynamic switching of sources at a network level.

You create a video routing source using:

```json
NDIlib_routing_instance_t NDIlib_routing_create( 
    const NDIlib_routing_create_t* p_create_settings);
```



The creation settings allow you to assign a name and group to the source that is created. Once the source is created, you can tell it to route video from another source using:

```
bool NDIlib_routing_change(NDIlib_routing_instance_t p_instance, 
                           const NDIlib_source_t* p_source);
```

and:

```
bool NDIlib_routing_clear(NDIlib_routing_instance_t p_instance);
```

Finally, when you are finished, you can dispose of the router using:

```
void NDIlib_routing_destroy(NDIlib_routing_instance_t p_instance);
```
