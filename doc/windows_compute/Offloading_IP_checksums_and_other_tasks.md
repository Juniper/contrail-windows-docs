# Offloading IP Checksums and other tasks

## Introduction

> Note: Offloading basically means "let somebody else do it".

There are [various tasks][ms-task-offload] related to the IP stack that can be offloaded,
such as computing checksums or segmenting large TCP packets.

It is beneficial to perform these offloads as late as possible. In the best case,
the offloading is performed in the network card hardware, so eg. no CPU cycles
are wasted on computing the TCP checksum.

Sometimes the offloading is not supported by the hardware, but still can be performed in the software.
This is best described on an example — the LSO (Large (TCP) Send Offload) means
that for the most time spent in the kernel the TCP packet can be stored as one single [`NET_BUFFER`][nbl],
bigger than a single Ethernet frame. That means that all operations like filtering, routing, etc.
can operate efficiently (because they look at a single packet instead of many)
and the actual segmenting will be performed just before passing the buffer
to the hardware. In the case of container-to-container connection, the actual segmentation
can even never happen! As they say, the fastest code is the code that never runs.

## Offloading in Hyper-V Virtual Switches

> Note: Refer to [the blog post about VMSwitches and offloading][vmswitch-offloads] for more information.

Usually, the OS Networking stack asks the network device about types
of offloading it supports and offloads only the tasks that the hardware
is capable of offloading. With VMSwitch it's not that simple, because
it's hard to propagate the hardware capabilities to all the ports
in VMSwitch (especially because when sending a packet into VMSwitch,
we don't know where will it end up: a container, a virtual machine,
a real network card?).

To sidestep this uneven-offload-support issue, VMSwitch **always**
advertises on all its ports that most offloads are supported.
Therefore, the vRouter must be prepared to handle NBLs
with these offloads requested.

The offloads that are always supported are:

*   IP, TCP, UDP send checksum offload.
*   LSO (Large Send Offload), which is basically TCP Segmentation offload.
*   Receive checksum offload (VMSwitch with the help of hardware or by
    itself will set appropriate flags that mark the validity of checksums
    in _incoming_ packets).

So yes, we must handle all these offload requests.
But this relationship is symmetric — when sending packets
we can rely on support for these offloads on all vPorts!

Other offloads than listed above are not always advertised as supported.

[section-epo]: #encapsulated-packet-offload
## Encapsulated Packet Offload

The most interesting type of offload which is _not_ always
advertised as supported by VMSwitch is the [Encapsulated Packet Offload][ms-nvgre].
In theory it enables to perform all the standard offload tasks
(like computing checksums and Large Send Offload) on the inner packet,
despite it being wrapped in a tunnel. It also allows to simultaneously
offload the checksums of the tunneling headers themselves.

Unfortunately it requires support from the network device or its driver.
To check for support and enable the feature you can use the following commandlests:

```powershell
Get-NetAdapterEncapsulatedPacketTaskOffload
Enable-NetAdapterEncapsulatedPacketTaskOffload
```

The drivers used in our test environments (vmxnet3) do not seem to support this feature
(at least, we haven't managed to enable it using these PowerShell commandlets).
If we try to use it anyway by setting appropriate NBL Info, the packet gets rejected.

### Only NVGRE?

The documentation on [Encapsulated Packet Offload][ms-nvgre] page suggests
that this feature can only be used with NVGRE tunneling.
It never excplicitely states it cannot be used with other kinds of tunneling though.

On the other hand, the fields of [the union that describes the parameters for EPO][supplemental-struct]
allow specifying arbitrary offsets between outer and inner headers,
suggesting that this feature can be used with different tunneling protocols.

In general, this feature is a little bit underdocumented,
so it's hard to get a definite answer whether other tunnellings than NVGRE are supported.

<small>
    Note: There's a small chance that the reason our experiments with Encapsulated
    Packet Offload had failed was not the lack of support from the driver,
    but rather the fact we were trying to use MPLSoGRE, not NVGRE.
    This seems very unlikely though.
</small>

## Handling Net Buffer List Info

All the information about offloading requests/responses is stored
in [various Net Buffer List Info unions][nbl-info-enum].
There are a few important things to note about these:

1.  Many of these unions have `Transmit` and `Receive` variants.
    A care should be taken when reading these, as only
    one variant is active at once — packets incoming from
    vHost or containers' ports are using `Transmit` variants
    and packets incoming from external interface are using `Receive`
    variants. Reading from inactive variant will result in reading garbage.

2.  Net Buffer List Info is not copied when using `NdisAllocateCloneNetBufferList`.

3.  When performing offload tasks in software, it's important to
    disable offloading requests by zeroing these fields.
    Not doing that will result in unexpected results.

    It's also important to remember that some offload tasks
    require writing additional info on Complete,
    like number of transfered bytes in LSO.

4.  When the packet is wrapped in a tunnel, the fields
    refer to the outer packet. A care should be taken
    to check whether these fields are still correct.

## Interaction with dp-core

The **LSO** (Large Send Offload) feature on Windows is really similar to
GSO (Generic Send Offload) on Linux and dp-core.
To support it, we only need to provide LSO-requested MSS in a getter.

`vr_packet` has some flags for GRO (Generic Receive Offload).
Analogous Windows feature is **LRO** (Large Receive Offload).
How to handle it in dp-core and whether it is necessary
wasn't investigated yet.

When the checksum offload was requested, packets will have
the [**partial checksum**][tcp-partial-csum] already computed
and stored in the TCP/UDP checksum field. When dp-core tries to
incrementally update checksums, it needs to know whether the checksum
fields contains full or partial checksum, in order to correctly
update the field. The `VR_FLAG_CSUM_PARTIAL` flag must be set accordingly.

## Possible performance improvements

1.  The offloading request for TCP checksum contains the offset
    of TCP header — that means it should be possible to leverage
    the TCP checksum offload also on tunnelled packets (by providing
    adjusted offset to inner TCP header, which includes the outer headers length).

2.  Checksums of outer headers could be offloaded unconditionally.

3.  Currently, we do TCP segmentation of tunnelled packets manually.
    It should be possible to use [Encapsulated Packet Offload][section-epo]
    on a supported hardware.

## References

1. [TCP/IP Task Offload (docs.microsoft.com)][ms-task-offload]
2. [Hyper-V Virtual Switch Performance — Offloads][vmswitch-offloads]
3. [Network Virtualization using Generic Routing Encapsulation (NVGRE) Task Offload][ms-nvgre]
4. [TCP Checksum Calculation and the TCP "Pseudo Header"][tcp-partial-csum]


[ms-task-offload]: https://docs.microsoft.com/en-us/windows-hardware/drivers/network/task-offload
[vmswitch-offloads]: https://blogs.msdn.microsoft.com/altr/2014/09/12/hyper-v-virtual-switch-performance-offloads/
[ms-nvgre]: https://docs.microsoft.com/en-gb/windows-hardware/drivers/network/network-virtualization-using-generic-routing-encapsulation--nvgre--task-offload

[supplemental-struct]: https://technet.microsoft.com/en-us/windows/jj991957(v=vs.100)
[nbl-info-enum]: https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ndis/ne-ndis-_ndis_net_buffer_list_info
[tcp-partial-csum]: http://www.tcpipguide.com/free/t_TCPChecksumCalculationandtheTCPPseudoHeader-2.htm

[nbl]: NBL_processing.md
