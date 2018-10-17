# Deploying Contrail with Windows compute nodes

## Troubleshooting

**Note**: if you encounter any problems during or after deployment,
see [troubleshooting section](../Known_issues/Troubleshooting.md).

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
        * Set `WINDOWS_PHYSICAL_INTERFACE` to dataplane interface name (run `Get-NetAdapter` from PowerShell to list available interfaces on Windows compute node).
          If interface name contains spaces, enclose it between quotation marks.
    * As of October 2018, only unsigned and debug builds of Contrail components are available. As a result, the following configuration is also required:
        * Add `WINDOWS_ENABLE_TEST_SIGNING` option and leave it empty. This option configures Windows Server to allow installation of unsigned drivers.
        * Set `WINDOWS_DEBUG_DLLS_PATH` to path on Ansible machine containing MSVC 2015 debug dlls. Since user space Contrail components are build in debug mode, to run them on Windows Server the following dlls are required:
            * `msvcp140d.dll`,
            * `ucrtbased.dll`,
            * `vcruntime140d.dll`
        * MSVC 2015 debug DLLs can be obtained by installing Visual Studio 2015. After installing Visual Studio they should be located in:
            * `mscvp140d.dll` and `vcruntime140d.dll` - `C:\Program Files (x86)\Microsoft Visual Studio 14.0\VC\redist\debug_nonredist\x64\Microsoft.VC140.DebugCRT`,
            * `ucrtbased.dll` - `C:\Program Files (x86)\Windows Kits\10\bin\x64\ucrt`.
        * You should copy them to the `~/dll` directory on your Ansible machine and point `WINDOWS_DEBUG_DLLS_PATH` variable to this directory.
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

## Next steps

1. Run `Invoke-DiagnosticCheck.ps1` script from [tools repository](https://github.com/Juniper/contrail-windows-tools).
    If deployment went correctly, all checks should pass.

    **Note**: to quickly have the ability to run this script on your Windows machine, you can use the following snippet:
    
        Invoke-WebRequest  https://raw.githubusercontent.com/Juniper/contrail-windows-tools/master/Invoke-DiagnosticCheck.ps1 -OutFile Invoke-DiagnosticCheck.ps1

    Consult the README on how to configure the diagnostic script (it's safe to run, so don't worry about
    misconfiguration).

1. Refer to [usage documentation](./usage.md) to learn how to create networks and containers.
1. You can run manual connection dev tests. Refer to [this document](./connection_scenarios.md).
