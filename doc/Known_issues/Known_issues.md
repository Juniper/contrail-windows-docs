# Workarounds for known issues

Below is a list of known issues with workarounds.

---

### DNS doesn't work in my networks

For DNS to work in containers, DHCP has to be turned on in Contrail network via WebUI.
Please refer to this bug: https://bugs.launchpad.net/opencontrail/+bug/1535856

---

### I can't get IP fragmentation to work

On Windows Server 2016, NAT network on compute node must be disabled in order for IP packet fragmentation to work.

To do that, use the following commands:
```
Get-NetNat | Remove-NetNat                      # Remove existing NAT networks
Stop-Service winnat                             # Stop WinNAT service
Set-Service -StartupType Disabled -Name winnat  # Disable autostart of WinNAT upon reboot
```

If you alread have some HNS networks, you will probably need to remove them as well (**Warning** this will remove all
container networks present on the Compute node):

```
# Remove all container networks twice for good measure, because HNS sometimes fails
Get-ContainerNetwork | Remove-ContainerNetwork -ErrorAction SilentlyContinue -Force
Get-ContainerNetwork | Remove-ContainerNetwork -Force
```

You also need to edit Docker config file located at `C:\ProgramData\Docker\config\daemon.json` to contain
`"bridge": "none"` entry, like so:
```
{
    "bridge" : "none"
}
```

Otherwise, Docker will try to reenable WinNAT upon restart.

---

### Creating container failed because of timeout

Symptom: running `docker run ...` results in one of these errors:
```
hcsshim::PrepareLayer failed in Win32: This operation returned because the timeout period expired. (0x5b4)
```
or
```
 hcsshim::PrepareLayer failed in Win32: The parameter is incorrect. (0x57)
 ```

It happens when mounted layer takes too much time to appear in system. Workaround requires repeating failed action.

Please refer to [this](https://github.com/moby/moby/issues/27588) and [this issue](https://github.com/moby/moby/issues/36831) on github.

---

### When I run Get-Net* command inside a container, it hangs

Symptom: running `Get-NetAdapter`, `Get-NetIpInterface` or other `Get-Net*` commands inside containers hangs.

The workaround consists of restarting the container.

---

### I see that vRouter is enabled, but not running

After finished deployment of Windows Contrail compute nodes, sometimes vRouter enters a state when it is enabled
but not running. To check run:

```
Get-VMSwitch | Get-VMSwitchExtension -Name "vRouter forwarding extension" | Select Enabled, Running
```

Workaround requires uninstalling and installing vRouter (possibly few times).

```
msiexec /x vRouter.msi
msiexec /i vRouter.msi
```

---

### I have received a cryptic HNS error

Please refer to [HNS error bestiary](./HNS_error_bestiary.md).

### I get authentication error "CredSSP encryption Oracle remediation" when trying to RDP into a Windows machine from another Windows machine

![CredSSP encryption Oracle remediation](CredSSPError.png)

If you see the above error, please refer to [CredSSP encryption Oracle remediation bestiary](CredSSP_error_bestiary.md)

### I cannot ping container on Windows from VM on Linux

This issue is related to checksum handling on the Linux Tungsten Fabric compute node.
If checksum offload is enabled, Linux transmits packets with inner IP checksums not calculated correctly.
When Windows compute nodes receive such encapsulated packets, after vRouter decapsulates them, receiving container's TCP/IP stack drops the packets due to incorrect checksum.

To work around this behaviour, you can disable checksum offloading on Linux compute node.
Assuming `ens224` is the data plane interface, please run the following as root on Linux compute node:

    ethtool -K ens224 rx off
    ethtool -K ens224 tx off

You can track progress on that issue on Launchpad: [https://bugs.launchpad.net/opencontrail/+bug/1806680](https://bugs.launchpad.net/opencontrail/+bug/1806680)

### Container reports an autoconfiguration IP address

This bug happens when container is configured with autoconfiguration address instead of one provided by Docker.
Genesis of this bug is unknown.

For recovery do the following:

- Check if compute node was deployed properly (Refer to 1-5 steps from [Troubleshooting](./Troubleshooting.md))
- Recreate the container

If it does not work, refer to workarounds for `Waiting for IP on interface Ethernet1 failed` bug.

[Launchpad link related to the issue](https://bugs.launchpad.net/opencontrail/+bug/1794263)

### Waiting for IP on interface Ethernet1 failed

This bug happens when there is no reassignment of IP address from vhost (`vEthernet`) to physical interface (e.g. `Ethernet1`) after deleting vRouter extension.
This is a long occuring hard to catch and hard to fix bug.

There are two workarounds:

- Invasive but easier:
    - Use cleanup script (Step 10 in [Troubleshooting](./Troubleshooting.md)).
    - Redeploy compute node with e.g. contrail-ansible-deployer.
- Less invasive but harder (this method sometimes needs to be repeated multiple times):
    - Delete containers and docker networks
    - Stop Docker and Contrail services

            Stop-Service contrail*
            Stop-Service docker

    - Reinstall vRouter extension

            # Enter folder where vRouter's msi is located
            # Uninstall vRouter extension
            msiexec /x vRouter.msi
            # Remove container networks
            Get-ContainerNetwork | Remove-ContainerNetwork -ErrorAction SilentlyContinue -Force
            Get-ContainerNetwork | Remove-ContainerNetwork -Force
            # Install vRouter extension
            msiexec /i vRouter.msi

    - Start Docker and Contrail services

            Start-Service docker
            Start-Service contrail*


[Launchpad link related to the issue](https://bugs.launchpad.net/opencontrail/+bug/1794262)
