
# What exactly happens during container creation

For Windows there is only one [graph driver](https://blog.mobyproject.org/where-are-containerds-graph-drivers-145fc9b7255) which is called windowsfilter. It is basically wrapper for hcsshim requests which is wrapper for HCS requests.

Let's follow calls from this command:
`docker run -id --network examplenetwork --name example microsoft/windowsservercore powershell`.

## Container creation

1. Calling POST `/v1.39/containers/create?name=example` to create a container.

    1. Function ContainerCreate from container.go is called with configuration parameters. Most relevant function in chain is create function from `daemon\graphdriver\windows\windows.go`.
    1. Request for layer mount path of parent layers, in this case windowsservercore image, is sent to hcsshim (`hcsshim::GetLayerMountPath`). This is just path to directory as image layers are read-only. To read about layers see [here](https://docs.docker.com/storage/storagedriver/) and [here](https://medium.com/@jessgreb01/digging-into-docker-layers-c22f948ed612).
    1. Request for creation of scratch layer (in other words read-write layer, which is for some unknown reason called sandbox layer in hcsshim) is sent to hcsshim. Note that id of this layer will be an id of a container (`hcsshim::CreateScratchLayer`).

1. Calling POST `/v1.39/containers/container_id/wait?condition=next-exit`.

    This just means "wait for the next time the state changes to a non-running state", which should happen immediately after creation is completed.

1. Calling POST `/v1.39/containers/container_id/start`.
    During the starting phase, if definition of networking stack is provided, it will also be created (as described in "Network configuration" section).

    At the beginning container base filesystem is mounted. This is done by calling `Get()` function from windowsfilter driver on newly created RW layer. `hcsshim::ActivateLayer` is the call that mounts RW layer (wiil appear as a volume on the host) and `hscshim::PrepareLayer` intializes the filesystem filter for that container.

Next part is network configuration:
See definitions for docker terminology [here](https://github.com/docker/libnetwork/blob/master/docs/design.md).

Next actions happening once container is created are described below.

## Compartments

Compartments are Windows network namespaces, that are - from user perspective - read-only.

To list compartments run `Get-NetCompartment`. There is one default compartment for OS which has `CompartmentId = 1` and type `Native`. Container compartments will have `CompartmentType` `Unspecified`.

To link compartment to a container look into `CompartmentDescription` field. It will have format `"\Container_(container_id)"`.

To check routes in the compartment run `Get-NetRoute -CompartmentId comapartmentId`.

## Interfaces

To list container interfaces run `Get-NetIpInterface -AddressFamily IPv4 -IncludeAllCompartments | Where InterfaceAlias -Like "*Container NIC*"`.

To get interace from specific compartment execute `Get-NetIpInterface -AddressFamily IPv4 -IncludeAllCompartments -CompartmentId id`.

## Network configuration

Once container is created (as described in "Contrainer creation" section), the following actions are executed:

1. Firstly network endpoint is created. In case of Contrail Windows project the call is sent ot contrail-cnm-plugin that subsequently calls `hscshim::CreateEndpoint` call. At his point endpoint extist only as a virtual resource in HNS.
2. Now, during `Join()` operation, network endpoint should be connected to sanbox but as network compartment API is not open and contrail-cnm-plugin cannot do it. It'll be done during creation of the container by hcsshim.

## Container creation in HCS

Last part is creation of the container in HCS:

1. `hcsshim::CreateComputeSystem` creates container. It seems that it also creates its vNIC, network compartment and vswitch port for it as they become visible only after this command is run. However, it requires further investigation to be certain.
1. `hcsshim::ComputeSystem::Start` and `hcsshim::ComputeSystem::CreateProces` start container and creates its process.

## Fun Facts

* To see HNS and HCS logs run `Get-WinEvent -Filterhashtable @{LogName="Microsoft-Windows-Hyper-V-Compute-Operational"}` but they just seem to have basic messages and timestamps.

* Docker on Windows doesn't support restoring containers when daemon dies.

* To see the id of image layer run `docker image inspect imagename` and look under `GraphDriver` key.

* To see layers just list `C:\ProgramData\docker\windowsfilter\` directory.

* To see layerchain for container see `C:\ProgramData\docker\windowsfilter\continer_id\layerchain.json`.

* Volume path for a given mounted layer may change over time on Windows by design.

* Windows Container Isolation (wcifs.sys) is the name of filesystem filter driver used for containers.
