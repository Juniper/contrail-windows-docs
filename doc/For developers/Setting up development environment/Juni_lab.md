# Dev environment in Juniper lab

Windows CI, as an additional feature, allows for deployment of devenvs similar to the ones used in
CI. This guide describes how to do it.

## Prerequisites

* Access to Juniper network.
* Account for Windows CI Jenkins.
* Account for Windows CI VMWare cluster.

## Steps

### 1. Choosing VLAN for the demo setup1.

**TODO** this is tedious and should be automated in the future.

1. Navigate to VCenter (currently at https://ci-vc.englab.juniper.net/ui)
2. Log in
3. Go to "Network" tab → CI-DC → dvs-ContrailWinCI
4. Click through VLANs 502-505 and find one that has "Virtual Machines: 0"\
    **Note** it is very important that you don't cause any collisions with other users of dev environments.

### 2. Running Jenkins job

1. Navigate to Jenkins webui (currently at http://10.84.12.26:8080/)
2. Log in
3. Navigate to `WinContrail-infra/deploy-demo-env`
4. Press `Build with Parameters`:
    * DEMOENV_NAME - choose a short prefix for your VMs hostnames
    * DEMOENV_VLAN - choose an empty VLAN (the one you've found)
5. Wait for 15 minutes
6. Deploy (install) artifacts manually

### 3. Destroying demo env

1. After you're done using dev environment, remove all virtual machines created by deploy-demo-env
2. Navigate to Jenkins and log in (as described above)
3. Navigate to `WinContrail-infra/destroy-demo-env`
4. Press `Build with Parameters`, giving exactly the same DEMOENV_NAME as was used for deploying this demo env.

