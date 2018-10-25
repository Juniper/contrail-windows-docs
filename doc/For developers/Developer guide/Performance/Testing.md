Contrail Windows Performance Testing
====================================

This document describes assumptions and tools utilized to measure Contrail Windows performance during development.
It's divided into following sections:

1. Test environment
2. Test scenarios
3. Performance baseline setup


## 1. Test environment

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


## 2. Test scenarios

### TCP performance

TCP test scenario:

- 2 compute nodes (node A and node B);
- 1 Contrail network named `network1`;
- 1 container `sender` on node A, attached to `network1` network;
- 1 container `receiver` on node B, attached to `network1` network;
- `sender` and `receiver` exchange TCP segments using `NTttcp` tool;

```
+----------------------+                       +----------------------+
|                      |                       |                      |
|  +------------+      |                       |      +------------+  |
|  |  VMSWITCH  |      |                       |      |  VMSWITCH  |  |
|  |     +      |      |                       |      |     +      |  |
|  |  VROUTER   |      |                       |      |  VROUTER   |  |
|  |            +--+ +---------+       +---------+ +--+            |  |
|  +------------+  | |         |       |         | |  +------------+  |
|    |        |    | |  10 Gb  +-------+  10 Gb  | |       |      |   |
|    |        |    +-+   NIC   +-------+   NIC   +-+       |      |   |
|  +----+ +------+   |         |       |         |    +------+ +----+ |
|  |    | |      |   +---------+       +---------+    |      | |    | |
|  | OS | | CONT |     |                       |      | CONT | | OS | |
|  |    | |      |     |                       |      |      | |    | |
|  +----+ +------+     |                       |      +------+ +----+ |
|                      |                       |                      |
+----------------------+                       +----------------------+
```

Following `NTttcp` options should be used for performance testing:

- on `sender`:

    ```
    # Flags description
    # -m 1,*,10.0.14 -- 1 thread, do not pin CPUs, receiver on IP 10.0.1.4
    # -l 128k -- send 128KB buffers
    # -t 15 -- test should run for 15 seconds

    .\NTttcp.exe -s -m 1,*,10.0.1.4 -l 128k -t 15
    ```

- on `receiver`:

    ```
    # Flags description
    # -m 1,*,10.0.14 -- 1 thread, do not pin CPUs, receiver on IP 10.0.1.4
    # -rb 2M -- configure 2MB receive buffers
    # -t 15 -- test should run for 15 seconds

    .\NTttcp.exe -r -m 1,*,10.0.1.4 -rb 2M -t 15
    ```

`NTttcp` outputs many metrics, but in case of TCP most important are:

- Throughput(MB/s)
- Retransmits
- Errors
- Avg. CPU %


### UDP performance

**TODO**


## 3. Performance baseline setup

### Raw OS

- 2 Windows Server compute nodes (node A and node B);
- compute nodes are configured without Hyper-V and Containers;
- node A and node B exchange TCP segments using `NTttcp` tool using command line options from _Test scenarios_ section.

```
+----------------------+                       +----------------------+
|    WINDOWS NODE A    |                       |    WINDOWS NODE B    |
|                      |                       |                      |
|                      |                       |                      |
|    +---------+     +-+-------+       +-------+-+     +---------+    |
|    |         |     |         |       |         |     |         |    |
|    |   OS    +-----+  10 Gb  +-------+  10 Gb  +-----+   OS    |    |
|    |  STACK  +-----+   NIC   +-------+   NIC   +-----+  STACK  |    |
|    |         |     |         |       |         |     |         |    |
|    +---------+     +-+-------+       +-------+-+     +---------+    |
|                      |                       |                      |
|                      |                       |                      |
|                      |                       |                      |
|                      |                       |                      |
+----------------------+                       +----------------------+
```

#### Results

Across multiple runs, the following average figures are observed:

- on node A:

| metric | result |
|---|---|
| Throughput (MB/s) | 921.895 |
| Retransmits | 32.2 |
| Errors | 0 |
| Avg. cpu % | 14.799 |

- on B:

| metric | result |
|---|---|
| Throughput (MB/s) | 921.900 |
| Retransmits | 0.1 |
| Errors | 0 |
| Avg. cpu % | 21.161 |


### Hyper-V with Containers

- 2 Windows Server compute nodes (node A and node B);
- Hyper-V and Docker are installed on both compute nodes;
- 1 container `sender` running on node A;
- 1 container `receiver` running on node B;
- `sender` and `receiver` exchange TCP segments using `NTttcp` tool using command line options from _Test scenarios_ section


```
+----------------------+                       +----------------------+
|                      |                       |                      |
|  +------------+      |                       |      +------------+  |
|  |  VMSWITCH  |      |                       |      |  VMSWITCH  |  |
|  |            +--+ +---------+       +---------+ +--+            |  |
|  +------------+  | |         |       |         | |  +------------+  |
|    |        |    | |  10 Gb  +-------+  10 Gb  | |       |      |   |
|    |        |    +-+   NIC   +-------+   NIC   +-+       |      |   |
|  +----+ +------+   |         |       |         |    +------+ +----+ |
|  |    | |      |   +---------+       +---------+    |      | |    | |
|  | OS | | CONT |     |                       |      | CONT | | OS | |
|  |    | |      |     |                       |      |      | |    | |
|  +----+ +------+     |                       |      +------+ +----+ |
|                      |                       |                      |
+----------------------+                       +----------------------+
```

#### Setup

```powershell
# System setup
Install-WindowsFeature Containers
Install-WindowsFeature NET-Framework-Features
Install-WindowsFeature Hyper-V -IncludeManagementTools
Restart-Computer
Install-Module DockerMsftProvider -Force
Install-Package Docker -ProviderName DockerMsftProvider -Force -RequiredVersion 17.06.2-ee-16
Restart-Computer

# Docker setup (on both VMs)
Start-Service Docker
docker image pull microsoft/windowsservercore:ltsc2016
docker network create -d transparent --subnet=172.16.0.0/24 --gateway=172.16.0.254 -o com.docker.network.windowsshim.interface="Ethernet1" mynetwork

# On sender
docker run -id --rm --network mynetwork --ip=172.16.0.21 --name sender microsoft/windowsservercore:ltsc2016 powershell
docker cp .\NTttcp.exe sender:C:\
docker exec -it sender powershell

# On receiver
docker run -id --rm --network mynetwork --ip=172.16.0.22 --name receiver microsoft/windowsservercore:ltsc2016 powershell
docker cp .\NTttcp.exe receiver:C:\
docker exec -it receiver powershell

# Testing (commands inside container)
sender   > .\NTttcp.exe -s -m 1,*,172.16.0.22 -l 128k -t 15
receiver > .\NTttcp.exe -r -m 1,*,172.16.0.22 -rb 2M -t 15
```


#### Results

Across multiple runs of baseline scenario, the following average figures are observed:

- on `sender` container:

| metric | result |
|---|---|
| Throughput (MB/s) | 199.086 |
| Retransmits | 41.9 |
| Errors | 0 |
| Avg. cpu % | 15.843 |

- on `receiver` container:

| metric | result |
|---|---|
| Throughput (MB/s) | 199.087 |
| Retransmits | 0 |
| Errors | 0 |
| Avg. cpu % | 42.278 |
