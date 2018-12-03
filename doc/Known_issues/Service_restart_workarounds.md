The purpose of this document is to describe problems and workarounds for restarting Contrail components on Windows.

## Docker/HNS networks issues

Windows Server 2016 with Docker 18.09 has an issue with associating HNS networks and Docker networks

Restarting Docker service causes Docker networks to lose their association to corresponding HNS networks, if they were created using custom network plugins.
It is caused by lack of information regarding network plugin in HNS itself.
Networks created using Contrail CNM plugin are registered in HNS with type `transparent`.
During a restart, Docker removes networks created with custom plugins and after that pulls networks from HNS.
Thus, network <-> plugin association is lost.

## Problems

### Docker restart

- `docker network prune`: using this command after docker restart isn't supported.
 Because Docker fills it's network database in incorrect way,
 using this command could delete a HNS root network which could lead to e.g. BSOD.

### Compute node reboot

- container deletion: Docker doesn't allow deleting a network which has container attached to it, regardless of the container's state.
 As the networks states are invalid after reboot, both networks and containers need to be deleted.

## Workaround: contrail-autostart

### Description

contrail-autostart is a script invoked at the startup of Windows whose job is to workaround weird Docker for Windows behaviour related to rebooting.

contrail-autostart does the following:

- Removes remaining networks from HNS
- Removes containers
- Starts docker and contrail services:
    - Order:
        - Docker
        - CNM Pugin
        - vRouter Agent
    - Agent has to start after CNM Plugin, because the plugin enables vRouter extension
    - Because HNS networks are deleted, the starting order of CNM Plugin/Docker doesn't matter

### Issues

- uncleaned ports in controller: After reboot cnm-plugin can't recognize upon deletion that some container had been connected to a contrail network.
The delete-endpoint request isn't sent to the controller and it's polluted with outdated data.
- manual startup: Docker's and Contrail components' services need to have startup type set to manual
