HNS operates by using two virtual switches:

* an external vswitch, which is connected directly to physical adapter. This adapter is called "vEthernet (HNSTransparent)", and the switch is called "Layered Etherenet0" depending on which physicial adapter it's connected to. If there is physical NIC teaming, then its name contains all the physical adapters separated by a coma, like "Layered Ethernet0,Ethernet1"
* an internal vswitch, for the purpose of NATing. The adapter that connects to the host is named "vEthernet (HNS Internal NIC)", and the switch is called "nat".

Those virtual switches are not created unless there is at least one HNS network using corresponding Mode: "transparent" or "nat". If the last network of corresponding Mode is removed, the vswitch is also removed. Creation/deletion of vswitch takes a couple seconds and disrupts network connectivity.

This dynamism of creating/deleting vswitches poses a problem, because we want vRouter Forwarding Extension to be persistent, no matter if there are many or no virtual networks. That's why, when CNM plugin initializes, it creates a "dummy" Root HNS Network, which makes HNS create an external vswitch "Layered?Ethernet*". Forwarding Extension is then enabled for that vswitch.
