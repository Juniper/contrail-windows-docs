## Link Local Service

As of 2019/01/24, Link Local Service logic in vRouter is working, however a manual workaround is required on Windows:

### Problem

Link local subnet (169.254.0.0/16) is treated differently on Linux and Windows.

-   **Windows** networking stack thinks that link local IPs are always ‘on-link’.
    So when sending packet from Windows container to link local IP, container sends ARP request to get the destination MAC.
    Windows does not use default gateway for these addresses.
    Link local subnet is used for IP autoconfiguration mechanism, which is described [below](#ip-autoconfiguration).
-   **Linux** networking stack does not assume that link local IPs are ‘on-link’.
    Linux uses default gateway for this subnet, thus vRouter has to handle the traffic.

Differences in ARP/routing handling between the operating systems are presented on some examples in the [following section](#arp-and-routing-tables).

### Workaround

Add routing rules for link local IPs in Windows container so the packets are sent to the gateway.

### Proposed solution

Add logic to vRouter agent so it responds to the ARP requests for link local IPs.

## Research

### IP Autoconfiguration

Windows implements IP autoconfiguration mechanism (APIPA) using a procedure described in [RFC 3927][zeroconf-ipv4-ietf].
The most important excerpt, which describes behavior we experienced during testing, is attached below:

    Whichever interface is used, if the destination address is in the
    169.254/16 prefix (excluding the address 169.254.255.255, which is
    the broadcast address for the Link-Local prefix), then the sender
    MUST ARP for the destination address and then send its packet
    directly to the destination on the same physical link.  This MUST be
    done whether the interface is configured with a Link-Local or a
    routable IPv4 address.

    In many network stacks, achieving this functionality may be as simple
    as adding a routing table entry indicating that 169.254/16 is
    directly reachable on the local link.

### ARP and routing tables

On Linux, after endpoint with an address from `169.254.0.0/16` subnet, when Avahi is not used (zeroconf implementation on Linux), the results are as follows:

    # Routing table
    ubuntu@vm1:~$ ip route show
    default via 10.0.1.1 dev ens3
    10.0.1.0/24 dev ens3  proto kernel  scope link  src 10.0.1.3

    # ARP table
    ubuntu@vm1:~$ ip neighbor show
    10.0.1.2 dev ens3 lladdr 00:00:5e:00:01:00 REACHABLE
    10.0.1.1 dev ens3 lladdr 00:00:5e:00:01:00 DELAY

On Windows, after endpoint with an address from `169.254.0.0/16` subnet the results are as follows:

    # Routing table
    Active Routes:
    Network Destination        Netmask          Gateway       Interface  Metric
    0.0.0.0          0.0.0.0         10.0.0.1        10.0.0.10    271
    10.0.0.0    255.255.255.0         On-link         10.0.0.10    271
    10.0.0.10  255.255.255.255         On-link         10.0.0.10    271
    10.0.0.255  255.255.255.255         On-link         10.0.0.10    271
    127.0.0.0        255.0.0.0         On-link         127.0.0.1    331
    127.0.0.1  255.255.255.255         On-link         127.0.0.1    331
    127.255.255.255  255.255.255.255         On-link         127.0.0.1    331
    224.0.0.0        240.0.0.0         On-link         127.0.0.1    331
    224.0.0.0        240.0.0.0         On-link         10.0.0.10    271
    255.255.255.255  255.255.255.255         On-link         127.0.0.1    331
    255.255.255.255  255.255.255.255         On-link         10.0.0.10    271

    # ARP table
    Internet Address                              Physical Address   Type
    --------------------------------------------  -----------------  -----------
    10.0.0.1                                      00-00-5e-00-01-00  Probe
    10.0.0.2                                      00-00-5e-00-01-00  Stale
    169.254.169.254                               00-00-00-00-00-00  Unreachable

[zeroconf-ipv4-ietf]: http://files.zeroconf.org/draft-ietf-zeroconf-ipv4-linklocal.txt
