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