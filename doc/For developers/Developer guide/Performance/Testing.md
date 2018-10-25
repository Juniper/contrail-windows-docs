Contrail Windows Performance Testing
====================================

This document describes assumptions, utilized tools and conclusions drawn from performance tests of Contrail on Windows Server 2016.

## Introduction

TCP throughput was measured in the following scenarios:

1. Raw Windows Server network stack (abbreviated `Raw` in _Results_ section).
1. Windows Server containers (WinSrv containers) on 2 compute nodes (abbreviated `Containers` in _Results_ section).
1. WinSrv containers on 1 compute node (abbreviated `Colocated Containers` in _Results_ section).
1. WinSrv containers on 2 compute nodes, with Contrail Windows (abbreviated `Containers w/ Contrail` in _Results_ section).
1. WinSrv containers on 2 compute nodes, traffic tuned to eliminate TCP segmentation (abbreviated `Containers (no seg)` in _Results_ section).
1. WinSrv containers on 2 compute nodes, with Contrail Windows, traffic tuned to eliminate TCP segmentation (abbreviated `Containers w/ Contrail (no seg)` in _Results_ section).

Test results and scenarios are described in the following sections.

## Results

| Metric              | Raw      | Containers | Colocated Containers | Containers w/ Contrail | Containers (no seg) | Containers w/ Contrail (no seg) |
|---------------------|----------|------------|----------------------|------------------------|---------------------|---------------------------------|
| Throughput (Mbit/s) | 7375.159 | 1592.691   | 1956.786             | TBD                    | 235.852             | 98.734                          |
| Retransmits         | 257      | 335        | 2333                 | TBD                    | 1                   | 1034                            |
| Errors              | 0        | 0          | 0                    | TBD                    | 0                   | 0                               |
| Avg. CPU %          | 14.799   | 15.843     | 57.098               | TBD                    | 29.373              | 34.544                          |

## Conclusions

Tests have shown that:

- Windows networking stack can achieve on average ~7300 Mbit/s of TCP throughput;
- Windows container networking stack can achieve on average ~1500 Mbit/s of TCP throughput;
- Windows networking stack, when Hyper-V is enabled, achieves throughput similar to Windows container networking stack;
- Windows container networking stack, when containers are colocated on the same node, can achieve on average ~1900Mbit/s of TCP throughput.
  However, a high number of retransmissions occurring is observed - on average, 2333 retransmissions;

Conclusions:

- Enabling Hyper-V on Windows Server 2016 reduces TCP throughput by a factor of 3-4.
- Reduced throughput can be explained by lack of support for VMQ in vmxnet3 adapters. `receiver` node has much higher CPU load when using Hyper-V.
- Difference between TCP throughput scenario where containers are colocated and scenario where containers are on separate nodes, suggests that VMSwitch is a bottleneck in this case.

Refer to _Performance baseline test results_ section for a more detailed description of test setups and results.

Performance baseline for Contrail Windows should be based on network performance of containers running on Hyper-V.


## Test environment assumptions

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


## Test scenarios

### Raw

Description:

- 2 Windows Server compute nodes (node A and node B);
- compute nodes are configured without Hyper-V and Containers;
- node A and node B exchange TCP segments using `NTttcp` tool;
- used `NTttcp` options:

    ```
    # On node A
    .\NTttcp.exe -s -m 1,*,172.16.0.12 -l 128k -t 15

    # On node B
    .\NTttcp.exe -r -m 1,*,172.16.0.12 -rb 2M -t 15
    ```

Diagram:

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

### Containers

Description:

- 2 Windows Server compute nodes (node A and node B);
- Hyper-V and Docker are installed on both compute nodes;
- 1 container `sender` running on node A;
- 1 container `receiver` running on node B;
- `sender` and `receiver` exchange TCP segments using `NTttcp` tool;
- used `NTttcp` options:

    ```
    # On sender
    .\NTttcp.exe -s -m 1,*,172.16.0.22 -l 128k -a 2 -t 15

    # On receiver
    .\NTttcp.exe -r -m 1,*,172.16.0.22 -rb 2M -a 16 -t 15
    ```

Diagram:


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

### Colocated Containers

Description:

- 1 Windows Server compute node;
- Hyper-V and Docker are installed on compute note;
- containers `sender` and `receiver` are running on this compute node;
- `sender` and `receiver` exchange TCP segments using `NTttcp` tool using command line options from _Test scenarios_ section.
- used `NTttcp` options:

    ```
    # On sender
    .\NTttcp.exe -s -m 1,*,172.16.0.22 -l 128k -a 2 -t 15

    # On receiver
    .\NTttcp.exe -r -m 1,*,172.16.0.22 -rb 2M -a 16 -t 15
    ```

Diagram:

```
+---------------------+
|                     |
|   +-------------+   |
|   |  VMSWITCH   |   |
|   |             |   |
|   +-+---------+-+   |
|     |         |     |
|     |         |     |
|  +--+---+ +---+--+  |
|  |      | |      |  |
|  | CONT | | CONT |  |
|  |      | |      |  |
|  +------+ +------+  |
|                     |
+---------------------+
```

### Containers w/ Contrail

Description:

- 2 compute nodes (node A and node B);
- 1 Contrail network named `network1`;
- 1 container `sender` on node A, attached to `network1` network;
- 1 container `receiver` on node B, attached to `network1` network;
- `sender` and `receiver` exchange TCP segments using `NTttcp` tool;
- used `NTttcp` options:

    ```
    # On sender
    .\NTttcp.exe -s -m 1,*,10.0.1.4 -l 128k -a 2 -t 15

    # On receiver
    .\NTttcp.exe -r -m 1,*,10.0.1.4 -rb 2M -a 16 -t 15
    ```

Diagram:

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

### Containers (no seg)

Description:

- 2 Windows Server compute nodes (node A and node B);
- Hyper-V and Docker are installed on both compute nodes;
- 1 container `sender` running on node A;
- 1 container `receiver` running on node B;
- `sender` and `receiver` exchange TCP segments using `NTttcp` tool;
- `NTttcp` options tuned so that no fragmentation and segmentation should occur; options used:

    ```
    # On sender
    .\NTttcp.exe -s -m 1,*,172.16.0.22 -l 1390 -t 15 -ndl

    # On receiver
    .\NTttcp.exe -r -m 1,*,172.16.0.22 -rb 2M -t 15 -ndl
    ```

Diagram:

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

### Containers w/ Contrail (no seg)

Description:

- 2 compute nodes (node A and node B);
- 1 Contrail network named `network1`;
- 1 container `sender` on node A, attached to `network1` network;
- 1 container `receiver` on node B, attached to `network1` network;
- `sender` and `receiver` exchange TCP segments using `NTttcp` tool;
- `NTttcp` options tuned so that no fragmentation and segmentation should occur; options used:

    ```
    # On sender
    .\NTttcp.exe -s -m 1,*,172.16.0.22 -l 1390 -t 15 -ndl

    # On receiver
    .\NTttcp.exe -r -m 1,*,172.16.0.22 -rb 2M -t 15 -ndl
    ```

Diagram:


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


__ASDJAJSDLJASDLJASLD__

### TCP performance

TCP test scenario:


Following `NTttcp` options should be used for performance testing:

- on `sender`:

    ```
    # Flags description
    # -m 1,*,10.0.1.4 -- 1 thread, do not pin CPUs, receiver on IP 10.0.1.4
    # -l 128k -- send 128KB buffers
    # -t 15 -- test should run for 15 seconds

    .\NTttcp.exe -s -m 1,*,10.0.1.4 -l 128k -t 15
    ```

- on `receiver`:

    ```
    # Flags description
    # -m 1,*,10.0.1.4 -- 1 thread, do not pin CPUs, receiver on IP 10.0.1.4
    # -rb 2M -- configure 2MB receive buffers
    # -t 15 -- test should run for 15 seconds

    .\NTttcp.exe -r -m 1,*,10.0.1.4 -rb 2M -t 15
    ```

## Appendices

### Hyper-V with Containers - setup

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
