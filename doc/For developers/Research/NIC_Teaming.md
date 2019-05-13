# Hyper-V/Docker NIC teaming research

## Useful links

- [NIC Teaming in Windows Server 2016][nic-teaming]
- [NIC Teaming: Load Balancing modes][nic-teaming-lb-modes]
- [NIC Teaming: MAC address use][nic-teaming-mac-use]
- [Switch Embedded Teaming with Docker Networks][set-docker]
- [SET (Switch Embedded Teaming)][set]

## Command dump

```
# Set up a NIC team
PS> New-NetLbfoTeam -Name Team0 -TeamMembers Ethernet1,Ethernet2

# Check NIC team status and wait until it is UP
PS> Get-NetLbfoTeam -Name Team0
Name                   : Team0
Members                : {Ethernet1, Ethernet2}
TeamNics               : Team0
TeamingMode            : SwitchIndependent
LoadBalancingAlgorithm : Dynamic
Status                 : Degraded

PS> Get-NetLbfoTeam -Name Team0
Name                   : Team0
Members                : {Ethernet1, Ethernet2}
TeamNics               : Team0
TeamingMode            : SwitchIndependent
LoadBalancingAlgorithm : Dynamic
Status                 : Up

# Now you can inspect network adapters
PS> Get-NetAdapter -Name Team0,Ethernet1,Ethernet2
Name                      InterfaceDescription                    ifIndex Status       MacAddress             LinkSpeed
----                      --------------------                    ------- ------       ----------             ---------
Team0                     Microsoft Network Adapter Multiplexo...      15 Up           00-50-56-8C-14-20        20 Gbps
Ethernet1                 vmxnet3 Ethernet Adapter                      3 Up           00-50-56-8C-14-20        10 Gbps
Ethernet2                 vmxnet3 Ethernet Adapter #2                   4 Up           00-50-56-8C-05-58        10 Gbps

# You must reconfigure IP Address on NIC Team Adapter
PS> New-NetIPAddress -InterfaceAlias Team0 -AddressFamily IPv4 -IPAddress 172.16.0.11 -PrefixLength 24

```

## Discovered issues

-   **Let**: `Team0` be a NIC team with `Ethernet1` and `Ethernet2` NIC adapters.
    Configure `Team0` on both compute nodes.
    Deploy Contrail Windows on both compute nodes.
    Start pinging between containers located on different compute nodes.

    **Do**: On first compute node disconnect `Ethernet1`.

    **Then**: Ping between containers should fail. `Ethernet2` did not take over.

    - Possible cause: Some MAC conflict in vRouter? According to Agent inspector, vRouter sends tunneled packet to destination MAC of disabled NIC adapter.
      Ethernet frame might be dropped because of MAC checking on VMware vSwitch, however after enabling all security policies, frames were still dropped.

[nic-teaming]: https://docs.microsoft.com/en-us/windows-server/networking/technologies/nic-teaming/nic-teaming
[nic-teaming-lb-modes]: https://docs.microsoft.com/en-us/windows-server/networking/technologies/nic-teaming/nic-teaming-settings#load-balancing-modes
[nic-teaming-mac-use]: https://docs.microsoft.com/en-us/windows-server/networking/technologies/nic-teaming/nic-teaming-mac-address-use-and-management
[set-docker]: https://docs.microsoft.com/en-us/virtualization/windowscontainers/container-networking/advanced#switch-embedded-teaming-with-docker-networks
[set]: https://docs.microsoft.com/en-us/windows-server/virtualization/hyper-v-virtual-switch/RDMA-and-Switch-Embedded-Teaming#bkmk_sswitchembedded
