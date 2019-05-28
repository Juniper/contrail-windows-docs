# Communication: Agent ⇄ Extension

There are three ways in which vRouter Agent exchanges information with the
Forwarding Extension, which mimics their Linux behaviour.


## Ksync

This communication channel is used for inserting forwarding rules into the
Forwarding Extension, like routes, next hops and flows.

On Linux, Netlink sockets are used for this.
Equivalent Windows mechanism is a Named Pipe.

Named pipe server is created by the Forwarding Extension.
Only vRouter Agent and other userspace programs can initialize the communiation.
Howerver, different programs (for example, Agent and utils) may talk via ksync
at the same time, so process separation (session layer) is required.

This communication is performed over a binary protocol called Sandesh
(the same one which is used by Contrail Analytics Nodes).


### Sandesh

Sandesh is a binary protocol based on Apache Thrift. It's a set of header
files to be included in projects that use it, but also a code generation tool.
Sandesh uses own Interface Definition Language to generate code in many
different languages, like Java, C, Python and Golang. Generated code consists
of definitions of data structures and getter/setter functions.


## pkt0

This channel is asynchronous - both vRouter Agent and Forwarding Extension can
send packets through it at arbitrary times.

When a "new" packet arrives at Forwarding Extension
(for example, ARP or DHCP request), it will transfer it using pkt0 to
vRouter Agent. vRouter Agent will then analyze the packet and, basing on data
from Contrail Controller, insert routes, flows or next hops into the
Forwarding Extension using ksync. Then, it re-injects the packet into
the Forwarding Extension.

Pkt0 is implemented on Linux using a traditional networking interface,
of AF_PACKET family. On Windows, this is done using a named pipe.
The reason for this is that Windows implementation of BSD sockets does not
allow any manipulation of any layer below IP.

The named pipe server is created by the Forwarding Extension.


## Shared memory

Shared memory is allocated inside Forwarding Extension,
and can be read by Agent and utils. Currently there are two structures mapped
to userspace using shared memory: flow table and bridge table.

Only Agent and utils need access to shared memory from userspace.
Since kernel memory is mapped into userspace
the team ensured that the implementation is secure.
