# RFC for CI roles extraction
### An idea to extract common roles for deploying controller from contrail-project-config to be used by Contrail-Windows and DevOps teams.

Author: Katarzyna Rybacka

---


## Abstract

Currently, controller5.0 is being deployed in Contrail-Windows CI using roles copied from `contrail-project-config` repository([Link](https://github.com/Juniper/contrail-project-config)).
There is a need to make this code more generic so it could be used by few teams simultaneously.
This document specifies the proposed solution to this issue and requests discussion and suggestions for
improvements.


## Introduction

In CI, specifically in `contrail-windows-ci` repository ([Link](https://github.com/Juniper/contrail-windows-ci)) there are three roles `contrail-ansible-deployer`, `kolla-provision-dockers`
and `yum-repos-prepare` used to deploy controller5.0. The fact that this code is copied from another repository
could cause some issues e.g. need for backporting code, when any change occures in source code.


## Proposal

Recently, in `contrail-ansible-deployer` repository ([Link](https://github.com/Juniper/contrail-ansible-deployer)) playbooks to deploy Contrail and OpenStack appeared.
One config file instances.yml is needed for proper deploy and its inside is described in `Involved steps` section.
Idea is to change Windows Contrail CI to use playbooks from this repository like the DevOps team does. That would be a single instance of code which could be shared by many.
Also there is a need to change deprecated role `yum-repos-prepare` to `configure-mirrors` from `zuul-jobs` repository ([Link](https://github.com/Juniper/zuul-jobs)).
All action steps are proposed to be performed by members of Contrail-Windows team.


### Involved steps

1. Remove roles `contrail-ansible-deployer`, `kolla-provision-dockers` and `yum-repos-prepare` from `contrail-windows-ci` repository ([Link](https://github.com/Juniper/contrail-windows-ci))
2. Change controller role to use contrail-ansible-deployer repository ([Link](https://github.com/Juniper/contrail-ansible-deployer)) (basically clone it) and `configure-mirrors` role.
3. Create config file instances.yml for deployment of OpenStack and Contrail, which should contain vars from roles used before e.g.:

        instances:
            bms1:
                provider: bms
                ip: "{{ ansible_all_ipv4_addresses[0] }}"
                roles:
                config_database:
                config:
                control:
                analytics_database:
                analytics:
                webui:
                vrouter:
                openstack:
        contrail_configuration:
            CLOUD_ORCHESTRATOR: "{{ cloud_orchestrator }}"
            : "{{ contrail_docker_registry }}"
            CONTRAIL_VERSION: "{{ contrail_version }}"
            CONTROLLER_NODES: "{{ ansible_all_ipv4_addresses[0] }}"
            LINUX_DISTR: centos7
            LOG_LEVEL: SYS_DEBUG
            OPENSTACK_VERSION: "{{ openstack_version }}"
            PHYSICAL_INTERFACE: "{{ physical_interface }}"
            VROUTER_GATEWAY: "{{ ansible_default_ipv4.gateway }}"
         {% if cloud_orchestrator == 'openstack' %}
            AUTH_MODE: keystone
            KEYSTONE_AUTH_ADMIN_PASSWORD: c0ntrail123
            KEYSTONE_AUTH_HOST: "{{ ansible_all_ipv4_addresses[0] }}"
            KEYSTONE_AUTH_URL_VERSION: "/v3"
            RABBITMQ_PORT: 5673
        {% endif %}
        kolla_config:
            kolla_globals:
                network_interface: "{{ network_interface }}"
                api_interface: "{{ network_interface }}"
                neutron_external_interface: "{{ network_interface }}"
                kolla_external_vip_interface: "{{ network_interface }}"
                enable_haproxy: no
                openstack_service_workers: 1
                openstack_release: "{{ kolla_version }}"
                docker_registry: "{{ kolla_docker_registry.fqdn }}:{{ kolla_docker_registry.port }}"
                neutron_opencontrail_init_image_full: "{{ kolla_docker_registry.fqdn }}:{{ kolla_registry_port }}/contrail-openstack-neutron-init:{{ contrail_version }}"
                nova_compute_opencontrail_init_image_full: "{{ kolla_docker_registry.fqdn }}:{{ kolla_registry_port }}/contrail-openstack-compute-init:{{ contrail_version }}"

4. Add vars mentioned above ( in `{{ }}` ) to the controller role.
5. Use `configure_instances.yml` and `install_contrail.yml` to deploy Contrail and Openstack.
 
