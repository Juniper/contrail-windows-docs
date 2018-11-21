On Linux, a special vRouter Kernel Module is enabled to perform the heavy-duty, high-performance datapath functionality of Contrail. The Forwarding Extension is a Windows equivalent. It's implemented as a forwarding extension of Window's Hyper-V Extensible Switch.

Note: Even though the name "forwarding extension" refers to the type of extension of Hyper-V Switch, the team calls the equivalent of Linux kernel module as "the Forwarding Extension".

DP-core is a cross platform module that is responsible for datapath logic. Even though its design is cross platform (DP-core exposes a set of function pointers, that need to be implemented differently on different operating systems, for example memory allocation, packet inspection procedures, timer functions etc.), its implementation uses many gcc-specific built ins, which require porting over to Windows.

Note: In theory, kernel extension compiled with GCC might work on Windows, but Visual Studio compiler is recommended.

The Extension implements an NDIS (Network Driver Interface Specification) API. The NDIS API is event driven, and is realized in terms of callback functions.

There is two-way interaction between DP-core and NDIS:

* Since the Extension is event driven, whenever an event occurs, a specific NDIS callback is ran - for example, if a packet is received, or a new port is connected. In those cases, DP-core's functions will need to be used inside the callback to enact vRouter logic and reach a forwarding decision.
* However, DP-core sometimes needs additional information to reach those decisions. It uses the "callbacks" in form of aforementioned function pointers to OS-specific implementations of those functionalities.

Note: In other words, there are two "kinds" of callbacks: "NDIS callbacks" and "DP-core callbacks". NDIS callbacks are a consequence of event-driven architecture of Hyper-V Extensions, while DP-core callbacks are actually just platform specific "dependency injections" or "C interfaces". 
The team usually refers to those "DP-core callbacks" as just callbacks. NDIS callbacks are usually referred to as "Extension/NDIS API implementation".

Note: There is a convenience library provided by Microsoft around the raw NDIS API. The library is referred to as SxBase, but it's just 4 source files. They can be found here: https://github.com/Microsoft/Windows-driver-samples/tree/master/network/ndis/extension/base. SxBase was used in the project but due to licensing issues it was removed.

The Forwarding Extension does not directly communicate with Contrail Controller. Instead, vRouter Agent is responsible for creating all the objects and flows inside the Extension. 
