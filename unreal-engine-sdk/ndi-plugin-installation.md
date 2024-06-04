# NDI Plugin Installation

The **NDI Runtime** is not included with this plugin. To work properly with [Unreal Engine](https://www.unrealengine.com/), the **NDI** **IO Plugin** requires the NDI 5.x Runtime to be installed on the host system. Please visit [**this page**](https://ndi.video/for-developers/ndi-sdk/) to download the latest SDK).&#x20;

The **NDI IO Plugin** install folder includes pre-built versions of the plugin. These can be installed in one of two locations: &#x20;

* Your Unreal Engine project&#x20;
* Your Unreal Engine install folder&#x20;

In either case, you must copy the `NDIIOPlugin` folder to the correct destination, as described in one of the following sections.

***

### Installing Your Unreal Engine Project

Use this install option to make the plugin available selectively (on a project-by-project basis).&#x20;

* Copy the `NDIIOPlugin` folder from the package matching your preferred Unreal Engine version. &#x20;
* Paste this folder into your Unreal Engine project's `Plugins` folder. (If your Unreal Engine Project does not have a “Plugins” folder yet, please create a folder with that name).&#x20;

After re-starting your Unreal Engine project, the **NDI IO Plugin** should appear in your Project’s `Plugins` settings as activated.

***

### Installing To the Unreal Engine Install Folder

Alternatively, use this install procedure to make the **NDI IO Plugin** available to all Unreal Engine projects on your system. &#x20;

* Copy the `NDIIOPlugin` folder from the package corresponding to your Unreal Engine version. &#x20;
* Paste that `NDIIOPlugin` folder into the `Plugins` folder of your Unreal Engine install (e.g., the Unreal Engine 5.3 “Plugins” folder is usually located at `C:\Program Files\Epic Games\UE_5.3\Engine`.&#x20;

With your project’s editor open, use the file menu item `Edit` -> `Plugins` to make sure that the **`NDI IO Plugin`** is enabled. Restart the editor as necessary. To learn about using the plugin within your application, read [Using the NDI Blueprints Nodes](broken-reference).

***

### Compile From Source

The plugin’s full source code is distributed within each version-specific `NDIIOPlugin` folder, as well as in the `NDIExamples` folder. Advanced users with [Microsoft Visual Studio](https://visualstudio.microsoft.com/%C2%B4) installed on their system can compile the plugin directly from the source code.
