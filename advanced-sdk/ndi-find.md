# NDI-FIND

This is provided to locate sources available on the network and is normally used in conjunction with **NDI-Receive**. Internally, it uses a cross-process P2P mDNS implementation to locate sources on the network. (It commonly takes a few seconds to locate all the sources available since this requires other running machines to send response messages.)

Although discovery uses mDNS, the client is entirely self-contained; Bonjour (etc.) is not required. mDNS is a P2P system that exchanges located network sources and provides a highly robust and bandwidth-efficient way to perform discovery on a local network.

On mDNS initialization (often done using the **NDI-FIND** SDK), a few seconds might elapse before all sources on the network are located. Be aware that some network routers might block mDNS traffic between network segments.

### Parameters

Creating the find instance is very similar to the other APIs – one fills out a `NDIlib_find_create_t` structure to describe the device that is needed. It is possible to specify a `NULL` creation parameter, in which case default parameters are used.

If you wish to specify the parameters manually, then the member values are as follows:

<table data-full-width="true"><thead><tr><th width="280">Supported Values</th><th></th></tr></thead><tbody><tr><td><strong>show_local_sources (bool)</strong></td><td>This flag tells the finder whether it should locate and report <strong>NDI-SEND</strong> sources that are running on the current local machine.</td></tr><tr><td><strong>p_groups (const char*)</strong></td><td>This parameter specifies groups for which this NDI finder will report sources. A full description of this parameter and what a <code>NULL</code> default value means is provided in the description of the <strong>NDI-SEND</strong> SDK.</td></tr><tr><td><strong>p_extra_ips (const char*)</strong></td><td><p>This parameter will specify a comma-separated list of IP addresses that will be queried for NDI sources and added to the list reported by NDI find.</p><p></p><p>These IP addresses need not be on the local network and can be in any IP visible range. <strong>NDI-FIND</strong> will be able to find and report any number of NDI sources running on remote machines and will correctly observe them coming online and going offline.</p></td></tr></tbody></table>

Once you have a handle to the **NDI-FIND** instance, you can recover the list of current sources by calling `NDIlib_find_get_current_sources` at any time. This will immediately return with the current list of located sources.

The pointer returned by `NDIlib_find_get_current_sources` is owned by the finder instance, so there is no reason to free it. It will be retained until the next call to `NDIlib_find_get_current_sources`, or until the `NDIlib_find_destroy` function is destroyed.

You can call `NDIlib_find_wait_for_sources` to wait until the set of network sources has been changed; this takes a time-out in milliseconds. If a new source is found on the network, or one has been removed before this time has elapsed, the function will return true immediately. If no new sources are seen before the time has elapsed, it will return false.

**The following code will create an** **NDI-FIND** **instance** **and then list the currently available sources**. It uses `NDIlib_find_wait_for_sources` to sleep until new sources are found on the network and, when they are seen, calls `NDIlib_find_get_current_sources` to get the current list of sources:

{% code fullWidth="true" %}
```json
// Create the descriptor of the object to create 
NDIlib_find_create_t find_create; 
find_create.show_local_sources = true; 
find_create.p_groups = NULL;

// Create the instance
NDIlib_find_instance_t pFind = NDIlib_find_create_v2(&find_create); 
if (!pFind)
    /* Error */;
    
while (true) // You would not loop forever of course ! 
{
    // Wait up till 5 seconds to check for new sources to be added or removed 
    if (!NDIlib_find_wait_for_sources(pFind, 5000))
    {
       // No new sources added!
       printf("No change to the sources found.\n"); 
    }
    else
    {
       // Get the updated list of sources 
       uint32_t no_sources = 0;
       const NDIlib_source_t* p_sources =
          NDIlib_find_get_current_sources(pFind, &no_sources);
          
       // Display all the sources.
       printf("Network sources (%u found).\n", no_sources); 
       for (uint32_t i = 0; i < no_sources; i++)
          printf("%u. %s\n", i + 1, p_sources[i].p_ndi_name);
    }
}
// Destroy the finder when you’re all done finding things 
NDIlib_find_destroy(pFind);
```
{% endcode %}

It is important to understand that mDNS discovery might take some time to locate all network sources. This means that an ‘early’ return to `NDIlib_find_get_current_sources` might not include all the sources on the network; these will be added (or removed) as additional or new sources are discovered. It commonly takes a few seconds to discover all sources on a network.

For applications that wish to list the current sources in a user interface menu, the recommended approach would be to create an `NDIlib_find_instance_t` instance when your user interface is opened and then – each time you wish to display the current list of available sources – call `NDIlib_find_get_current_sources`.&#x20;
