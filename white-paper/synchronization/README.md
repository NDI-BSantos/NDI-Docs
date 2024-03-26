---
layout:
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Synchronization

NDI transmitters or receivers do not need any synchronization method to be connected and work. **An NDI infrastructure can perfectly work “sync free”** without the complexity and cost of network infrastructure that supports a synchronization layer. However, with NDI, software developers and hardware manufacturers have several ways to approach synchronization:

### NDI Advanced SDK – Genlock API

When using NDI to send video onto the network, it is very common that one uses the computer clock to know what speed to send frames at; at this point, the SDK will use the system clock to pace the sending of frames for you.

It is common in video systems that users wish to make sure that all your NDI sources are synchronized together so that they all send video at the same rate and time. While it is tempting to solve this by having a very high precision “reference clock” (e.g., PTP), this often works very well on local networks but does not easily extend to systems that are remote from each other (e.g., your local network and in the WAN).

The NDI SDK now allows you to easily clock any number of video sources on the network to match a centralized clock and even interface those with external video clocks like local SDI sources commonly used as a genlock signal.

**The NDI genlock allows one to create a “genlock clock” attached to any NDI sender on the network.** That genlock clock can then be used to correctly time all senders on the network so that they are correctly timed with the NDI sender. By sharing this NDI source into the cloud, you can ensure that you have full genlock support that spans both on-premises, remote networks, and in-cloud connections. By driving an NDI source using an SDI (or PTP, 2110, HDMI) converter, it is even simple to genlock your entire NDI network to a physical genlock signal.

For an NDI source to correctly operate as an NDI Genlock, it is essential to bear in mind a couple of key ingredients as outlined below:

* It is strongly recommended that the NDI source is a stream from NDI version 5, which has been significantly improved to support Genlock capabilities. It is possible that some NDI streams from previous versions are not fully compliant with how NDI genlock operates.
* Considering that the default NDI is unicast, it is essential that the source has enough network bandwidth to deliver a signal to all the NDI instances that need to be genlocked. Configuring this source for multicast might also help, although multicast often is complex to full support.
* NDI genlock is very robust and supports correct cross-frame-rate locking. For instance, a sender might be 30Hz, and you are genlocking a 60Hz signal to it. This is, however, not a recommended workflow where it can be avoided.
* Some NDI sources like Test Pattern Generator and NDI Screen Capture do not always send a regular stream of frames. They do this to save network bandwidth and CPU time. Sources such as these cannot be used as a basis for genlock.
* If the genlock clock cannot correctly genlock to an NDI sender, it will fall back to using the system clock and so can continue to work reasonably.
* Since there is some (low) overhead associated with each genlock instance, it is recommended only to have one for each source that you wish to lock, too.

### NDI Advanced SDK – Timestamp, AV Sync API

NDI transmitters and receivers utilize the system clock as a point of reference. This system clock can be aligned with robust sources like NTP or PTP. The timing details from these sources are integrated into the NDI Stream through timestamps embedded in each video and audio frame (with audio frames being smaller than video frames).

Subsequently, the receiver can realign audio and video sources using this timestamp information. This method provides an ingenious approach to achieving synchronization, making it an ideal solution for remote or cloud-based production scenarios.
