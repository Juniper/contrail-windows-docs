# Devenv in CodiLime VMware lab

This document describes the steps required to provision a development environment in CodiLime VMware lab.
From high-level perspective steps are as follows:

1. Provision Windows and Linux virtual machines with suitable configuration in CodiLime VMware lab.
2. Configure Networking on Windows and Linux virtual machines using VMware console.
3. Run Ansible playbook to deploy Windows Contrail compute and Contrail Controller nodes.

Performing steps described below correctly should result in a configured devenv consiting of:

- 2 virtual machines configured as Windows Contrail compute, named `[INITIALS]-tb1` and `[INITIALS]-tb2`
- 1 virtual machine configured as Contrail Controller node named `[INITIALS]-ctrl`

## Prerequisites

### Select private network for devenv

Because of lacking infrastructure and licensing we simulate private networks based on PVLANs, using VMware's standard virtual switches.
This limits a communication in this private network to a single ESXi host.
Thus each virtual machine from a single devenv must be placed on a single ESXi host.

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
    - On Windows - `Ethernet1`
    - On CentOS - `ens190`
- `Network Adapter 2` corresponds to control adapter (Contrail's control and data plane)
    - On Windows - `Ethernet2`
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

        ```
        Rename-Computer "[INITIALS]-tb1"
        ```

    - Disable DHCP on Ethernet1 and preserve address on restart

        ```
        Set-NetIPInterface -InterfaceAlias Ethernet1 -Dhcp Disabled -PolicyStore PersistentStore
        ```

    - Restart Ethernet1

        ```
        Restart-NetAdapter -InterfaceAlias Ethernet1
        ```

    - Set static IP address on dataplane adater Ethernet1 (based on addressing scheme from _Virtual machines addressing_)

        ```
        New-NetIPAddress -InterfaceAlias Ethernet1 -IPAddress "172.16.0.1" -PrefixLength 24
        ```

    - Verify if IP address is configured correctly

        ```
        Get-NetIPAddress -InterfaceAlias Ethernet1
        ```

    - Turn off the firewall

        ```
        Set-NetFirewallProfile -Enabled false
        ```

    - Restart virtual machine

        ```
        Restart-Computer
        ```

    - Press `Ctrl+Alt` to escape VMware Remote Console
    - Close VMware Remote Console

Now repeat these steps to create a second testbed `[INITIALS]-tb2`

## Controller setup

1. Clone 1 VM from `centos7.4-bare` template
2. Login through VMware remote console
    - change hostname
    - configure static IP address od data plane adapter
    - reboot

## Setup inventory

1. Copy `inventory.testenv` to `my-inventory`
2. Add Windows virtual machines to `testbed` group

    ```
    [testbed]
    MGMT_IP_TB01
    MGMT_IP_TB02
    ```

3. Add Controller virtual machine to `controller` group

    ```
    [controller]
    MGMT_IP_CTRL ansible_user=root ansible_ssh_pass=PASSWORD
    ```

## Run playbook

1. Setup Ansible environment
2. Run `configure-local-testenv.yml`

    ```
    ansible-playbook -i my-inventory configure-local-testenv.yml --skip-tags yum-repos
    ```

## To do

- Changes in `controller` role required to provision Linux compute node.
