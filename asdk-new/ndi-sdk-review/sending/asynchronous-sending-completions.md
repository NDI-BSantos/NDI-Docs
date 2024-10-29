# Asynchronous Sending Completions

When using asynchronous sending, the default way that it works is that you provide a buffer to the call and ownership of that buffer is held by the SDK until the return of the next asynchronous sending call. While this allows fully asynchronous sending, a problem that can occur is that if multiple receivers are connected to a source and one of those receivers is not responsive (either because it has become ungracefully disconnected or because its connection is slow) then sending a new frame via an asynchronous send might lock until sending to that network receiver is complete, causing potential video delays or freezes.

It is possible to assign an asynchronous completion handler for an NDI sender. When this is assigned, each call to an asynchronous sending function like `NDIlib_send_send_video_async_v2` and `NDIlib_send_send_video_scatter_async` will always return immediately and the frame submitted will be sent to all connections that are currently actively receiving. When the buffer is no longer needed by the SDK the completion routine is called which tells you that you may now free this buffer. Note that while it is rate, if a connection is stalled and holds onto a buffer until the connection times-out that there is a chance that completions are called out of order.

One can assign a completion handler for asynchronous sending calls, with opaque data that is passed into the completion routine with:

```
void NDIlib_send_set_video_async_completion(
    NDIlib_send_instance_t p_instance,
    void* p_opaque,
    NDIlib_video_send_async_completion_t p_deallocator
);
```



If one passes in a `NULL` as a completion callback pointer, then the behavior is returned to the default buffer ownership behavior which will lock on a send until the previous call has been asynchronously sent.

It is important to know that the completion will occur on a thread that is called by the NDI SDK, you should ensure that you return quickly from this allocation or free call if you wish to avoid video or audio stalling. Your code should be re-entrant since it might be called from multiple threads at once and should avoid any permanent locks that might occur between sending frames and completion handlers. The handler can be changed out at any time, including at run-time. When you provide a completion handler, the callback is called exactly once for every asynchronous sending call.

When a connection is closed (`using NDIlib_send_destroy`), all outstanding completions will be called before the destroy event is complete.

####
