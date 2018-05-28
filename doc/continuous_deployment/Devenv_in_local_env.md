# Devenv in local environment

This document describes the steps required to provision a development environment in local VMware environment.

## Testbed setup

1. Clone 2 VMs from `win2016core-bare` template
    - single ESXi host
    - destination folder: `Dev\[initials]`
    - datastore: `esxi*-main`
    - management network: `v1800_VM`
    - data plane network: `vS-DevEnvX-Y`
        - `X` corresponds to host
        - `Y` is a network number
2. Login through VMware remote console
    - change hostname
    - configure static IP address on data plane adapter
    - reboot

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
