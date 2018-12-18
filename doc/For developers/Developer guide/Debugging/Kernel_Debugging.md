# Kernel debugging over network

## Setup

### Target host setup

1. Open a PowerShell or cmd.exe with Administrative privileges.
2. Enable kernel debugging:

    ```
    bcdedit /debug on
    ```

3. Enable network kernel debugging:

    ```
    bcdedit /dbgsettings net hostip:w.x.y.z port:n key:1.2.3.4
    ```

    * w.x.y.z should be an IP address of the host where you will run a debugger,
    * n should be a port number, e.g. 50001,
    * key is a 256-bit authentication key (4 64-bit values); you can choose your own custom key.


4. Obtain busparams using following PowerShell one liner:

    ```
    Get-NetAdapterHardwareInfo -InterfaceDescription *Intel* | select Name, InterfaceDescription, DeviceType, Busnumber, Devicenumber, Functionnumber | FL
    ```

5. Configure busparams in bcdeit:

    ```
    bcdedit /set "{dbgsettings}" busparams b.d.f
    ```

    * b is bus number
    * d is device number
    * f is funtion number

6. Reboot target host after configuration is done.

### Debugger host setup

1. Open WinDbg Preview
2. Select `File > Start debugging > Attach to kernel > Net`
3. Provide the same port number and key as in bcdedit. Press OK.
4. Press `Break` (or use `Ctrl` + `Break`) to enter a debugging session

## Troubleshooting

## WinDbg: Module load completed but symbols could not be loaded for vRouter.sys

First of all, enable noisy symbol prompts and reload `vRouter.sys`

```
!sym noisy
.reload /f vRouter.sys
```

If you can see in the logs that:

```
...
DBGHELP: x:\build\debug\vrouter\extension\vrouter\vRouter.pdb - file not found
...
```

Modify sympath, so it contains a correct path. Assuming `vRouter.pdb` is located in `X:\build\debug\vrouter\extension` use the following command:

```
.sympath+ X:\build\debug\vrouter\extension
.reload /f vRouter.sys
```

After executing these commands, you should see the following in the logs:

```
0: kd> .reload /f vRouter.sys
...
BGHELP: x:\build\debug\vrouter\extension\vRouter.pdb cached to C:\Program Files (x86)\Windows Kits\10\Debuggers\x64\sym\vRouter.pdb\ED611F7B78B44C6D9339812199DE141B2\vRouter.pdb

DBGHELP: vRouter - private symbols & lines
         C:\Program Files (x86)\Windows Kits\10\Debuggers\x64\sym\vRouter.pdb\ED611F7B78B44C6D9339812199DE141B2\vRouter.pdb
SYMSRV:  BYINDEX: 0x5A2
         C:\Program Files (x86)\Windows Kits\10\Debuggers\x64\sym
         kdnic.pdb
         58AE3A2A99FB4866BA9265064652CBB71
SYMSRV:  PATH: C:\Program Files (x86)\Windows Kits\10\Debuggers\x64\sym\kdnic.pdb\58AE3A2A99FB4866BA9265064652CBB71\kdnic.pdb
SYMSRV:  RESULT: 0x00000000
DBGHELP: C:\Program Files (x86)\Windows Kits\10\Debuggers\x64\sym\kdnic.pdb\58AE3A2A99FB4866BA9265064652CBB71\kdnic.pdb cached to C:\Program Files (x86)\Windows Kits\10\Debuggers\x64\sym\kdnic.pdb\58AE3A2A99FB4866BA9265064652CBB71\kdnic.pdb

DBGHELP: kdnic - public symbols
         C:\Program Files (x86)\Windows Kits\10\Debuggers\x64\sym\kdnic.pdb\58AE3A2A99FB4866BA9265064652CBB71\kdnic.pdb
0: kd>
```

Verify that symbols are loaded by using:

```
0: kd> x /D /f vRouter!v*
 A B C D E F G H I J K L M N O P Q R S T U V W X Y Z

fffff80b`d14ded10 vRouter!vr_flow_nat (struct vr_flow_entry *, struct vr_packet *, struct vr_forwarding_md *)
fffff80b`d14e4420 vRouter!vhost_drv_add (struct vr_interface *, struct _vr_interface_req *)
...
```
