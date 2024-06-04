# Getting Started with Example Blueprint Projects

{% hint style="info" %}
The **NDI IO Plugin** install folder contains an example Unreal project containing maps for common use cases like creating NDI output streams of the Unreal viewport and showing NDI input on a texture within an Unreal map.&#x20;
{% endhint %}

The example projects are **C++ Unreal Engine projects**, so you will need [**Visual Studio 2019**](https://visualstudio.microsoft.com/) installed on your system (the Community Edition can be downloaded for free on the Microsoft website). It is recommended that you copy the `NDIExamples` folder from the NDI SDK for Unreal installation directory to a folder of your choosing.&#x20;

**Double-click** on the `NDIExamples.uproject` file in the `NDIExamples` folder to open the project in Unreal Engine.&#x20;

{% hint style="info" %}
You can switch between Unreal Engine versions by right-clicking on the `NDIExamples.uproject` file and selecting the `Switch Unreal Engine version…` entry from the context menu.&#x20;
{% endhint %}

Afterward, you may load any example map from the `Maps` folder in the Unreal Content Browser.&#x20;

<figure><img src="../.gitbook/assets/image (56).png" alt=""><figcaption></figcaption></figure>

To open and experiment with NDI Blueprint setups, navigate to the `Game Controller Blueprints` in the `Blueprints/Controllers`”folder using the Unreal Content Browser.&#x20;

<figure><img src="../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

Some Blueprints require editing NDI stream names for a specific stream to be shown in the map (for example, in the `BP_SampleInputController`).&#x20;

In such cases, select the `Stream Name` Blueprints node, and edit the `Default Value` to reflect the input NDI stream you would like to show.&#x20;

<figure><img src="../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
Please consult the [next section](broken-reference) regarding using the NDI Blueprints nodes to set up such Blueprints from scratch.
{% endhint %}
