# NDI Proxy and bandwidth optimization

By default, an NDI Sender has the capability to generate two types of streams: a full-quality stream and a proxy stream. A receiver can readily activate the proxy stream. A simple way to visualize a proxy stream is by utilizing NDI Studio Monitor and enabling the Low Bandwidth mode within the Video settings menu.

The Proxy stream serves as a means for an application to enhance the efficiency of NDI distribution across the network. For instance, an NDI-based Video Mixer can leverage the proxy stream for sources that are intended for use in a preview monitor rather than an actual output. Then, when a source is transitioned to an output, the application can seamlessly switch from the proxy stream to the full bandwidth stream.

This approach empowers receivers to effectively manage a more significant number of NDI sources while utilizing less network bandwidth.
