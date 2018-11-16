# Heterogenous Contrail Deployment: Linux + Windows

The following procedure allows the operator to bring up heterogenous Contrail deployment. It will consist of at least
one Contrail Controller (CentOS machine) and at least one Windows Compute node.

**Note**: if you encounter any problems during or after deployment,
see [troubleshooting section](../Known_issues/Troubleshooting.md).

## 1. Prerequisites

To deploy Windows Contrail, you will need:

* Machine with CentOS 7.5 installed (for Contrail Controller),
* Machine(s) with Windows Server 2016 (for Windows compute nodes - please see requirements below),
* Machine for running Ansible playbooks (Linux CentOS or Windows with WSL). This can be your laptop.
You need Ansible v2.4.2.

Requirements for Windows Server 2016 machine:

* Minimum hardware requirements:
    * 2 CPU,
    * 4 GB RAM,
    * 60 GB HDD.
* Virtualization support must be enabled:
    * in case of a bare metal - enable VT-x in BIOS,
    * in case of a virtual machine - please see [Hypervisors](#hypervisors) for configuration details.
* Newest Windows updates should be installed.
* Windows machines should have different hostnames.
* Windows machines should be accessible using the same set of credentials.

## 2. Enable Ansible remoting

On each of the Windows hosts enable Ansible remoting:

    # PowerShell
    Invoke-WebRequest https://raw.githubusercontent.com/ansible/ansible/devel/examples/scripts/ConfigureRemotingForAnsible.ps1 -OutFile ConfigureRemotingForAnsible.ps1
    .\ConfigureRemotingForAnsible.ps1 -DisableBasicAuth -EnableCredSSP -ForceNewSSLCert -SkipNetworkProfileCheck

## 3. Configure Contrail-Ansible-Deployer

On the Ansible machine:

1. Clone the Contrail-Ansible-Deployer repository:

        # bash
        git clone https://Juniper/contrail-ansible-deployer
        cd contrail-ansible-deployer
        vim config/instances.yaml

1. Fill in the `config/instances.yaml` file. See the instructions below on how to do it.

### Example configurations

Refer to examples of `instances.yaml` file in Contrail-Ansible-Deployer repository:

* `config/instances.yaml.bms_win_example` if you have already deployed Contrail controller and you only want
    Windows compute nodes
* `config/instances.yaml.bms_win_full_example` if you want to deploy Contrail controller, Openstack and Windows
    compute nodes together. This is only useful if you want to use Keystone for auth.
* `config/instances.yaml.bms_win_no_openstack_example` if you want to deploy Contrail controller (without OpenStack)
    and Windows compute nodes together

### Instances

You will need to know the IP addresses of CentOS and Windows hosts.

* For Windows computes use `bms_win` dict instead of regular `bms`.
* You need to add `vrouter` and `win_docker_driver` roles for Windows compute nodes.
* Set `WINDOWS_PHYSICAL_INTERFACE` to dataplane interface name (run `Get-NetAdapter` from PowerShell to list available
    interfaces on Windows compute node). If your Compute nodes have only one interface, specify it. Otherwise, you
    can split data and control planes between two interfaces - you can choose. Refer to Contrail documentation
    regarding data and control plane separation.
* If interface name contains spaces, enclose it between quotation marks.

### [FIXME] Windows Contrail dev build - deployment configuration

Currently, only unsigned and debug builds of Windows Contrail components are available. As a result, the following
configuration is also required:

* In BIOS of every Windows node you need to disable secure boot.
* Add `WINDOWS_ENABLE_TEST_SIGNING` option and leave it empty. This option configures Windows Server to allow
    installation of unsigned drivers.
* Set `WINDOWS_DEBUG_DLLS_PATH` to path on Ansible machine containing MSVC 2015 debug dll libraries.Since user space
    Contrail components are build in debug mode, to run them on Windows Server the following dlls are required:
    * `msvcp140d.dll`,
    * `ucrtbased.dll`,
    * `vcruntime140d.dll`
    
    MSVC 2015 debug DLLs can be obtained by installing Visual Studio 2015. After installing Visual Studio they should
    be located in:

    * `mscvp140d.dll` and `vcruntime140d.dll` -
        `C:\Program Files (x86)\Microsoft Visual Studio 14.0\VC\redist\debug_nonredist\x64\Microsoft.VC140.DebugCRT`,
    * `ucrtbased.dll` - `C:\Program Files (x86)\Windows Kits\10\bin\x64\ucrt`.

    You should copy them to the `~/dll` directory on your Ansible machine and point `WINDOWS_DEBUG_DLLS_PATH`
    variable to this directory. Make sure they are 64-bit.

### Contrail controller configuration

Configure Contrail Controller it would be a normal controller node for a Linux ecosystem. Refer to
[Contrail-Ansible-Deployer wiki](https://github.com/Juniper/contrail-ansible-deployer/wiki).

If you wish to use keystone as authentication service on controller:

* Add `openstack-*` roles to the controller node and set `CLOUD_ORCHESTRATOR` to `openstack`
* Fill Keystone credentials and Kolla config. Refer to `config/instances.yaml.bms_win_full_example`.

Otherwise:

* Set `CLOUD_ORCHESTRATOR` to `none`.

## 4. Run Contrail-Ansible-Deployer

Proceed with running Ansible playbooks:

* If you have already deployed the Controller **or** if you want to deploy Controller without OpenStack (noauth mode):

        sudo -H ansible-playbook -i inventory/ playbooks/configure_instances.yml
        sudo -H ansible-playbook -i inventory/ playbooks/install_contrail.yml

* If you want to deploy controller with OpenStack (Keystone auth):

        sudo -H ansible-playbook -e orchestrator=openstack -i inventory/ playbooks/configure_instances.yml
        sudo -H ansible-playbook -i inventory playbooks/install_openstack.yml
        sudo -H ansible-playbook -e orchestrator=openstack -i inventory/ playbooks/install_contrail.yml

**Important**: you can re-run any `ansible-playbook`, but you shouldn't change their order. E.g. if you already ran
`install_contrail.yml`, then rerunning `configure_instances.yml` may lead to errors.

## 5. Verify deployment

1. Run `Invoke-DiagnosticCheck.ps1` script from [tools repository](https://github.com/Juniper/contrail-windows-tools)
    on Windows compute nodes. If deployment went correctly, all checks should pass.

    **Note**: to quickly have the ability to run this script on your Windows nodes, you can use the following snippet:
    
        Invoke-WebRequest  https://raw.githubusercontent.com/Juniper/contrail-windows-tools/master/Invoke-ScriptInRemoteSessions.ps1 -OutFile Invoke-ScriptInRemoteSessions.ps1
        Invoke-WebRequest  https://raw.githubusercontent.com/Juniper/contrail-windows-tools/master/Invoke-DiagnosticCheck.ps1 -OutFile Invoke-DiagnosticCheck.ps1
        .\Invoke-ScriptInRemoteSessions.ps1 -ScriptFileName ".\Invoke-DiagnosticCheck.ps1" -Addresses "<IP1>,<IP2>" -Credential (Get-Credential) -OtherParams...

    Consult the README on how to configure the diagnostic script (it's safe to run, so don't worry about
    misconfiguration).

1. Refer to [usage documentation](./usage.md) to learn how to create networks and containers.
1. (Optional) Refer to usage examples and run manual tests. Refer to [this document](./connection_scenarios.md).

## 6. Maintain

1. (Optional) Upgrade Windows Contrail to newest version. See [upgrading section](./upgrading.md).

## 7. Appendices

### Hypervisors

Contrail Windows deployment has been tested on the following hypervisors:

- VMware,
- KVM,
- Hyper-V.

Each of them requires a few tweaks required to enable nested Hyper-V.

#### VMware

Contrail Windows deployment has been successfully tested on VMware ESXi 6.5 hypervisor.
Virtual machines for compute nodes should be configured to support `ESXi 6.5 or later` (VM version 13) and VM secure boot should be disabled.

VMware should expose hardware virtualization to the guest OS.
To configure it:

1. Locate virtual machine in vSphere Web Client.
1. Make sure it is powered off.
1. Right click on it and select `Edit settings` option.
1. In `Virtual hardware`, in `CPU` section, check `Expose hardware assisted virtualization to the guest OS` option.
1. Click `OK`.
1. Power on the virtual machine.

#### KVM

Contrail Windows deployment has been successfully tested on KVM virtual machines.
The following configuration is required:

1. Hypervisor configuration:
    - Linux kernel version `>= 4.15`.
        - There were several issues with nested Hyper-V in KVM.
          Please refer to [https://ladipro.wordpress.com/2017/07/21/hyperv-in-kvm-known-issues/](https://ladipro.wordpress.com/2017/07/21/hyperv-in-kvm-known-issues/) for more information.
    - QEMU version `>= 2.11`.
2. VM configuration:
    - virtio NICs should be used.
    - Windows virtio driver version `>= 0.1.141`.
        - downloadable from [https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso).

#### Hyper-V

Contrail Windows deployment has been successfully tested on Hyper-V virtual machines.
The following VM configuration is required:

- Hyper-V Generation 2 VMs must be used.
- Secure boot must be disabled on the VM.
