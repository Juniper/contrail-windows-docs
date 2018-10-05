# Known issues and workarounds

## vRouter extension

1. After finished deployment vRouter extension sometimes enter state when it is enabled but not running. To check run:

        get-vmswitch | get-vmswitchextension -name "vRouter forwarding extension" | select enabled,running

    Workaround requires uninstalling and installing vRouter (possibly few times).

        msiexec /x vRouter.msi
        msiexec /i vRouter.msi
