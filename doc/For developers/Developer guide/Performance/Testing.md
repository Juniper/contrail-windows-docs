Contrail Windows Performance Testing
====================================


## Test environment

Performance is evaluated on 2 VMware virtual machines.
Virtual machines should have the following specs:

- VMware Virtual Machine version 13,
- CPU: 2 vCPU,
- RAM: 4 GB,
- Storage: 50 GB HDD (VMware Paravirtual SCSI),
- NIC1: 1Gb (Emulated E1000E),
- NIC2: 10Gb (VMXNET3).

Both virtual machines should be spawned on the same ESXi.
NIC1 is used as a MANAGEMENT plane adapter and can be attached to any vSwitch.
NIC2 is used as a CONTROL and DATA adapter and it should be attached to vSS created solely
for performance testing.
This vSS should not have any physical NICs attached.


### Bare Hyper-V

Setup

```powershell
Install-WindowsFeature Containers
Install-WindowsFeature NET-Framework-Features
Install-WindowsFeature Hyper-V -IncludeManagementTools
Restart-Computer
Install-Module DockerMsftProvider -Force
Install-Package Docker -ProviderName DockerMsftProvider -Force -RequiredVersion 17.06.2-ee-16
Restart-Computer
docker image pull microsoft/windowsservercore:ltsc2016
```


### Contrail Windows

**TODO**


## Test cases

TCP traffic (with `NTttcp`):

- 2 containers, 1 network, 1 node
- 2 containers, 2 networks, 1 node
- 2 containers, 1 network, 2 nodes
- 2 containers, 2 networks, 2 nodes

What to measure, for TCP, on both sender and received side:

- Throughput(MB/s)
- Retransmits
- Errors
- Avg. CPU %

UDP traffic:

- 2 containers, 1 network, 1 node
- 2 containers, 2 networks, 1 node
- 2 containers, 1 network, 2 nodes
- 2 containers, 2 networks, 2 nodes


## Test method

### NTttcp - TCP

NTttcp - https://gallery.technet.microsoft.com/NTttcp-Version-528-Now-f8b12769

```
# On receiver node
.\NTttcp.exe -r -m 2,*,172.16.0.12 -rb 2M -a 16 -t 15

# On sender node
.\NTttcp.exe -s -m 2,*,172.16.0.12 -l 128k -a 2 -t 15
```

Test results:

```
# MB/s, from receiver
1352.541
1478.696
1380.415
1474.988
1459.776
1372.184
1434.986
1306.985
548.483
1435.494
```


### NTttcp - UDP

**TODO**
