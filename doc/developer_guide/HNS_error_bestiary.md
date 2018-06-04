# HNS errors bestiary

HNS error messages ofter aren't descriptive enough, and the cleanup is quite hard.
The following document describes common cleanup techniques and error interpretations.

## HNS Decontamination procedures

Before working with HNS, one must realize the risks.
There are many ways in which HNS can stop working and affect other parts of the system.
For a developer, it is important to be able to recognize those, which can be recovered 
from. Below are some techniques which may help.

Always try cleaning using the lowest level procedure. The higher the level, the more 
potential damage the cleanup can cause.

### A word of caution

*Never* develop HNS on a baremetal or own laptop.

### Level Alpha decontamination procedure

1. First, stop the docker service.
```
Stop-Service docker
```
2. Remove all virtual switches, container networks and NAT
```
Get-ContainerNetwork | Remove-ContainerNetwork
Get-VMSwitch | Remove-VMSwitch
Get-NetNAT | Remove-NetNAT
```

3. Restart HNS and docker services.
```
Restart-Service hns
Restart-Service docker
```

### Level Beta decontamination procedure

1. Remove all container networks. Always do this because some vmswitches may not be removed properly if you just execute the next command.
```
Get-ContainerNetwork | Remove-ContainerNetwork
```

2. Manually remove `HNS.data` file. `net` command sometimes works better at restarting HNS than PowerShell's `Restart-Service`.

> Warning: this is not recommended by Microsoft, but they do it in their official cleanup script, so I guess it's not that bad.

```
net stop hns; 
del C:\programdata\Microsoft\Windows\HNS\HNS.data; 
net start hns;
```

### Level Gamma decontamination procedure

Use official Microsoft script to cleanup. It will also cleanup some registry entries and do much more.

https://github.com/MicrosoftDocs/Virtualization-Documentation/tree/live/windows-server-container-tools/CleanupContainerHostNetworking

### Level [REDACTED] decontamination procedure

Everything is lost. All we can do wipe all devices and hope that the contamination won't spread to other hosts.

> Warning: this might BSOD.

```
// perform as single command because you will lose connectivity during netcfg -D
netcfg -D; Restart-Computer -force
```

# HNS error specimen list

The following chapter contains a list of HNS errors encountered throughout development, along with reproduction methods and natural habitat.

## 1. HNS Unspecified Error

### Natural habitat:

1.  When attempting to create a transparent HNS network
    * when creation of another network or VMSwitch is already in progress
      (eg. we've just started Docker service and it tries to create the NAT network).

    We can work around this bug by retrying after a few seconds.

### Reproduction:

Try to create multiple HNS networks in a loop simultaneously with multiple processes.
We suspect that this error occurs during a high load.

## 2. HNS Invalid Parameter

### Natural habitat:

TODO

### Reproduction:

TODO

## 3. HNS Element not found

### Natural habitat:

1. [Hypothesis] When attempting to create a transparent docker network
    * when no other transparent docker network exists and
    * Ethernet adapter to be used by the transparent networks has no IP address or it's invalid.

### Reproduction:

See https://github.com/Microsoft/hcsshim/issues/95

## 4. HNS failed with error : {Object Exists} An attempt was made to create an object and the object name already exists

### Natural habitat:

This error probably happens when docker tries to create NAT network, but HNS left over some trash after last NAT network.

Cleanup everything as explained in the Decontamination Procedures chapter. If the problem still persists, just create a random NAT network:
```
New-ContainerNetwork foo
```

### Reproduction:

TODO
