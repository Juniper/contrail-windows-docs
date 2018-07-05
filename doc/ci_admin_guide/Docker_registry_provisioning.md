# Provisioning Docker registry

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

### Pulling images

First, we need to add `ci-repo` to insecure registries, as it uses HTTP:

Add this line to `/etc/docker/daemon.json` (if the file is not there, create it):

```json
{ "insecure-registries":["ci-repo.englab.juniper.net:5000"] }{}
```

Restart docker:
```bash
systemctl restart docker
```

Then, we need to determine a list of images to pull. You can run this command
on any working controller (*not on a registry machine!*):

```bash
docker image list | grep ci-repo | awk '{print $1 ":" $2}' > pullme
```

<details><summary>Example list of images (updated 2018-06-22)</summary>
```
ci-repo.englab.juniper.net:5000/kolla/centos-binary-glance-api:ocata
ci-repo.englab.juniper.net:5000/kolla/centos-binary-glance-registry:ocata
ci-repo.englab.juniper.net:5000/kolla/centos-binary-nova-compute:ocata
ci-repo.englab.juniper.net:5000/kolla/centos-binary-nova-placement-api:ocata
ci-repo.englab.juniper.net:5000/kolla/centos-binary-nova-novncproxy:ocata
ci-repo.englab.juniper.net:5000/kolla/centos-binary-nova-api:ocata
ci-repo.englab.juniper.net:5000/kolla/centos-binary-nova-conductor:ocata
ci-repo.englab.juniper.net:5000/kolla/centos-binary-nova-consoleauth:ocata
ci-repo.englab.juniper.net:5000/kolla/centos-binary-nova-ssh:ocata
ci-repo.englab.juniper.net:5000/kolla/centos-binary-nova-scheduler:ocata
ci-repo.englab.juniper.net:5000/kolla/centos-binary-keystone:ocata
ci-repo.englab.juniper.net:5000/kolla/centos-binary-neutron-server:ocata
ci-repo.englab.juniper.net:5000/kolla/centos-binary-neutron-metadata-agent:ocata
ci-repo.englab.juniper.net:5000/kolla/centos-binary-swift-account:ocata
ci-repo.englab.juniper.net:5000/kolla/centos-binary-swift-object:ocata
ci-repo.englab.juniper.net:5000/kolla/centos-binary-swift-rsyncd:ocata
ci-repo.englab.juniper.net:5000/kolla/centos-binary-swift-object-expirer:ocata
ci-repo.englab.juniper.net:5000/kolla/centos-binary-swift-proxy-server:ocata
ci-repo.englab.juniper.net:5000/kolla/centos-binary-swift-container:ocata
ci-repo.englab.juniper.net:5000/kolla/centos-binary-barbican-api:ocata
ci-repo.englab.juniper.net:5000/kolla/centos-binary-barbican-keystone-listener:ocata
ci-repo.englab.juniper.net:5000/kolla/centos-binary-barbican-worker:ocata
ci-repo.englab.juniper.net:5000/kolla/centos-binary-heat-api:ocata
ci-repo.englab.juniper.net:5000/kolla/centos-binary-heat-api-cfn:ocata
ci-repo.englab.juniper.net:5000/kolla/centos-binary-heat-engine:ocata
ci-repo.englab.juniper.net:5000/kolla/centos-binary-horizon:ocata
ci-repo.englab.juniper.net:5000/kolla/centos-binary-swift-base:ocata
ci-repo.englab.juniper.net:5000/kolla/centos-binary-memcached:ocata
ci-repo.englab.juniper.net:5000/kolla/centos-binary-cron:ocata
ci-repo.englab.juniper.net:5000/kolla/centos-binary-rabbitmq:ocata
ci-repo.englab.juniper.net:5000/kolla/centos-binary-kolla-toolbox:ocata
ci-repo.englab.juniper.net:5000/kolla/centos-binary-nova-libvirt:ocata
ci-repo.englab.juniper.net:5000/kolla/centos-binary-fluentd:ocata
ci-repo.englab.juniper.net:5000/kolla/centos-binary-mariadb:ocata
ci-repo.englab.juniper.net:5000/contrail-openstack-neutron-init:ocata-master-91
ci-repo.englab.juniper.net:5000/contrail-openstack-compute-init:ocata-master-91
```
</details>

Then, copy this file to the registry, and pull the images:

```bash
for image in `cat pullme`; do docker pull $image; done
```

### Starting the registry

> NOTE: This section is adapted from [Docker registry deploying instruction][docker-registry-deploying].

First, start the registry container:

```bash
docker run -d -p 5000:5000 --restart=always --name registry registry:2
```

Then, we need to tag the images we've pulled and push them to the registry:

Note: this example uses the `ocata` and `ocata-master-91` tags
(see the example `pullme` list above).
You need to replace occurrences of `ocata` and `ocata-master-91`
in the following scripts, according to the images you've pulled.

Kolla images:

```bash
for image in $(docker images | grep ci-rep | grep kolla | awk '{print $1}'); do
    name=$(echo $image | cut -f2- -d/);
    docker tag $image:ocata localhost:5000/$name:ocata;
    docker push localhost:5000/$name;
done
```

Other images:

```bash
for image in $(docker images | grep ci-rep | grep ocata-master | awk '{print $1}'); do
    name=$(echo $image | cut -f2- -d/);
    docker tag $image:ocata-master-91 localhost:5000/$name:ocata-master-91;
    docker push localhost:5000/$name;
done
```

> TODO: Don't hardcode the tags in these hacky for loops

### Further steps

Perhaps we can also try mirroring base images for speed.

[docker-registry-deploying]: https://docs.docker.com/registry/deploying/
