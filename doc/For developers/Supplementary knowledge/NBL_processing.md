# NET_BUFFER_LIST processing

## NET_BUFFER_LIST and vr_packet relationship

![nbl-vrpacket](nbl-fragmenting.png)

- `NET_BUFFER_LIST` is a linked list of packet containers
- `NET_BUFFER` represents a single packet
- `MDL` linked list represent a series of memory buffers
- Windows API mostly accepts `NET_BUFFER_LIST` structures
- vRouter expects `vr_packet` to be a single network packet
- Thus Windows vRouter code must assure that `vr_packet` will encapsulate:
    - single `NET_BUFFER_LIST`
    - single `NET_BUFFER`

## Receiving fragmented packets from container

![nbl-fragmented-container-input](nbl-fragmentation1.png)

**Scenario**: Packet is fragmented in container, before reaching vRouter

- Extension receives a single `NET_BUFFER_LIST` with a set of `NET_BUFFER` structures
- Each `NET_BUFFER` should be in its own `NET_BUFFER_LIST` to fulfill single network packet - `vr_packet` relationship required by dp-core

## Fragmenting egress tunneled packets

![nbl-fragmenting-egress-tunneled](nbl-fragmentation2.png)

**Scenario**: Packet must be tunneled, but it's too big, thus must be fragmented by vRouter

- Windows API supports dividing packet into fragments
- Although it's vRouter responsibility to:
    - Add fragmentation info to IP headers
    - Fix packet headers (e.g. header checksums)
- Note: vRouter on Linux fragments the inner packet
    - As a result each fragment is encapsulated separately
    - vRouter on Windows implements the same behaviour
