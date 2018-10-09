# Deploying Contrail with Windows compute nodes

## Prerequisites

* Machine with CentOS 7.5 installed,
* Machine(s) with Windows Server 2016 (please see requirements below),
* Machine for running Ansible playbooks (Linux or Windows with WSL).

Requirements for Windows Server 2016 machine:

* Minimum hardware requirements:
    * 2 CPU,
    * 4 GB RAM,
    * 60 GB HDD.
* Virtualization support must be enabled:
    * in case of a bare metal - enable VT-x in BIOS,
    * in case of a virtual machine - enable nested virtualization.
* Newest Windows updates should be installed.
* Windows machines should have different hostnames.
* Windows machines should be accessible using the same set of credentials.

## Steps

- On each of the Windows hosts enable Ansible remoting:

        # PowerShell
        Invoke-WebRequest https://raw.githubusercontent.com/ansible/ansible/devel/examples/scripts/ConfigureRemotingForAnsible.ps1 -OutFile ConfigureRemotingForAnsible.ps1
        .\ConfigureRemotingForAnsible.ps1 -DisableBasicAuth -EnableCredSSP -ForceNewSSLCert -SkipNetworkProfileCheck

* On the Ansible machine:

        # bash
        git clone git@github.com:Juniper/contrail-ansible-deployer.git
        cd contrail-ansible-deployer
        vim config/instances.yaml

    * Refer to examples:
        * `config/instances.yaml.bms_win_example` if you have already deployed controller and you only want windows compute nodes
        * `config/instances.yaml.bms_win_full_example` if you want to deploy controller and windows compute nodes together
        * `config/instances.yaml.bms_win_no_openstack_example` if you want to deploy controller without OpenStack and windows compute nodes together
    * More precise explanation of the required fields:
        * For Windows computes use `bms_win` dict instead of regular `bms`
        * Roles supported on Windows are `vrouter` and `win_docker_driver` - specify them
        * Set `WINDOWS_PHYSICAL_INTERFACE` to dataplane interface name (run `Get-NetAdapter` from PowerShell to list available interfaces on Windows compute node)
        * Add `WINDOWS_ENABLE_TEST_SIGNING` option and leave it empty, if you want to install untrusted artifacts (this is needed at this moment)
        * Set `WINDOWS_DEBUG_DLLS_PATH` to path on Ansible machine containing MSVC 2015 debug dlls, specifically: `msvcp140d.dll`, `ucrtbased.dll` and `vcruntime140d.dll`. DLLs can be obtained by installing Windows SDK on Windows and copying DLLs to Ansible machine.
    * To deploy a controller for windows compute nodes:
        * Configure as it would be a controller node for a linux ecosystem
        * If you wish to use keystone as authentication service on controller:
            * Add openstack-* roles to the controller node and set `CLOUD_ORCHESTRATOR` to `openstack`
            * Fill keystone credentials and kolla config. Check `config/instances.yaml.openstack_example`
    * Proceed with running Ansible playbooks:
        * If you have already deployed the controller **or** if you want to deploy controller without OpenStack (noauth mode):

                sudo -H ansible-playbook -i inventory/ playbooks/configure_instances.yml
                sudo -H ansible-playbook -i inventory/ playbooks/install_contrail.yml

        * If you want to deploy controller with OpenStack:

                sudo -H ansible-playbook -e orchestrator=openstack -i inventory/ playbooks/configure_instances.yml
                sudo -H ansible-playbook -i inventory playbooks/install_openstack.yml
                sudo -H ansible-playbook -e orchestrator=openstack -i inventory/ playbooks/install_contrail.yml


## Testing the new setup

Refer to [this document](./connection_scenarios.md)
