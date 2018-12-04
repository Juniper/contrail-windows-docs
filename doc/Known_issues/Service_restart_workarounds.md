The purpose of this document is to describe problems with workarounds for restarting Contrail components on Windows.

## Docker/HNS networks issues

Windows Server 2016 with Docker 18.09 has an issue with associating HNS networks and Docker networks.

Restarting Docker service causes Docker networks to lose their association to corresponding HNS networks, if they were created using custom network plugins.
It is caused by lack of information regarding network plugin in HNS itself.
Networks created using Contrail CNM plugin are registered in HNS with type `transparent`.
During a restart, Docker removes networks created with custom plugins and after that pulls networks from HNS.
Thus, network <-> plugin association is lost.

## Problems

### Docker restart

- **Docker network removal**: removing networks after docker restart is not supported.
 Docker fills it's network database in incorrect way.
 Deleting all networks would delete a HNS root network leading to vRouter Agent crash,
 because it is dependent on this network (more specifically, on VMSwitch associated with this network).

### Compute node reboot

- **container deletion**: Docker doesn't allow deleting a network which has container attached to it, regardless of the container's state.
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

Notes on starting order:

- Agent has to start after CNM Plugin, because the plugin enables vRouter extension
- Because HNS networks are deleted, the starting order of CNM Plugin/Docker does not matter

### Issues

- **Leaked ports in controller**: After reboot cnm-plugin cannot recognize upon deletion of containers that some container had been connected to a contrail network.
The delete-endpoint request is not sent to the controller and it is polluted with outdated data.
- **Manual startup**: Docker's and Contrail components' services need to have startup type set to manual
