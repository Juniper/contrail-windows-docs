# Prepare an ESXi Host for Maintenance

This document describes steps required to prepare an ESXi host for the maintenance.
Examples of such maintenance include:

- Replacing broken/old hard drives.
- Reinstalling ESXi system.


## Prerequisites

Requirements for CI admin:

- Access to Contrail Windows CI infrastructure.
- Credentials to Contrail Windows CI VMware cluster.
- Browser with Flash support, preferably Chrome.

Before proceeding with the following steps take note of:

- ESXi host hostname
- ESXi local datastore list

In this document, the ESXi host being prepared for the maintenance will be referred to as _maintaned host_.


## Steps

1. Open the vSphere Web Client and login using credentials provided by Contrail Windows CI team.
1. In `Navigator` pane select `Storage` tab.
1. Expand `WinCI-Datastores-SSD` datastore cluster.
    - Identify local datastores of maintained host.
    - For each such datastore perform:
        - Right click on the datastore.
        - Click `Move To...` option.
        - In the `Move To...` window, select `CI-DC`.
        - Click `Ok` button.
1. Perform the previous step for other datastore clusters:
    - `WinCI-Datastores-SATA`
    - `WinCI-Datastores-Infra`
1. NOTE: Testbeds in CI are spawned on datastores from `WinCI-Datastores-SSD` datastore cluster. Removing host's datastores from it denies new testbeds from being spawned on this host.
1. In `Navigator` pane select `Hosts and Clusters` tab.
1. Expand `CI-DC` datacenter entry.
1. Expand `WinCI` cluster entry.
1. Find the maintained host entry click on it.
1. Migrate virtual machines off the maintained host.
    - Left click on the host entry in the `Navigator` pane.
    - In the main pane select `VMs` tab.
    - For each VM in the table perform the following steps:
        - Right click on the VM.
        - Select `Migrate...` option.
        - `Migrate` wizard should open up.
        - In `Select the migration type` step:
            - Select `Change both compute resource and storage` option.
            - Select `Select compute resource first` option.
            - Click `Next` button.
        - In `Select a compute resource` step:
            - Expand `CI-DC` datacenter entry.
            - Expand `WinCI` cluster entry.
            - Select a host different than maintained host.
            - Click `Next` button.
        - In `Select storage` step:
            - Select a datastore local to host selected in the previous step.
            - Click `Next` button.
        - In `Select networks` step:
            - Verify that `Source Network` matches `Destination Network` in the presented table.
            - Click `Next` button.
        - In `Select vMotion priority` step:
            - Select `Schedule vMotion with high priority` option.
            - Click `Next` button.
        - In `Ready to complete` step:
            - Click `Finish` button.
1. Migrate templates.
    - Left click on the host entry in the `Navigator` pane.                                                                                                                                                                               47     - In the main pane select `VMs` tab.
    - In the main pane select `VMs` tab.
    - Below `VMs` tab, select `VM Templates if Folders`.
    - For each template perform the following steps:
        - Right click on the template.
        - Select `Convert to Virtual Machine...` option.
        - `Convert Template to Virtual Machine` wizard should open up:
        - In `Select a compute resource` step:
            - Click on `WinCI` cluster object.
            - In the bottom of the windows, you should see `Compatibility checks succeeded`.
            - Click `Next` button.
        - In `Ready to Complete` step:
            - Click `Finish` button.
        - Change screen from `VM Templates in Folders` to `Virtual Machines`.
        - If the template is listed on the VM list:
            - Perform the same migration steps as for virtual machines from step 11.
        - If the template is not listed:
            - It probably was already located on shared storage and VMware reassigned it to the different host.
            - To confirm search for this template in `VMs and Templates` tab, in `Navigator` pane.
        - Right click on this relocated/migrated VM and select `Template > Convert to Template` option.
            - In the `Confirm Convert` windows, click `Yes` button.
        - **NOTE**: `testbed` templates should stay on `NFS-Datastore`, since template location on shared storage is a requirement of linked clones.
        - **NOTE**: If enough space is free on `NFS-Datastore`, latest templates should be located on it.
1. Switch host to the maintenance mode.
    - Right click on the host entry in the `Navigator` pane.
    - Select `Maintenance Mode > Enter Maintenance Mode` option.
    - In the `Confirm Maintenance Mode` window, uncheck `Move powered-off and suspended virtual machines to other hosts in the cluster` option.
    - Click `OK` button.
1. Move host out of the `WinCI` cluster.
    - Right click on the host entry in the `Navigator` pane.
    - Select `Move To...` option.
    - Click on `CI-DC` datacenter object.
    - Click `OK` button.
