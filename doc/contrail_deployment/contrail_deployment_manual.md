# How to deploy Contrail with Windows compute nodes

## Prerequisites
* Machine with CentOS 7.4 installed
* Machine(s) with Windows Server 2016 and updates installed
* Machine for running Ansible playbooks (Linux or Windows with WSL)
## Steps
- On each of the Windows hosts:

        Invoke-WebRequest https://raw.githubusercontent.com/ansible/ansible/devel/examples/scripts/ConfigureRemotingForAnsible.ps1 -OutFile ConfigureRemotingForAnsible.ps1
        .\ConfigureRemotingForAnsible.ps1 -DisableBasicAuth -EnableCredSSP -ForceNewSSLCert -SkipNetworkProfileCheck

* On the Ansible machine:

        git clone git@github.com:Juniper/contrail-ansible-deployer.git
        cd contrail-ansible-deployer
        vim config/instances.yaml

    * Refer to examples:
        * `config/instances.yaml.bms_win_example` if you have already deployed controller and you only want windows compute nodes
        * `config/instances.yaml.bms_win_full_example` if you want to deploy controller and windows compute nodes together
    * More precise explanation of the required fields:
        * For Windows computes use `bms_win` dict instead of regular `bms`
        * Roles supported on Windows are `vrouter` and `win_docker_driver` - specify them
        * Set `WINDOWS_PHYSICAL_INTERFACE` to dataplane interface alias
        * Specify `WINDOWS_ENABLE_TEST_SIGNING` if you want to install untrusted artifacts (this is needed at this moment)
        * Set `WINDOWS_DEBUG_DLLS_PATH` to path on Ansible machine containing MSVC 2015 debug dlls, specifically: `msvcp140d.dll`, `ucrtbased.dll` and `vcruntime140d.dll`
    * To deploy a controller for windows compute nodes:
        * Configure as it would be a controller node for a linux ecosystem
        * For now docker driver on windows needs keystone set up on controller:
            * Add openstack-* roles to the controller node and set `CLOUD_ORCHESTRATOR` to `openstack`
            * Fill keystone credentials and kolla config. Check `config/instances.yaml.openstack_example`
    * Proceed with running Ansible playbooks:
        * If you have already deployed the controller:

                sudo -H ansible-playbook -i inventory/ playbooks/configure_instances.yml
                sudo -H ansible-playbook -i inventory/ playbooks/install_contrail.yml

        * If you don't have the controller:

                sudo -H ansible-playbook -e orchestrator=openstack -i inventory/ playbooks/configure_instances.yml
                sudo -H ansible-playbook -i inventory playbooks/install_openstack.yml
                sudo -H ansible-playbook -e orchestrator=openstack -i inventory/ playbooks/install_contrail.yml

## Testing the new setup
Refer to [this document](../user_guide/connection_scenarios.md)
