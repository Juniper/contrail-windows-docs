TODO(mc): from step 11. it's not exactly true, we've removed the hashmap, also agent can injects vifs before interface is attached to vswitch.

1. Docker client tells docker daemon to run a container and attach it to a Docker network (that corresponds to a chunk of a Contrail subnet).
1. Docker deamon delegates the network configuration of the container that is being created to the CNM plugin.
1. CNM plugin queries docker daemon about metadata associated with docker's network resource and receives Contrail tenant name and subnet CIDR.
1. CNM plugin performs a series of queries against Contrail Controller that create various Contrail resources, most notably: virtual-machine, virtual-machine-interface, instance-ip.
1. CNM plugin receives Instance IP, MAC and default gateway to configure the newly created container's interface with. It also receives UUID of virtual-machine-interface resource
1. CNM plugin knows all the information necessary to recreate HNS Network's name. It uses this name to identify which HNS network to attach the container to. CNM plugin sends requests to HNS to configure endpoint with newly received IP, MAC and Default gateway.
1. In the background, HNS creates the network compartment for the container and configures its interface.
1. The moment the interface is created, the Forwarding Extension receives an event about new adapter being connected. 
1. Forwarding Extension doesn't know what to do with it yet, so it stores it in a hash map, where the key is adapters FriendlyName. Then it waits, dropping all packets except the ones sent from native Hyper-V driver.
1. Meanwhile, if container creation was successful, HNS returns ID of the endpoint to CNM plugin.
1. CNM plugin sends "add port" request to vRouter Agent, with all the necessary information to identify the port both in Contrail terms (UUID of virtual-machine-interface) as well as in Windows terms (using Endpoint ID, which is a part of FriendlyName).
1. vRouter Agent communicates with Contrail Controller to let it know about newly added port.
1. vRouter Agent inserts basic rules and flows into the Forwarding Extension. 
1. Forwarding Extension uses the FriendlyName to determine which port seen in userspace corresponds to port waiting in Forwarding Extension's hash map.
