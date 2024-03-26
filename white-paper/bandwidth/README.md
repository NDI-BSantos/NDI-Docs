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

# Bandwidth

NDI operates most efficiently in a dedicated network with high bandwidth and high availability. This contrasts with unmanaged environments such as the public Internet or networks where video rides along with data without priority.

Gigabit (1000 Mbps[^1]) networks are essential in production workflows. A typical NDI stream consisting of 1080 60P video yields a data rate up to 150 Mbps per stream. This extremely efficient stream is designed to have very low latency and allows multiple streams to be stacked together on a single Gigabit network. Even so, a production environment may require more capacity based on a simultaneous number of NDI streams required.

**The following tables** are intended to calculate bandwidth needs based on video resolutions and frame rates. It should be noted, however, that NDI is not deterministic. The bandwidth needed for NDI should be based on the determination of the [average utilization required](#user-content-fn-2)[^2].

[^1]: Mbps (Megabits per second) is a unit of measurement for data transfer speed, with one megabit equal to one million bits. Network transmissions are commonly measured in Mbps.

[^2]: Bandwidth numbers are given as reference and are subject to change
