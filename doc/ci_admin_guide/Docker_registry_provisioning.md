# How to provision Docker registry

> Note. This document describes provisioning procedure.
> If you want just to update the images, see the [updating manual][winci-registry-update].

To provision a docker registry, you will:

1. Prepare the machine
2. Install Docker
3. Start the registry
4. Pull and mirror the images

The following sections describe these steps in details.

## Preparing the machine

1.  Spawn a VM in VMWare using the CentOS / Ubuntu image as template (the rest of instruction will assume (CentOS 7).
    * Choose the machine name `winci-registry`
    * When asked about which customization to use, select DHCP.

2.  Log in using ssh and set the hostname:

    Edit the file `/etc/hostname` and type the same name as used in step 1.

3.  Update the system using `yum upgrade` and reboot.

4.  **Set up static IP**:

    Log in using ssh and change the IP configuration to manual, by modifying the following file:
    `/etc/sysconfig/network-scripts/ifcfg-ens192`: Change the line with `BOOTPROTO` to

        BOOTPROTO=static

    and add the following lines at the end of that file:

        IPADDR=10.84.12.27
        NETMASK=255.255.255.0
        GATEWAY=10.84.12.254
        DNS1=172.21.200.60
        DNS2=172.29.131.60
        DNS3=8.8.8.8
        DOMAIN="contrail.juniper.net englab.juniper.net juniper.net jnpr.net"

    This IP is used by the `controller` role in our CI,
    [as a `docker_registry` parameter][controller-docker-registry-param]

5.  Then reboot the machine and log in using the new IP.

[controller-docker-registry-param]: https://github.com/Juniper/contrail-windows-ci/blob/development/ansible/roles/controller/defaults/main.yml#L1

## Installing Docker

1.  Install Docker using package manager

        yum install docker
        systemctl enable docker
        systemctl start docker

2.  Allow the `ci-repo` (from which we will be pulling the images) to use HTTP:

    *   Add this line to `/etc/docker/daemon.json` (if the file is not there, create it):

            { "insecure-registries":["ci-repo.englab.juniper.net:5000"] }{}

    *   Restart docker to apply changes

            systemctl restart docker

## Starting the registry


### Starting the registry

Start the registry container:

```bash
docker run -d -p 5000:5000 --restart=always --name registry registry:2
```

The container with the registry is set to autostart, so no further
configuration is needed.

## Pulling and mirroring the images

Refer to [the registry update manual][winci-registry-update] to pull and start mirroring the images.

## Possible improvements

Perhaps we can also try mirroring base images for speed.

## References

1. [Deploy a registry server — Docker Documentation][docker-registry-deploying]

[docker-registry-deploying]: https://docs.docker.com/registry/deploying/
[winci-registry-update]: Update_private_docker_registry.md
