Contrail Windows Performance Testing
====================================

This document describes assumptions, utilized tools and conclusions drawn from performance tests of Contrail on Windows Server 2016.

## Introduction

TCP throughput was measured in the following scenarios:

| ID  | Test name                       | Description | Comments |
|-----|---------------------------------|-------------|----------|
| A   | Raw                             | Raw Windows Server network stack | |
| B   | Containers                      | Windows Server containers (WinSrv containers) on 2 compute nodes | |
| C   | Colocated Containers            | WinSrv containers on 1 compute node | |
| D   | Containers w/ Contrail          | WinSrv containers on 2 compute nodes, with Contrail Windows | WIP PR https://review.opencontrail.org/#/c/47292/ |
| E   | Containers (no seg)             | WinSrv containers on 2 compute nodes, traffic tuned to eliminate TCP segmentation | |
| F   | Containers w/ Contrail (no seg) | WinSrv containers on 2 compute nodes, with Contrail Windows, traffic tuned to eliminate TCP segmentation | |

Test results and scenarios are described in the following sections.

## Results

On `sender` nodes:

| Metric              | A        | B        | C        | D       | E       | F      |
|---------------------|----------|----------|----------|---------|---------|--------|
| Throughput (Mbit/s) | 7375.159 | 1592.691 | 1956.786 | 658.291 | 235.852 | 98.734 |
| Avg. CPU %          | 14.799   | 15.843   | 57.098   | 40.992  | 29.373  | 34.544 |

On `receiver` nodes:

| Metric              | A        | B        | C        | D       | E       | F      |
|---------------------|----------|----------|----------|---------|---------|--------|
| Throughput (Mbit/s) | 7375.198 | 1592.698 | 1948.824 | 657.520 | 235.784 | 98.732 |
| Avg. CPU %          | 21.161   | 42.278   | 57.086   | 48.184  | 25.064  | 25.763 |

## Conclusions

Conclusions:

- Enabling Hyper-V on Windows Server 2016 reduces TCP throughput by a factor of 3-4.
    - Observed reduced throghput is expected. As per Microsoft Networking blog (here: [VMQ Deep Dive][vmq]) this is by design.
      Issue is discussed more deeply in [Raw vs Hyper-V performance](#raw-vs-hyper-v-performance) section.
- Difference between TCP throughput scenario where containers are colocated and scenario where containers are on separate nodes, suggests that VMSwitch is a bottleneck.
- Comparing `Containers (no seg)` test with `Containers w/ Contrail (no seg)` shows that vRouter code paths could account for 50-60% drop in TCP throughput.

Performance baseline for Contrail Windows should be based on network performance of containers running on Hyper-V, which is `~1600 MBit/s` of TCP throughput between containers on different compute nodes.


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
- compute nodes without Hyper-V and Containers Windows features installed;
- node A and node B exchange TCP segments using `NTttcp` tool;
- used `NTttcp` options:

        # On node A
        .\NTttcp.exe -s -m 1,*,172.16.0.12 -l 128k -t 15

        # On node B
        .\NTttcp.exe -r -m 1,*,172.16.0.12 -rb 2M -t 15

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

        # On sender
        .\NTttcp.exe -s -m 1,*,172.16.0.22 -l 128k -a 2 -t 15

        # On receiver
        .\NTttcp.exe -r -m 1,*,172.16.0.22 -rb 2M -a 16 -t 15

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

        # On sender
        .\NTttcp.exe -s -m 1,*,172.16.0.22 -l 128k -a 2 -t 15

        # On receiver
        .\NTttcp.exe -r -m 1,*,172.16.0.22 -rb 2M -a 16 -t 15

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

        # On sender
        .\NTttcp.exe -s -m 1,*,10.0.1.4 -l 128k -a 2 -t 15

        # On receiver
        .\NTttcp.exe -r -m 1,*,10.0.1.4 -rb 2M -a 16 -t 15

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

        # On sender
        .\NTttcp.exe -s -m 1,*,172.16.0.22 -l 1390 -t 15 -ndl

        # On receiver
        .\NTttcp.exe -r -m 1,*,172.16.0.22 -rb 2M -t 15 -ndl

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

        # On sender
        .\NTttcp.exe -s -m 1,*,172.16.0.22 -l 1390 -t 15 -ndl

        # On receiver
        .\NTttcp.exe -r -m 1,*,172.16.0.22 -rb 2M -t 15 -ndl

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

# Docker setup (on both Windows hosts)
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

### Raw vs Hyper-V performance

According to the Microsoft Networking blog (here: [VMQ Deep Dive][vmq]) the following behavior can be observed:

- Without any VMSwitch/Hyper-V configured, packets are directed to different hardware queues based on flow hash.
  Each queue is assigned to a different CPU core.
  Windows utilizes RSS on NICs to achieve that.
- With VMSwitch configured, packets are directed to different hardware queues based on destination MAC address.
  Each queue is assigned to a different CPU core.
  As a result each virtual adapter is assigned to a single CPU core and network throughput should be capped at c. 2-3 GBps.
  This mechanism is called VMQ on Windows and it _effectively_ disables RSS.

Initial tests confirm these claims - after Hyper-V was enabled and VMSwitch was configured, the network throughput dropped from ~7400 Mbps to ~1600 Mbps for single TCP stream.
However, inspecting VMQ configuration provides that VMQ is enabled on the host (VMXNET3 virtual adapters do not support VMQ in nested Hyper-V), so packets should not be processed based on destination MAC address.
Thus, performance drop, in this case, comes from bare computational overhead imposed by Hyper-V/VMSwitch.

To test out this assumption, the following scenarios were tested:

| ID  | Test name               | Description |
|-----|-------------------------|-------------|
| A   | Raw                     | Raw Windows Server network stack |
| B   | 2 Containers            | 2 WinSrv containers on 2 compute nodes (1 container per node) |
| C   | 4 Containers            | 4 WinSrv containers on 2 compute nodes (2 containers per node) |
| D   | 2 Containers, 2 threads | Same as B, but NTttcp uses 2 threads instead of one |

If assumption is correct, then:

- network throughput in scenario `C` should be twice as high as in `B`,
- since throughput is bound per virtual adapter, throughput in scenario `D` should not go beyond the results for scenario `B`.

The results were as follows:

**sender** node:

| Metric              | A        | B        | C        | D        |
|---------------------|----------|----------|----------|----------|
| Throughput (Mbit/s) | 7375.159 | 1592.691 | 2313.072 | 2500.388 |
| Avg. CPU %          | 14.799   | 15.843   | 36.044   | 34.294   |

**receiver** node:

| Metric              | A        | B        | C        | D         |
|---------------------|----------|----------|----------|-----------|
| Throughput (Mbit/s) | 7375.198 | 1592.698 | 2316.991 | 2501.5896 |
| Avg. CPU %          | 21.161   | 42.278   | 69.274   | 76.055    |

Conclusions:

- enabling Hyper-V and introducing VMSwitch to a network stack, increases CPU usage on receiver side by a factor of 2-4;
- throughput does not seem to be capped per virtual adapter, since we observe the same performance boost in scenarios `C` and `D`;
- as a result, performance drop seems to be related to computational overhead imposed by Hyper-V/VMSwitch.

To test out the limits of this setup, we have increased a number of vCPUs on testbeds from 2 to 8 and ran a test similar to scenario `D`, but with 8 threads in NTttcp.
The results were as follows:

**sender** node:

| Metric              |          |
|---------------------|----------|
| Throughput (Mbit/s) | 4845.864 |
| Avg. CPU %          | 26.034   |

**receiver** node:

| Metric              |          |
|---------------------|----------|
| Throughput (Mbit/s) | 4845.932 |
| Avg. CPU %          | 63.549   |

These results support an assumption, that performance drop after enabling Hyper-V is mostly a result of software overhead.

[vmq]: https://blogs.technet.microsoft.com/networking/2013/09/10/vmq-deep-dive-1-of-3/
