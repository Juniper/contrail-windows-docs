# Structure of vRouter code on Windows

This document describes general structure of vRouter code and implementation
details of the most important features including dp-core callbacks, driver
lifecycle, packet flow, communication channels and structures abstractions.


## vRouter structure overview

The vRouter module on Windows is implemented as NDIS modifying filter driver:
the Hyper-V Extensible Switch forwarding extension
(see [NDIS types overview](../Supplementary knowledge/NDIS_driver_types_overview)
for more details on NDIS driver types).

The implementation consists of:

* OS-independent, generic `dp-core` directory, where the main logic of packet
processing is placed,
* header files in `include` directory,
* Windows specific code, including dp-core callback in `windows` directory;
All OS-dependent callcacks required by dp-core are can be found in
`vr_host.c` and `vr_host_interface.c` files,
* userspace applications used to read and modify the state of vRouter
placed in `utils` directory,
* unit tests that can be run in userspace, in `test`.


## Driver lifecycle


### Loading the driver

When the driver implementing vRouter logic is loaded, the first function
called by the system is a
[`DriverEntry`](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/wdm/nc-wdm-driver_initialize)
routine, which is responsible for two main things:

* setting up the pointer to
[`DriverUnload`](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/wdm/nc-wdm-driver_unload)
routine which will be called when the driver is unloaded. For more information,
see [Unloading the driver](#unloading-the-driver),

* calling the
[`NdisFRegisterFilterDriver`](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ndis/nf-ndis-ndisfregisterfilterdriver)
function with a pointer to structure containing driver characteristic which
consists of driver names, versions and, the most important, functions called
by the system on speciffic conditions.
vRouter implements the following sets of functions:

    * [`FilterAttach`](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ndis/nc-ndis-filter_attach),
    [`FilterDetach`](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ndis/nc-ndis-filter_detach),
    [`FilterPause`](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ndis/nc-ndis-filter_pause),
    [`FilterRestart`](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ndis/nc-ndis-filter_restart):
    see more in the [Driver states](#driver-states) section,

    * [`FilterSendNetBufferLists`](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ndis/nc-ndis-filter_send_net_buffer_lists),
    [`FilterSendNetBufferListsComplete`](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ndis/nc-ndis-filter_send_net_buffer_lists_complete):
    not related to the driver lifecycle, described more precisely in the
    [Packet flow](#packet-flow) section,

    * [`FilterOidRequest`](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ndis/nc-ndis-filter_oid_request),
    [`FilterOidRequestComplete`](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ndis/nc-ndis-filter_oid_request_complete),
    [`FilterCancelOidRequest`](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ndis/nc-ndis-filter_cancel_oid_request):
    see [Switch parameters and events](#switch-parameters-and-events) section,

    * [`FilterNetPnpEvent`](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/content/ndis/nc-ndis-filter_net_pnp_event):
    see [Plug and Play and Power Management events](#plug-and-play-and-power-management-events) section.


### Driver states

TODO: State graph, description of Attach, Detach, Pause, Restart.


### Switch parameters and events

TODO: OID


###  Plug and Play and Power Management events

TODO: PnP events


### Unloading the driver

TODO: DriverUnload


## Packet flow

TODO: Link to separate document (link to NBL description inside)


## Communication channels

TODO: ksync, pkt0, pipes, shmem


## Windows structures abstraction

TODO: WinPakcet, WinPacketRaw, VrPacket, ...
