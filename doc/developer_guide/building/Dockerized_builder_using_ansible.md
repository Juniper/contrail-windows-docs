# Dockerized builder using ansible

This document describes what _would_ be needed create
a builder docker image (for building vrouter agent and extension)
by reusing [ansible] scripts that are already used in
the [contrail-windows-ci] repository to deploy builder machines.

This method of building is not yet supported, but this document
describes the steps needed to make it work. It also tries to
list the potential obstacles.

This document describes the process from the developer's point
of view, but it could be adapted to running in CI too.

[ansible]: https://www.ansible.com/
[contrail-windows-ci]: https://github.com/Juniper/contrail-windows-ci

## Windows

Currently, this process is supported only for Windows machines.

## Docker

Install [Docker Community Edition for Windows][docker-ce]. Note that:

* You need to sign in / create to accout to download the installer.
* You need to choose windows containers (not linux) when installing.

TODO: Is there another installation method?

[docker-ce]: https://store.docker.com/editions/community/docker-ce-desktop-windows

## Ansible

To execute ansible on Windows Docker containers you need to have:

1. A container image prepared for connecting via ansible to.
2. A (virtual) linux machine to run ansible.
3. (Optionally) To troubleshoot remote connection to container,
   you need to have the container IP in [trusted hosts lists][trusted-hosts].
   Depending on the configuration of your system, this may require
   changing the policy in your domain.

[trusted-hosts]: http://winintro.ru/windowspowershell2corehelp.en/html/f23b65e2-c608-485d-95f5-a8c20e00f1fc.htm

### Preparing container for ansible

Notes:

* Perhaps there exists a ready container prepared for ansible?
* There exists an example script [`ConfigureRemotingForAnsible.ps1`][configure-remoting].
  Unfortunately, the `microsoft/windowsservercore` image does not have the firewall
  service used by the script, so steps in this scripts need to be run manually.
  This wasn't finished, and it needs some additional investigation.

[configure-remoting]: https://github.com/ansible/ansible/blob/devel/examples/scripts/ConfigureRemotingForAnsible.ps1 

### Machine for ansible

The preferred way to use ansible on windows machine is to use
the Linux Subsystem on Windows. You can install [Ubuntu 18.04
from the Windows Store][wsl-ubuntu-store]. (18.04 version is
needed for the ansible 2.4 to be available. On the older
version you can probably install ansible using pip)

> Note: Alternatively, you can install ansible and its dependencies also
> using [this python-requirements.txt file][ansible-requirements-txt]
> from [contrail-windows-ci] repository with
> `pip install -r python-requirements.txt`

[ansible-requirements-txt]: https://github.com/Juniper/contrail-windows-ci/blob/development/ansible/python-requirements.txt

Install ansible >= 2.4 using `apt`:

```bash
sudo apt install ansible
```

There are additional python packages that need to be installed
to let ansible connect to Windows machine:

```bash
sudo apt install python-requests
sudo apt install python-pip
pip install pywinrm
pip install requests-credssp
```

The `group_vars/windows` also need to be configured for windows:
NOTE: `group_vars`, inventory and other scripts would be
already prepared, so this step won't be needed for the end user.

```yaml
ansible_port: 5986
ansible_connection: winrm
ansible_winrm_transport: credssp
ansible_winrm_server_cert_validation: ignore
ansible_winrm_operation_timeout_sec: 60
ansible_winrm_read_timeout_sec: 120
ansible_user: Administrator@localhost
ansible_password: 'hunter2'
```

[wsl-ubuntu-store]: https://www.microsoft.com/en-us/p/ubuntu-1804/9n9tngvndl3q

## Container components

### Build tools and .NET

Installation of .NET on `microsoft/windowsservercore` is not supported,
but microsoft provides the `microsoft/dotnet-framework` image.

TODO: Do we need to install multiple .NET versions? If not,
this image would be sufficient.

The build tools can be installed on the container following
the instruction on [this page in microsoft docs][build-tools-container].
Unfortunately, I haven't managed to finish it, due to some
HCS errors. Perhaps it was related to insufficient disk
space (before running the Dockerfile I had 50GB free space).
Perhaps there is some other way of installing build tools,
which I haven't tested.

[build-tools-container]: https://docs.microsoft.com/en-us/visualstudio/install/build-tools-container


## Troubleshooting

When encountering weird errors in `docker build`
(the errors may come from HCS), it's probable that they're caused by:

* Insufficient available RAM/swap on the host machine.
* Insufficient RAM assigned to container.
* Insufficient disk space on the host machine.
* Insufficient disk space assigned to container.
