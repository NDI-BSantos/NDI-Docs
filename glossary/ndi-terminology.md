# NDI Terminology

### Bridge

An NDI Tool that allows you to connect and share NDI streams between remote NDI infrastructures across a WAN.

### Discovery Server

A software application that stores and shares the discoverability information of NDI senders. It is an alternative to the default discovery and registration method in NDI based on mDNS and does not require multicast.

### Discovery Service

The service provided by NDI Discovery Server.

### Embedded Bridge

A version of NDI bridge capable of connecting an NDI stream from an embedded NDI device to an NDI Bridge host via a WAN

### H.264/H.265

Two specific video codecs which are used for the NDI HX formats

### Input/Output

A sender or receiver, typically audio and/or video, that can be connected to another device, potentially something other than NDI but able to send or receive NDI streams. A single NDI sender/receiver can support multiple inputs and outputs and each of which could support audio, video, metadata or a combination. This is part of the negotiation between receiver and sender.&#x20;

{% hint style="warning" %}
_The terms input and output should be avoided in an NDI context and the terms Sender and Receiver should be used instead as these are NDI standard terms_
{% endhint %}

### NDI Finder

A software component in the SDK used to find NDI senders and/or other devices on a network.

### NDI Genlock

NDI genlock supports using an NDI signal as a timing reference for software applications which otherwise lack an internal video timebase

### NDI Groups

A way to organize NDI devices to filter NDI senders using NDI discovery service for easier management and discovery

### NDI High Bandwidth

In an NDI system usually refers to a Speed HQ based NDI stream. Potentially could refer to the NDI sender to sending a proxy or a full stream.

### NDI HX

Encompasses all previous versions of HX.

### NDI HX3

An NDI premium product and one of our high efficiency formats.

### NDI Tools Router

In the NDI context a router is an NDI SDK instance that can map NDI streams to alternate senders.

### NDI Stream

An NDI connection between two or more hosts that negotiate the start of data transmission. The negotiation primarily concerns the transmission protocol (TCP, UDP, mTCP, or RUDP), codec and the type of stream, either video, audio, metadata or a combination of the three.

### Receiver

A device or application (NDI SDK instance) able to receive an NDI stream.

### Sender

A device, application or NDI SDK instance able to send an NDI stream.

### Speed HQ

The original proprietary codec used with NDI.

