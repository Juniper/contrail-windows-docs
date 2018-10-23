# Unsupported features

## Platform

* Nano Server running as Compute Node is currently not supported, as it lacks the ability to install Hyper-V networking extensions.
* Windows 10 is not supported due to it having a different version of Docker (Community, instead of Enterprise).

## Networking

* On Windows Server 2016, NAT network on compute node must be disabled in order for IP packet fragmentation to work.
* IPv6 in overlay is not supported.

## Containers

* Hyper-V and Linux containers are not supported. Only Windows Server containers are supported, 
due to differences in network handling with Hyper-V Containers.

## Container networking

* Multiple endpoints (interfaces) per container are not supported.
* Multiple IPs/networks per container endpoint (interface) are not supported.
* Multiple config nodes not supported in CNM plugin.
* `docker network inspect` on Contrail network does not show correct IPAM subnet, if subnet was not specified on creation.
    * Please refer to this bug: https://bugs.launchpad.net/opencontrail/+bug/1789237

## Authentication

* Only Keystone v2 API is supported.
