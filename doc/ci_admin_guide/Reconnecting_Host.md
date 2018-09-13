# Reconnecting Host to Contrail Windows CI cluster

This document describes a procedure required to reconnected a VMware ESXi host to the Contrail Windows CI cluster.
Reconnecting a host to CI cluster is understood as a set to steps leading to usage of this host in Contrail Windows CI.

Described procedure must be performed only when:

- a host with reinstalled ESXi must be reconnected to the cluster,
- a host with replaced hard drive must be reconnected to the cluster.

## Prerequisites

Requirements for CI admin:

- Access to Contrail Windows CI infrastructure.
- Credentials to Contrail Windows CI VMware cluster.
- Browser with Flash support, preferably Chrome.

Requirements for a reconnected host:

- Host must be connected to vCenter.

## Steps

1. Open the vSphere Web Client and login using credentials provided by Contrail Windows CI team.
1. In `Navigator` pane select `Hosts and clusters` tab.
1. Expand `CI-DC` datacenter entry.
1. Expand `WinCI` cluster entry.
1. Find a host entry representing a being reconnected.
    - If a host cannot be found, it is not connected to vCenter. In that case, please contact infrastructure team.
    - if a host is connected to vCenter, but is not a part of `WinCI` cluster:
        - Right click on the host and select `Move To...` option.
        - In the `Move To...` window, expand `CI-DC` datacenter.
        - Click on `WinCI` cluster.
        - Click `OK` button.
        - If a dialog `Move Host into This Cluster` comes up, select `Put all of this host's virtual machines in the cluster's root resource pool` option.
        - Click `Ok`.
1. If a host is marked as being in `maintenance mode`:
    - Right click on a host entry and select `Maintenance mode > Exit maintenance mode`.
1. Click on the host entry.
1. Host networking reconfiguration:
    - In the middle pane select `Configure` tab.
    - Select `Virtual switches` from the list on the left.
    - Select `vSwitch0` from the virtual switches list.
    - Click on `VM Network` port group in the bottom.
    - Click `Edit settings` button.
    - Type in `VM-Network` in the `Network label` input and click `Ok` button.
    - Select `VMkernel adapters` from the list on the left.
    - Select on `vmk0` adapter from the adapter list.
    - Click `Edit settings` button.
    - In `Port properties` wizard page, check `vMotion` checkbox. Click `Ok` button.
1. In `Navigator` pane select `Storage` tab.
    - Right click on `NFS-Datastore` and select `Mount Datastore to Additional Hosts`.
    - Mark a checkbox next to a host entry representing a being reconnected.
    - Click `OK`.
    - Right click on `winci_nfsbackup` and select `Mount Datastore to Additional Hosts`.
    - Mark a checkbox next to a host entry representing a being reconnected.
    - Click `OK`.
1. In `Navigator` pane select `Hosts and clusters` tab.
1. Find a host entry representing a being reconnected and click on it.
1. Cleanup of orphaned VMs must be performed.
    - In the middle pane select `VMs` tab.
    - In the VM table, click `Name` header to sort VMs by name in ascending order.
    - For each VM perform a following process:
        - Determine if the VM is critical.
            - Refer to [List of all important VMs in Windows CI][list-of-vms] and check if this VM is on the list marked as `CRITICAL`.
                - In case of any doubts, please contact Contrail Windows CI team.
        - If the VM is marked as `Orphaned` and it is not critical to CI functioning:
            - Left click on the VM.
            - In the middle pane, scroll down to `Related Objects` window and check if VM is still located on local datastore.
            - If it is the VM must be reregistered and then removed.
                - Please follow steps in [Reregistering a VM](#reregistering-a-vm) to reregister the VM.
                - Right click on the VM and select `Delete from disk` option.
            - If it is not, the VM can be safely removed from inventory.
                - Right click on the VM.
                - Select `All Virtual Infrastructure Actions > Remove from Inventory` option.
        - If a VM is marked as `Orphaned` and it is critical to CI functioning:
            - Left click on the VM.
            - In the middle pane, scroll down to `Related Objects` window and check if VM is still located on local datastore.
            - If it is the VM must be reregistered.
                - Please follow steps in [Reregistering a VM](#reregistering-a-vm) reregister the VM.
                - Power on the VM.
            - If it is not, the VM must be restored from backups.
                - Please refer to [Infrastructure backups][backups].
1. Assign host's datastores to datastore clusters.
    - In the middle pane select `Datastores` tab.
    - Please contact a Contrail Windows CI team regarding datastore - cluster association and perform the following steps:
        - Right click on a datastore.
        - Select `Move to` option.
        - In the `Move To...` window select a datastore cluster based on information from Contrail Windows CI team.


## Appendices

### Reregistering a VM

This guide assumes that the user is located in the VM window.

1. Take note of a datastore names presented in `Related Objects` windows.
1. Right click on the VM in the list on the left.
1. Select `All Virtual Infrastructure Actions > Remove from Inventory` option.
1. In the `Navigation` pane, left click `Storage` tab.
1. In the datastore list, click on the one of datastores listed previously in `Related Objects` window.
1. Navigate to `Files` tab.
1. Locate a folder named like a VM and enter it.
    - If it cannot be found on this datastore, try another one.
1. Right click on `[VM-NAME].vmx` file and select `Register VM` option.
    - The `Register Virtual Machine` wizard should show up.
    - In the `Name and Location` step select a suitable folder to put a VM in.
    - Click `Next`.
    - In the `Host / Cluster` step select a `WinCI` cluster.
    - Click `Next`.
    - In the `Ready to Complete` step click `Finish` button.

[backups]: https://github.com/Juniper/contrail-windows-docs/blob/master/doc/ci_admin_guide/Infrastructure_backups.md
[list-of-vms]: https://github.com/Juniper/contrail-windows-docs/blob/master/doc/ci_admin_guide/List_of_VMs_in_Windows_CI.md
