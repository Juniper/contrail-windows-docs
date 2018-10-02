# Contrail deployment in Windows CI

## Environment

![Windows CI Environment](Environment.png)

## Playbooks

* provision_instances.yml - creates server instances (e.g. on AWS) not used in Contrail Windows CI
* configure_instances.yml - installs dependencies installs and starts docker
* install_contrail.yml - installs Contrail and installs orchestrator (optional)

## Configuration

* Orchestrators:
  * None
  * OpenStack
  * VCenter
* Contrail version and kolla version:
  * ocata-master-91 and ocata
* Docker registry
  * ci-repo.englab.juniper.net or [opencontrailnigthly](https://hub.docker.com/u/opencontrailnightly/)
* Interfaces
  * Example [config](https://github.com/Juniper/contrail-ansible-deployer/blob/master/config/)
  * Windows-CI [config](https://github.com/Juniper/contrail-windows-ci/blob/development/ansible/roles/contrail-ansible-deployer/templates/instances.j2)

## Kolla

* Kolla == production-ready containers and deployment tools for operating OpenStack
* contrail-ansible-deployer internally clones Juniper/contrail-kolla-ansible (branch: contrail/ocata)
* Bugs which may affect us can be introduced in contrail-ansible-deployer as well as in contrail-kolla-ansible

## Base image

* CentOS 7.4 (kernel >= 3.10.0-693.17.1)
* Two interfaces:
  * Control Plane
  * Data Plane
* IP address is required on every interface - can be static
* More information:
  * https://github.com/Juniper/contrail-ansible-deployer#prerequisites

## Windows-CI

![Controller Deployment Flow](ControllerDeployment.png)

## Troubleshooting

* Try to run playbook locally from laptop
* Try to look for similar issue on contra-cloud.slack channel: contrail-ansible
* Check recent commits in contrail-ansible-deployer and contrail-kolla-deployer
* Ask our devops team
* Ask on slack channel

## Zuul V3

* Controller image from pull request
* Correct version of contrail-ansible-deployer
* Correct version of contrail-kolla-deployer
* Enable voting on contrail-ansible-deployer and contrail-kolla-deployer