# How to provision Docker registry

> Note. This document describes provisioning procedure.
> If you want just to update the images, see the [updating manual][winci-registry-update].

## Preparing machine

Spawn a VM using the CentOS / Ubuntu image (the rest of instruction will assume (CentOS 7),
use DHCP and name `winci-registry`

Log in using ssh and set the hostname in `/etc/hostname`.

Update the system using `yum upgrade` and reboot.

### Networking

Log in using ssh and change the IP configuration to manual, by modifying the following file:
`/etc/sysconfig/network-scripts/ifcfg-ens192`. Change the line with `BOOTPROTO` to

```ini
BOOTPROTO=static
```

and add the following lines at the end of that file:

```ini
IPADDR=10.84.12.27
NETMASK=255.255.255.0
GATEWAY=10.84.12.254
DNS1=172.21.200.60
DNS2=172.29.131.60
DNS3=8.8.8.8
DOMAIN="contrail.juniper.net englab.juniper.net juniper.net jnpr.net"
```

This IP is used by the `controller` role in our CI,
[as a `docker_registry` parameter][controller-docker-registry-param]

Then reboot the machine and log in using the new IP.

[controller-docker-registry-param]: https://github.com/Juniper/contrail-windows-ci/blob/development/ansible/roles/controller/defaults/main.yml#L1

## Installing docker

```bash
yum install docker
systemctl enable docker
systemctl start docker
```

## Configuring registry

### Starting the registry

First, start the registry container:

```bash
docker run -d -p 5000:5000 --restart=always --name registry registry:2
```

### Mirroring or updating docker images

The update procedure is documented [in a separate document][winci-registry-update].

### Further steps

Perhaps we can also try mirroring base images for speed.

## References

1. [Deploy a registry server — Docker Documentation][docker-registry-deploying]

[docker-registry-deploying]: https://docs.docker.com/registry/deploying/
[winci-registry-update]: Update_private_docker_registry.md
