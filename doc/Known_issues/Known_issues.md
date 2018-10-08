# Workarounds for known issues

Below is a list of known issues with workarounds.

---

### IP fragmentation doesn't work

On Windows Server 2016, NAT network on compute node must be disabled in order for IP packet fragmentation to work.

---

### Get-Net* commands ran inside containers hang

Running `Get-NetAdapter`, `Get-NetIpInterface` or other `Get-Net*` commands inside containers hangs.

The workaround consists of restarting the container.

---

### vRouter is enabled, but not running

After finished deployment vRouter extension sometimes enters a state when it is enabled but not running. To check run:

    get-vmswitch | get-vmswitchextension -name "vRouter forwarding extension" | select enabled,running

Workaround requires uninstalling and installing vRouter (possibly few times).

    msiexec /x vRouter.msi
    msiexec /i vRouter.msi

---

### Various cryptic HNS errors

Please refer to [HNS error bestiary](./HNS_error_bestiary.md).
