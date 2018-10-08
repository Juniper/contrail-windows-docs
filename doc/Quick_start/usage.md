# Usage

The following section describes usage of Windows Contrail.

## Prerequisites

Make sure that Windows Compute node was deployed successfuly. See [deployment instructions](./deployment.md).

## Creating a network

In current workflow, virtual network (with IP pool) must be created manually in
Contrail, e.g. using WebUI. Once it's created, run the following command on Windows
Compute node to create a local Contrail network:

    docker network create --ipam-driver windows --driver Contrail --opt tenant=$tenant --opt network=$network --subnet $subnet $localnetwork

where:

* `$tenant` is the name of your tenant in Contrail,
* `$network` is the name of your network in Contrail,
* `$subnet` is subnet CIDR (e.g. `10.0.0.0/24`). This parameter must be specified, if you Contrail network has multiple subnets defined.
* `$localnetwork` is an arbitrary name that docker network should have on local compute node.

Parameter `--ipam-driver windows` is specified to override docker's default IPAM
driver with a noop one (named `windows`). CNM plugin assigns IP address in a
different way.

Example:

    docker network create --ipam-driver windows --driver Contrail --opt tenant=admin --opt network=rednetwork --subnet 10.0.0.0/24 my_local_red_net

## Removing a network

To remove a network, simply use `docker network remove` command. The network cannot
have active endpoints for it to be removed.

**Note**: this operation does **not** remove a virtual network in Contrail. To
remove it as well, use Contrail WebUI.

## Creating a container

When creating a container on Windows Compute, specify `--net` parameter:

    docker run -d --net $network microsoft/nanoserver ping -t localhost

where:

* `$network` is a name of previously created local Contrail network.

Unlike when creating a virtual network, manual creation of any resources for a container
in Contrail is not required.

Example:

    docker run -d --net my_local_red_net microsoft/nanoserver ping -t localhost

## Removing a container

To remove a container, use `docker rm` command, as normal.
