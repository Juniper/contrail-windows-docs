# Few words about DNS in Contrail

This document describes DNS modes in Contrail and gives some hints about troubleshooting.

## DNS Modes In Contrail
DNS queries from specific VM located on Contrail compute node can be handled differently depending on chosen DNS mode in configuration of IPAM in which the VM resides. There are 3 available DNS modes in Contrail:

* None - No DNS support for VMs
* Default - DNS requests from VMs are resolved via the fabric name servers
* Tenant - DNS requests from VMs are resolved via DNS servers deployed on tenant’s VMs
* Virtual - provides virtual DNS server which resolves DNS requests from VMs

## How to specify DNS mode in Contrail

First, modify agent's config. The config needs to have specified DNS server with IP of Contrail control node. Example of config:

    ...
    [DNS]
    # Client port used by vrouter-agent while connecting to contrail-named
    dns_client_port=53

    # List of IPAdress:Port of DNS Servers separated by space.
    servers=<controller's ip>:53

    # Timeout for DNS server queries in milli-seconds
    dns_timeout=10000

    # Maximum retries for DNS server queries
    dns_max_retries=5
    ...

Next, follow the steps at: [DNS and IPAM configuration](https://github.com/Juniper/contrail-controller/wiki/DNS-and-IPAM#configuration)

## Debugging

* Common troubleshooting:
    * Go to `http://<hypervisor's_ip>:8085/Snh_SandeshTraceRequest?x=DnsBind` for overall DNS information received/sent by agent
      * If agent knows doesn't know any DNS resolver then there should be **"...no DNS resolver for..."** log message.
    * Go to `http://<hypervisor's_ip>:8085/Snh_DnsInfo` for detailed information about DNS queries received/responces sent by agent since it had been running
* Specific mode troubleshooting:
    * Default:
        * Verify if Windows hypervisor where agent is running knows any DNS servers:
          * Compile the code from this [link](https://docs.microsoft.com/en-us/windows/desktop/api/iphlpapi/nf-iphlpapi-getnetworkparams)
          * Execute the program on the machine with agent, if no IP is shown, then OS doesn't know any DNS resolver.
          * If the program prints nothing, then it could be compiled for wrong architecture(i.e. x86 program running on x64 machine)
        * **"...no DNS resolver for..."** log message present when hypervisor knows some DNS servers could indicate a bug in implementation of agent's function responsible for building the default list of DNS resolvers.
    * Tenant:
        * To find out if Tenant DNS mode works correctly, deploy a simple (i.e. python) DNS server in one of VMs and pass IP of the VM as a a DNS server in configuration of IPAM.
    * Virtual:
        * Refer to: [vDNS debugging](https://github.com/Juniper/contrail-controller/wiki/vDNS-Debugging) for additional help.
