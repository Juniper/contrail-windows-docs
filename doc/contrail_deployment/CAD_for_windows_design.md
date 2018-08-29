# Contrail-ansible-deployer for Windows design document
This is a document describing initial support of deploying Windows compute nodes with contrail-ansible-deployer.

## Ansible for Windows
  * A lot of ansible modules don't work on Windows. Possible workarounds:
    * Use modules with `win_` prefix if they exist, however a lot of modules don't have Windows equivalent,
    * Mimic function of not working module with other ways possible (e.g. using `win_shell`).
## OS-specific roles:
  * There are particular roles which have to be done differently for Windows.
  * Thus, these roles are divided into `*_Linux` and `*_Win32NT` for usage in c-a-d for Linux and c-a-d for Windows respectively.

## Orchestrators:
  * OpenStack must be specified as an orchestrator, because Keystone is required by Windows Docker driver for authorization.

## Playbooks:
  * `provision_instances.yml` - not supported at this moment.
  * `configure_instances.yml`
  * `install_openstack.yml`
  * `install_contrail.yml`

## configure_instances:
  * Some differences between Windows and non-Windows c-a-d workflows exist for this playbook:
    * Particular packages must be present on ansible host machine used for secure communication between the ansible machine and Windows nodes. The packages are installed by this role.
    * Windows compute nodes need Windows-specific dependencies and particular features turned on/off. `install_software_Win32.yml` is responsible for that. This role makes following changes on Windows nodes:
      * Installs following software:
        * Windows-Containers
        * NET-Framework-Features
        * Hyper-V
        * NSSM
        * Docker
        * MS Visual C++ Redistributable
      * Sets these Windows features:
        * ON:
          * Test-Signing Mode - reason explained below
        * OFF:
          * WinNAT
          * Windows Firewall
  * In the future, vRouter kernel module will be digitally signed, but for now, it uses self-signed certificates.
    To do that, Windows Test-Signing Mode must be enabled and reboot is done to apply the change.

## install_openstack:
  * There are no changes to this playbook.
  * Whole OpenStack is installed for now, but in the future there should have to specify only the role in `config/instances.yaml` which is responsible for installing Keystone.

## install_contrail:
  * Specific dlls must be present in main Windows system directory (`C:/Windows/System32`) for the software to run.
  * Because on Windows only Contrail compute nodes are supported, there are 2 roles for Windows compute nodes:
    * `vrouter` - installs vRouter kernel module, vRouter agent and utils (`create_vrouter_Win32NT.yml`):
      * Pull `contrail-windows-vrouter` image with artifacts. Artifacts' directory structure:
        * `C:/Artifacts/`
          * `agent/`
            * `contrail-vrouter-agent.msi` - installs vRouter agent and creates `ContrailAgent` service;
          * `vrouter/`
            * `utils.msi` - installs vRouter utils;
            * `vRouter.msi` - installs vRouter kernel module and creates `vRouter` service;
            * `vRouter.cer` - certificate for vRouter kernel module;
          * `vtest/*` - vtest utility (needed in Windows CI);
    * `win_docker_driver` - installs Windows Docker driver (`create_win_docker_driver.yml`):
      * Pulls `contrail-windows-docker-driver` image with artifact. Artifact's directory structure:
        * `C:/Artifacts/`
          * `docker-driver/docker-driver.msi` - installs Docker driver and creates `DockerDriver` service;
      * From high-level perspective Docker driver is very similiar to OpenStack Nova Agent, however they differ significantly at low-level, because:
        * The driver needs to have communication with Windows-specific modules,
        * Docker driver communicates directly with Contrail config node.
    * The artifacts have to be built on Windows. Built artifacts are pulled from docker registry specified in `WINDOWS_CONTAINER_REGISTRY` in `config/instances.yaml`.
  * On Windows contrail-ansible-deployer starts components as Windows services,
    in contrast to Linux where they reside in separate containers.
    * Reasons:
      * It's not possible to run Linux containers on Windows Server 2016,
      * Since priviliged containers do not exist on Windows, vrouter-agent wouldn't be able to access shared memory for flow and bridge tables
      * Docker for Windows Server 2016 doesn't support passing named pipes from host to containers. vRouter *pkt0* and *ksync* interfaces are implemented with named pipes on Windows.
    * There are plans to move to containers in the future.
