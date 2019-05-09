There are three ways in which vRouter Agent exchanges information with the Forwarding Extension, which mimics their Linux behaviour.

### Ksync

This communication channel is used for inserting forwarding rules into the Forwarding Extension, like routes and next hops.

On Linux, Netlink sockets are used for this. Equivalent Windows mechanism is a Named Pipe.

Only vRouter Agent and other userspace programs can initialize the communiation. Howerver, different programs (for example, Agent and utils) may talk via ksync at the same time, so process separation (session layer) is required.

This communication is performed over a binary protocol called Sandesh (the same one which is used by Contrail Analytics Nodes).

## Flow

All flows are inserted into the Forwarding Extension using shared memory mechanism. Shared memory is allocated inside Forwarding Extension, and is written to by Agent. Only Agent may allocate flows inside the shared memory. Forwarding Extension may modify the shared memory contents, but it never directly communicates with vRouter Agent using it or creates new flows - it mostly updates flow statistics.

Only Agent and "flow" util need access to shared memory from userspace.

Since kernel memory is mapped into userspace, so the team ensured that the implementation is secure.

## pkt0

This channel is asynchronous - both vRouter Agent and Forwarding Extension can send packets through it at arbitrary times.

When a "new" packet arrives at Forwarding Extension (for example, ARP or DHCP request), it will transfer it using pkt0 to vRouter Agent. vRouter Agent will then analyze the packet (possibly sending it to Contrail Controller for even more analysis), and insert ksync rules or flows into the Forwarding Extension. Then, it re-injects the packet into the Forwarding Extension. 

Pkt0 is implemented on Linux using a traditional networking interface, of AF_PACKET family. On Windows, this is done using a named pipe. The reason for this is that Windows implementation of Berkeley sockets does not allow any manipulation of any layer below IP.

Named pipe server is created by the Forwarding Extension.

# Sandesh

Sandesh is a binary protocol based on Apache Thrift. It's a set of header files to be included in projects that use it, but also a code generation tool. Sandesh uses own Interface Definition Language to generate code in many different languages, like Java, C, Python and Golang. Generated code consists of definitions of data structures and getter/setter functions.
