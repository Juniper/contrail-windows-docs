Note: Hyper-V Extensible Switch is referred to throughout this document as "Hv Switch", "Hyper-V Switch", "virtual switch" or just "vswitch".

Windows Server 2012 introduced Hyper-V Extensible Switch. It’s a virtual switch that runs in OS running in Hyper-V’s parent partition. It’s mainly used for forwarding packets flowing in and out of child partitions (VMs and containers).

Three types of switches can be instantiated:

* private - only interfaces connected to the vSwitch can communicate
* internal - like private, but the host (hypervisor) is also connected
* external - like internal, but external network adapter is also connected. Usually, this is the physical adapter. Multiple external switches may exist at any given time.

Hyper-V Extensible Switch is "Extensible", which means it can be extended using third party extensions (plugins).

Multiple extensions can be enabled at the same time. Hyper-V Switch's plugin architecture resembles a stack. The order in which ingress and egress packets will be processed by a specific extension depends on their position in the filter stack.

There are three types of extensions:

* capture - which can inspect packets, and, for example, log them somewhere
* filter - like capture, but has the ability to drop packets
* forwarding - like filter, but has the ability to forward and modify the packets.

Multiple instances of capture and filter extensions can be enabled in arbitrary locations in the extensions stack, but only one forwarding extension can be enabled at a time. It must be located at the bottom of the stack as well, which means that it is the last to process ingress packets, but the first to process egress packets.
