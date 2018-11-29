Windows Server 2016 with Docker 18.09 has several issues with associating HNS networks and Docker networks.

- Restarting Docker service causes Docker networks to lose their association to corresponding HNS networks, if they were created using custom network plugins.
  It is caused by lack of information regarding network plugin in HNS itself.
  Networks created using Contrail CNM plugin are registered in HNS with type `transparent`.
  During a restart, Docker removes networks created with custom plugins.

## Problems
