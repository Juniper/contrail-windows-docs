# Contrail Controller

Contrail Controller is a logical component. In fact, it's a distributed system consisting of many nodes, most importantly:

* Config nodes. These nodes expose a REST API used by various applications to configure Contrail resources. Contrail's web client communicates with it, and so does Docker Network Driver. Config information is eventually propagated to Control nodes.
* Control nodes, responsible for interacting with vRouter Agent via XMPP protocol. These interactions include inserting forwarding rules and flows and sending packets for further analysis.
* Analytics nodes, which collect high volume information using Sandesh binary protocol.

vRouter Agent mainly interacts with Control and Analytics nodes, while Docker network driver interacts with Config nodes. 

# Compute Node

Compute Node is a Contrail component. It's a hypervisor that is able to spawn VMs or containers. Every compute node must have an instance of vRouter Agent running.

Windows Server 2016 and Nano Server are new versions of Microsoft's OS, that can act both as Hyper-V hypervisor as well as Windows Containers host.

Nano Server running as Compute Node is currently not supported, due to lacking the capability of installing Hyper-V networking extensions.

# vRouter Agent

Note: The team sometimes refers to vRouter Agent simply as "vRouter", "Agent" or "vRA".

vRouter Agent is a Contrail agent that runs in userspace on a compute node. It has many functions. Most importantly:

* it communicates with Contrail Controller via XMPP protocol. Controller will inject rules and flows into the agent so that it enacts it's routing policies and implements network virtualization,
exposes an API for the hypervisor, that is used for registering newly created virtual machines or containers. This * is most notably used in Contrail-OpenStack integration, where Nova-Agent informs vRouter Agent whenever a new VM is spawned. On Windows, the API is exposed over a named pipe.
* communicates with the Forwarding Extension (three communication channels in total). Since vRouter Agent is a control plane component, it must inject all low level information like next hops, virtual interface information, routes and flows into the datapath component (the Forwarding Extension).
* sometimes forwards packets from Forwarding Extension to Contrail Controller
* sometimes injects packets sent from Contrail Controller into the Forwarding Extension
