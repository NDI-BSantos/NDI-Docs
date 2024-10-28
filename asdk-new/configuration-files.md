# Configuration Files

## Configuration Files

All default settings for NDI® are controlled through a configuration file. This file is located at $HOME/.ndi/ndi-config.v1.json. It contains settings for multicast and unicast sending as well as vendor information. Your vendor ID is specified here and must be registered with NDI for compressed data pass-through to be enabled.

The configuration file will automatically be loaded off disk by default and these settings used. If you wish to work this way, you can simply restart your application when configuration settings on the device are changed, and the settings will be updated.

Alternatively, the NDI creation functions can be passed in a string representation of this configuration file, allowing you to change it for senders and receivers by simply recreating them at run time.

There is a setting for p\_config\_json that may be set in creation of devices to permit an in-memory version of the configuration file to be passed in. An example showing how to set p\_config\_json, if that is your preferred method of specifying the configuration, follows below.

```
const char *p_config_json ="{"
    "\ndi\: {"
        "\vendor\: {"
            "\name\: \CoolCo, inc.\,"
            "\id\: \00000000-0000-0000-0000-000000000000\"
        "}"
    "}"
"}";
```



The ability to specify per connection settings for any sender, finder and receiver is a very powerful ability since it allows every connection to be completely customized per use, allowing for instance different NICs to be used for different connection types, multicast to be manually specified by connection, different groups and much more.

{% hint style="info" %}
The full details of the configuration file are provided in the manual under the section on _<mark style="color:red;">Performance and Implementation Details</mark>_.
{% endhint %}
