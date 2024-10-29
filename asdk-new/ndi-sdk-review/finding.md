# Finding

### Finding

As documented elsewhere, the NDI filter allows you to locate all NDI sources on the network. When using a discovery server, the Advanced SDK allows you to specify per connection metadata by specifying a "source.metadata" field in the sender JSON (see documentation for configuration files). When using an NDI finder one can receive the list of all sources on the network with their associated metadata using the function:

<pre><code>const NDIlib_source_v2_t* NDIlib_find_get_current_sources_v2(
<strong>    NDIlib_find_instance_t p_instance, uint32_t* p_no_sources
</strong>);
</code></pre>

This function returns the list of sources, including their metadata in a fashion that is identical to the regular `NDIlib_find_get_current_sources` function.
