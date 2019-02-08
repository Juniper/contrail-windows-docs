## How to run docker daemon in debug mode

Just run dockerd.exe -D in console or add "debug": true to config and restart daemon.

## What exactly happens during container creation

For windows there is only one [graph driver](https://blog.mobyproject.org/where-are-containerds-graph-drivers-145fc9b7255) which is called windowsfilter. It is basically wrapper for hcsshim requests which is wrapper for HCS requests.

Command discussed: docker run -id --name example microsoft/windowsservercore powershell

Calling POST /v1.39/containers/create?name=example

Container creation is pretty straightforward:

1. Function ContainerCreate from container.go is called with configuration parameters. Most relevant function in chain is create function from daemon\graphdriver\windows\windows.go
1. Request for layer mount path of parent layers, in this case windowsservercore image, is sent to hcsshim (hcsshim::GetLayerMountPath). This wiil be just path to directoy as they are read-only.
1. Resuest for creation of scratch layer (in other words read-write layer, which is for some unknown reason called sandbox layer in hscshim) is sent to hcsshim. Note that id of this layer will and should be an id of a container. (hcsshim::CreateScratchLayer)

Calling POST /v1.39/containers/f7288bb4dc15d08dfe6cddb1830b06d88e1384967b6cda19165c5019a9575d3d/wait?condition=next-exit
This just means "wait for the next time the state changes to a non-running state", which should happen immediately after creation is completed.

This is it for creation part, now request for starting container is sent. During the starting phase if networking is provided it will also be created.
Calling POST /v1.39/containers/f7288bb4dc15d08dfe6cddb1830b06d88e1384967b6cda19165c5019a9575d3d/start

1. At the beginning container base filesystem is mounted. This is done by calling Get() function from windowsfilter driver on newly created RW layer. hcsshim::ActivateLayer is the call that mounts RW layer (wiil appear as a volume on the host) and hscshim::PrepareLayer intializes the filesystem filter for that container.

Next part is network configuration:
 
## What exactly happens during attaching container to network

### Compartments

Compartments are Windows network namespaces. To list comnpartments run Get-NetCompartment. there is one default compartment for OS which has CompartmentId = 1 and type Native. Container compartments will have CompartmentType Unspecified. To link compartment to a container look into CompartemntDesciption field, it will have format "\Container_(container_id)"

### Interafaces

To list container interfaces run  get-netipinterface -addressfamily Ipv4 -includeallcompartments | Where InterfaceAlias -Like "*Container NIC*"
To get interace from specific compartment check execute get-netipinterface -addressfamily Ipv4 -includeallcompartments -compartmentId id.

## Fun Facts

Docker on Windows doesn't support restoring containers when daemon dies

To see the id of image layer run docker image inspect imagename and look under GraphDriver key

To see layers just list C:\ProgramData\docker\windowsfilter\ directory.

To see layerchain for container see C:\ProgramData\docker\windowsfilter\continer_id\layerchain.json

Winddows doesn't behave well when mounted RW layer id isnt't the same as id of container

Volume path for a given mounted layer may change over time on Windows by design.

Windows Container Isolation (wcifs.sys) is the name of filesystem filter driver used for containers.
