---
hidden: true
---

# Formatting NDI Configuration file

### Introduction <a href="#toc178857523" id="toc178857523"></a>

JSON (JavaScript Object Notation) is a lightweight data-interchange format that is easy for humans to read and write. It is commonly used in NDI configurations to store settings. Understanding how to format a JSON file correctly is essential to ensuring your NDI system operates smoothly.

This guide will explain how to properly format your NDI configuration JSON file in simple terms.

### Basic Structure of a JSON File: <a href="#toc178857524" id="toc178857524"></a>

A JSON file is composed of key-value pairs. Each pair has a key (a label) and a value (the data or setting). The key and value are separated by a colon : and each pair is enclosed within curly braces {} to form an object.

```
Example:
{
"key": "value"
}
```

In NDI configuration files, keys represent specific settings, and values can be numbers, text (strings), or even other objects.

### Key Formatting Elements <a href="#toc178857525" id="toc178857525"></a>

**Curly Braces** {}: Curly braces define an object. Every JSON file starts and ends with curly braces. Inside these braces, you place all your settings.



```
Example:
{ 
    "ndi": { 
        "machinename": "", 
        "tcp": { 
            "recv": { 
                "enable": true 
            } 
        } 
    } 
} 
```

In this example, "ndi" is the main object, and within it are nested objects like "tcp" and "recv".

**Square Brackets** \[]: Square brackets define a list or array of items. If there are multiple items, they are placed inside square brackets, separated by commas.\


```
Example: 

{ 
    "adapters": { 
        "allowed": [] 	 
    } 
} 
```

In this case, the "allowed" setting is an array which is currently empty (no adapters are listed).

Here is an example of an array with more than one element:

```
{
    "adapters": { 
        "allowed": ["10.28.2.10", "10.28.2.11", "10.28.2.12"] 
    } 
} 
```

**Commas** ,: Commas separate multiple key-value pairs or items in an array. A common mistake is placing a comma after the last item, which is not allowed in JSON.

**Correct use of comma:**

```
{ 
    "networks": { 
        "ips": "", 
        "discovery": "192.168.25.250" 
    }, 
    "multicast": { 
        "send": {  
            "enable": false 
        } 
    } 
} 
```

The comma after `"discovery": "192.168.25.250"` is correct because more settings follow it.

**Incorrect use of comma:**

```
{ 
    "multicast": { 
        "send": {  
            "enable": false  
        } 
    }, 
} 
```

The comma after `"send": { "enable": false }` is incorrect because it is the last item in the object.

**Quotation Marks** "": Keys and string values in JSON must be wrapped in double quotation marks. This applies to both the setting's name (the key) and the text values.

```
Example: 
{ 
    "machinename": "" 
} 
```

**Nested Objects**: JSON allows for nested objects, meaning an object can contain another object. This is useful for organizing settings hierarchically.

```
Example: 
{ 
    "ndi": { 
        "tcp": { 
            "recv": { 
                "enable": true 
            } 
        } 
    } 
} 
```

In this example, the "tcp" object contains another object "recv", which contains the "enable" key.

**Whitespace in JSON:**

In JSON, parsers ignore whitespace (spaces, tabs, newlines), which means it does not affect the validity or meaning of the JSON data. You can format your JSON in many different ways without changing its functionality. The use of whitespace is purely for human readability.

Here are a few equivalent versions of the same JSON data, demonstrating different uses of whitespace.

```
Example 1: Indented with 4 spaces (readable format): 
{ 
    "ndi": { 
        "machinename": "NDI_Device_1", 
        "tcp": { 
            "recv": { 
                "enable": true 
            } 
        } 
    } 
} 

Example 2: Left justified (minimal indentation) 
{ 
  "ndi": { 
  "machinename": "NDI_Device_1", 
  "tcp": { 
  "recv": { 
  "enable": true 
  } 
  } 
  } 
} 

Example 3: No extra whitespace (compact format): 
{ "ndi": { "machinename": "NDI_Device_1", "tcp": { "recv": { "enable": true } } } }
```

**Comments in JSON**

One important rule is that comments are not allowed in JSON files. JSON was designed to be a lightweight data-interchange format, so it only supports data structures like objects, arrays, strings, numbers, and booleans—but not comments.

Unlike some programming languages or other data formats (such as XML or JavaScript), you cannot add comments to JSON files. Attempting to do so will cause JSON parsers to reject the file as invalid.

**Why No Comments?**

The lack of comments in JSON intentionally keeps the format simple and efficient. If you need to document or describe parts of a JSON configuration file, consider doing so in:

External documentation: Write explanations or instructions outside the JSON file.

Inline metadata: Include descriptive keys in the JSON structure itself, such as "description": "This key enables the receiver".

**Booleans in JSON**

In JSON, the values true and false represent boolean data types. A boolean can only have two values—true or false. These values are used to indicate conditions such as "enabled/disabled", "on/off", or "yes/no" in many JSON configurations.

It’s important to note that booleans are not strings, so they should not be enclosed in quotes.

```
Correct Usage, True/False without quotes: 
{ 
    "enable": true, 
    "active": false 
} 
```

Here, true and false are treated as boolean values.

```
Incorrect Usage, Quoted booleans: 
{ 
    "enable": "true", 
    "active": "false" 
} 
```

This would be incorrect because "true" and "false" would be interpreted as strings, not booleans.\


**Numbers in JSON**

In JSON, numbers are another basic data type and should be written without quotes. They can represent integers or floating-point numbers. Numbers are written directly unlike strings, which must be enclosed in double quotes.

```
Correct Usage, Numbers without quotes: 
{ 
    "codec": { 
        "shq": { 
            "quality": 100 
        } 
    } 
} 
```

In this example, the "quality" key has a value of 100, which is an integer and is correctly written without quotes.

```
Incorrect Usage, Quoted numbers (Incorrect): 
{ 
    "codec": { 
        "shq": { 
            "quality": "100" 
        } 
    } 
}
```

Here, "100" is treated as a string, which would be incorrect if the value is intended to be a number.

### Common Mistakes to Avoid <a href="#toc178857526" id="toc178857526"></a>

**No Commas After the Last Item**: Remember, the last key-value pair in an object should not have a comma. This is a common error that can cause your configuration to fail.

**Matching Braces**: Ensure that for every opening curly brace {, there is a closing curly brace }. Similarly, for every opening square bracket \[, there must be a corresponding closing bracket ].

**Use Double Quotes**: Always use double quotes " for both keys and string values. Single quotes ' are not allowed in JSON.

```
Example of a Correctly Formatted NDI JSON Configuration File
{ 
    "ndi": { 
        "machinename": "NDI_Device_1", 
        "tcp": { 
            "recv": { 
                "enable": true 
            } 
        }, 
        "rudp": { 
            "recv": { 
                "enable": true 
            } 
        }, 
        "groups": { 
            "send": "Public", 
            "recv": "Public" 
        }, 
        "networks": { 
            "ips": "", 
            "discovery": "192.168.25.250,10.28.2.250" 
        }, 
        "adapters": { 
            "allowed": ["192.168.25.11", "10.28.2.11"] 
        }, 
        "multicast": { 
            "send": { 
                "ttl": 1, 
                "enable": true, 
                "netmask": "255.255.0.0", 
                "netprefix": "239.255.0.0" 
            }, 
            "recv": { 
                "enable": true, 
                "netmask": "255.255.255.0", 
                "netprefix": "239.255.0.0" 
            } 
        } 
    } 
}
```



**Key Points in This Example:**

The file starts with an opening curly brace { and ends with a closing curly brace }.

Nested objects like "tcp", "rudp", and "multicast" are enclosed in their own curly braces.

Every key (e.g., "machinename", "tcp", "send") is in double quotes.

Commas are placed after each key-value pair, except for the last pair within an object.
