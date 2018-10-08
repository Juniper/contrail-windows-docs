# Limitations, known issues and workarounds

## Workarounds

* On Windows Server 2016, NAT network on compute node must be disabled in order for IP packet fragmentation to work.
* Container must be restarted when `Get-NetAdapter`, `Get-NetIpInterface` or other `Get-Net*` command - invoked inside the container - hangs.
* After finished deployment vRouter extension sometimes enters state when it is enabled but not running. To check run:

        get-vmswitch | get-vmswitchextension -name "vRouter forwarding extension" | select enabled,running

    Workaround requires uninstalling and installing vRouter (possibly few times).

        msiexec /x vRouter.msi
        msiexec /i vRouter.msi

## Deployment

* Nano Server running as Compute Node is currently not supported, as it lacks the ability to install Hyper-V networking extensions.
* Windows 10 is not supported due to it having a different version of Docker (Community, instead of Enterprise).

## Container networking

* Currently only Windows Server containers are supported, due to differences in network handling with Hyper-V Containers.
* Multiple config nodes not supported in Docker Driver.
* Multiple endpoints per container are not supported.
* Multiple IPs/networks per container endpoint are not supported.
* `docker network inspect` on Contrail network does not show correct IPAM subnet, if subnet was not specified on creation
    * Please refer to this bug: https://bugs.launchpad.net/opencontrail/+bug/1789237
* IPv6 in overlay is not supported.
