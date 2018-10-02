There is a slight discrepancy between Docker's and Contrail's networking model. Contrail can implement a logical, overlay network for containers. However, docker can only create a network locally, on the hypervisor.

This means that "docker network create" command needs to be ran on each hypervisor that will contain any container that could be connected to a specific network. In other words, "local" networks must be prepared on each host.

Furthermore, Docker's "network" is actually equivalent to Contrail's "subnet". This means, that during docker network creation, specific Contrail subnet (using CIDR notation) must be specified as a parameter.

1. Docker client tells docker daemon to create a docker network that will represent a chunk of a Contrail subnet. Tenant name, network name and subnet's CIDR are passed as parameters.
1. Docker daemon creates a network resource in its own, local database, and then delegates handling of network configuration to the docker driver.
1. Docker driver queries Contrail Controller whether specified tenant, network and subnet combination exists. It also asks for their details.
1. Contrail Controller returns subnet's default gateway address as well as subnet's UUID
1. Docker driver calls HNS function to create a HNS network with subnet's CIDR and default gateway address. It also sets the HNS network's name as a concatenation of "Contrail" with tenant name, network name and subnet ID.
1. Docker driver tells docker daemon to store tenant and network names in docker network's metadata. Docker daemon stores this information in its own, local database.
