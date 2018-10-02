# Devenv in CodiLime VMware lab

This document describes the steps required to provision a development environment in CodiLime VMware lab.
From high-level perspective steps are as follows:

1. Provision Windows and Linux virtual machines with suitable configuration in CodiLime VMware lab.
2. Configure Networking on Windows and Linux virtual machines using VMware console.
3. Run Ansible playbook to deploy Windows Contrail compute and Contrail Controller nodes.

Performing steps described below correctly should result in a configured devenv consiting of:

- 2 virtual machines configured as Windows Contrail compute, named `[INITIALS]-tb1` and `[INITIALS]-tb2`
    - Windows will have vRouter dependencies installed
    - vRouter/vRouter Agent/Docker Driver will not be installed
- 1 virtual machine configured as Contrail Controller node named `[INITIALS]-ctrl`
    - Contrail Controller will be fully installed and configured

## Prerequisites

### Select private network for devenv

Because of lacking infrastructure and licensing we simulate private networks based on PVLANs, using VMware's standard virtual switches.
This limits a communication in this private network to a single ESXi host.
Thus each virtual machine from a single devenv must be placed on a single ESXi host.
There are two ESXi hosts at this moment: `10.5.88.85` and `10.5.88.86`

These private networks are named `vS-DevEnvX-Y`.
`X` is an identifier of the host.
`Y` is an identifier of the network.
`X` can be:

- If you chose `10.5.88.85` as your host, then `X = 1`
- If you chose `10.5.88.86` as your host, then `X = 2`

To find an available network for use do the following:

- In vSphere Web Client, navigate to `Networking` tab in the navigation pane on the left
- On the left should be a list of VMware networks
- Look for an available network from networks named `vS-DevEnvX-Y`:
    - Click on the network `vS-DevEnvX-Y` 
    - Select `VMs` tab in the center pane
    - Make sure `Virtual Machines` option is selected
    - If the table should be empty, there are no virtual machines in this network, thus it is available
    - If there are some virtual machines, check another network


### Virtual machines addressing

This document supports the following addressing scheme for the devenv:

| Virtual machine  | Type                | Network Adapter 1             | Network Adapter 2      |
| ---------------- | ------------------- | ----------------------------- | ---------------------- |
| [INITIALS]-tb1   | Windows Compute     | DHCP, from 10.7.0.0/24 subnet | Static, 172.16.0.1/24  |
| [INITIALS]-tb2   | Windows Compute     | DHCP, from 10.7.0.0/24 subnet | Static, 172.16.0.2/24  |
| [INITIALS]-ctrl  | Contrail Controller | DHCP, from 10.7.0.0/24 subnet | Static, 172.16.0.10/24 |

In this scenario:

- `Network Adapter 1` corresponds to management adapter (Contrail's mgmt plane)
    - On Windows - `Ethernet0`
    - On CentOS - `ens192`
- `Network Adapter 2` corresponds to control adapter (Contrail's control and data plane)
    - On Windows - `Ethernet1`
    - On CentOS - `ens224`


## Devenv setup

### Windows Testbed setup

1. Create a folder for virtual machines
    - Select `VMs and Templates` tab in navigation pane on the left
    - Traverse to `Dev` folder
    - Right clink on `Dev` directory and select `New folder`
    - Provide folder name in the text box
        - Name it with your initials, e.g. `DS`
        - I'll refer to this folder's name as `[DESTINATION]`
2. Clone virtual machine from `Codilime/__Templates/windows/win2016core-bare` template
    - Select `VMs and Templates` tab in navigation pane on the left
    - Traverse to `Codilime/__Templates/windows/` folder
    - Right click on `win2016core-bare` template and select `New VM from This Template`
    - `Deploy from Template` wizard should appear
        - In `Select a name and folder` step do the following:
            - Provide virtual machine name in `Enter a name for the virtual machine` input box
                - Name it using your initials, appending testbed number, e.g. `DS-tb1`
                - I'll refer to this named as `[INITIALS]-tb1`
            - Using `Select a location for the virtual machine` window traverse to `Codilime/Dev/[DESTINATION]` folder
            - Select `[DESTINATION]` folder in the tree
            - Click `Next`
        - In `Select a compute resource` step do the following:
            - Expand `MSDN` item in the resource tree viewer in the middle
            - Select one of the available hosts from `MSDN` cluster
                - **IMPORTANT** Choice is arbitrary, however it is required that all devenv virtual machines are located on the same host
            - Click `Next`
        - In `Select storage` step  do the following:
            - Select `esxiX-main` datastore
                - `X` is a host identifier
                - If you chose `10.5.88.85` as your host, then `X = 1`
                - If you chose `10.5.88.86` as your host, then `X = 2`
            - Select `Thin provision` in `Select virtual dist format` select box
            - Click `Next`
        - In `Select clone options` step do the following:
            - Check `Customize this virtual machine's hardware (Experimental)`
            - Check `Power on virtual machine after creation`
            - Click `Next`
        - In `Customize hardware` step do the following:
            - For `Network Adapter 1` choose `v1800_VM` from the select menu
            - For `Network Adapter 1` check `Connect At Power On`
            - For `Network Adapter 2` choose `vS-DevEnvX-Y` from the select menu
                - If no such network is on the list, click `Show more networks...`
                - Network should be chosen based on steps from _Select private network for devenv_
            - Click `Next`
        - In `Ready to complete` step verify your settings and click `Finish`
3. Configure virtual machine
    - Select `VMs and Templates` tab in navigation pane on the left
    - Traverse to `Codilime/Dev/[DESTINATION]` folder
    - Left click on `[INITIALS]-tb1`
    - On virtual machine preview click a gear icon and select `Launch Remote Console`
        - If you do not have a VMware Remote Console installed, click `Install Remote Console` and install a provided MSI
    - Click `Send Ctrl+Alt+Del to virtual machine` button
    - Click on the interior to passthrough mouse and keyboard to the virtual machine
    - Login using provided credentials
    - Start PowerShell console
    - Change hostname

            Rename-Computer "[INITIALS]-tb1"

    - Disable DHCP on Ethernet1 and preserve address on restart

            Set-NetIPInterface -InterfaceAlias Ethernet1 -Dhcp Disabled -PolicyStore PersistentStore

    - Restart Ethernet1

            Restart-NetAdapter -InterfaceAlias Ethernet1

    - Set static IP address on dataplane adater Ethernet1 (based on addressing scheme from _Virtual machines addressing_)

            New-NetIPAddress -InterfaceAlias Ethernet1 -IPAddress "172.16.0.1" -PrefixLength 24

    - Verify if IP address is configured correctly

            Get-NetIPAddress -InterfaceAlias Ethernet1

    - Turn off the firewall

            Set-NetFirewallProfile -Enabled false

    - Restart virtual machine

            Restart-Computer

    - Check IP address of `Ethernet0` adapter

            Get-NetIPAddress -InterfaceAlias Ethernet0 -AddressFamily IPv4

        - Note down shown address; It will be required to connect to this virtual machine using Ansible

    - Press `Ctrl+Alt` to escape VMware Remote Console
    - Close VMware Remote Console

Now repeat these steps to create a second testbed `[INITIALS]-tb2`

## Controller setup

1. Clone virtual machine from `Codilime/__Templates/windows/centos7.4-bare` template
    - Steps are the same as in _Clone virtual machine from `Codilime/__Templates/windows/win2016core-bare` template_
    - This time use `[INITIALS]-ctrl`
3. Configure virtual machine
    - Select `VMs and Templates` tab in navigation pane on the left
    - Traverse to `Codilime/Dev/[DESTINATION]` folder
    - Left click on `[INITIALS]-ctrl`
    - On virtual machine preview click a gear icon and select `Launch Remote Console`
    - Click on the interior to passthrough mouse and keyboard to the virtual machine
        - If nothing is shown, start typing
    - Login using provided credentials
    - Change hostname

            # hostnamectl set-hostname "[INITIALS]-ctrl"

    - Configure static addressing on `ens224` adapter (`Network Adapter 2` from addressing scheme)
        - Edit file `/etc/sysconfig/network-scripts/ifcfg-ens224` to include following entries

                BOOTPROTO=static
                DEFROUTE=no
                IPADDR=172.16.0.10
                NETMASK=255.255.255.0

        - Restart networking

                systemctl restart network

        - Verify address configuration

                ip addr show

        - Check `ens192` address

                ip addr show ens192

            - Note down shown address; It will be required to connect to this virtual machine using Ansible

    - Reboot virtual machine

            systemctl reboot

## Run Ansible playbook

This section assumes the reader has Windows Subsystem for Linux (WSL) installed and configured.
If not, please refer to [this][WSL] document for install and configuration details.

All steps below should be done from WSL.

1. Install required packages

        sudo apt install python3 python3-pip python3-virtualenv sshpass

1. Clone `contrail-windows-ci` git repository from [https://github.com/Juniper/contrail-windows-ci](https://github.com/Juniper/contrail-windows-ci)

        mkdir ~/dev
        cd ~/dev
        git clone https://github.com/Juniper/contrail-windows-ci.git

1. Checkout `development` branch

        cd ./contrail-windows-ci
        git checkout development

1. Traverse to `contrail-windows-ci/ansible` directory

        cd ./ansible

1. Create virtualenv for Python

        python3 -m virtualenv -p /usr/bin/python3 venv

1. Active virtualenv
    - shell prompt should change to indicate that virtualenv is activated

            $ . venv/bin/activate
            (venv) $

1. Install Python dependencies

        pip install -r python-requirements.txt

1. Create a file for a provided Ansible vault key

        touch ~/ansible-vault-key
        vi ~/ansible-vault-key
        # Enter vault key

1. Copy `inventory.testenv` file to `inventory`

        cp inventory.testenv inventory

1. Add Windows virtual machines to `testbed` group by editing the `inventory` file

        [testbed]
        [INITIALS]-tb1 ansible_host=MGMT_IP_TB1
        [INITIALS]-tb2 ansible_host=MGMT_IP_TB2

    - `MGMT_IP_TB1`, `MGMT_IP_TB2` are addresses of `Ethernet0` adapters we checked earlier

1. Add Controller virtual machine to `controller` group

        [controller]
        [INITIALS]-ctrl ansible_host=MGMT_IP_CTRL ansible_user=PROVIDED_USER ansible_ssh_pass=PROVIDED_PASSWORD

    - `MGMT_IP_CTRL` is an address of `ens192` adapter we checked earlier
    - `PROVIDED_USER` and `PROVIDED_PASSWORD` are provided credentials for CentOS

1. Test network connection to devenv virtual machines

        ping MGMT_IP_TB1
        ping MGMT_IP_TB2
        ping MGMT_IP_CTRL

1. Test Ansible connectivity to Windows virtual machines

        ansible -i inventory testbed -m win_ping

1. Test Ansible connectivity to Controller virtual machine

        ansible -i inventory controller -m ping

1. Run `configure-local-testenv.yml`

        ansible-playbook -i inventory configure-local-testenv.yml --skip-tags yum-repos

    - Playbook should take at least an hour to complete

1. Verify if Contrail Controller is accessible
    - Access `https://MGMT_IP_CTRL:8143`

## To do

- Changes in `controller` role required to provision Linux compute node.

[WSL]: https://docs.microsoft.com/en-us/windows/wsl/install-win10
