# Upgrading existing deployment

**Deprecation warning**: the need for cleanup phase will be removed in the future. This documentation section will be
updated when it happens.

**Warning**: this will remove all traces of Windows Contrail, along with any containers, networking and configuration.
Redeployment procedure will configure all the services correctly. However, the containers and networking must be
recreated manually by the user (according to [usage](./usage.md)).

Currently, upgrading Windows Contrail deployment consists of two steps:

1. cleanup of Compute nodes,
2. redeployment using Ansible playbooks.

This will result in rolling out the newest containers from opencontrailnightly repository.

## [FIXME] 1. Cleanup

Run `Clear-ComputeNode.ps1` script from [tools repository](https://github.com/Juniper/contrail-windows-tools) for each
Windows Compute node that needs upgrading.

**Note**: to quickly have the ability to run this script on your Windows machine, you can use the following snippet:
    
    Invoke-WebRequest  https://raw.githubusercontent.com/Juniper/contrail-windows-tools/master/Clear-ComputeNode.ps1 -OutFile Clear-ComputeNode.ps1

Consult the README on how to configure the script.



**Note**: you can use `Invoke-ScriptInRemoteSessions.ps1` script from
[tools repository](https://github.com/Juniper/contrail-windows-tools) to execute the cleanup script on multiple remote
nodes at the same time.

## 2. Redeploy

Follow the [deployment](./deployment.md) procedure as normal, supplying newer versions of Windows Contrail containers.
By default, deployment Ansible playbooks will pull the latest images.