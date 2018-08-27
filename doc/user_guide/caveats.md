# Known issues

* On Windows Server 2016, NAT network on compute node must be disabled in order for IP packet
fragmentation to work.
* Multiple endpoints per container are not supported.
* Multiple IPs/networks per container endpoint are not supported.
* `docker network inspect` on Contrail network does not show correct IPAM subnet, if subnet was not specified on creation
    * Please refer to this bug: https://bugs.launchpad.net/opencontrail/+bug/1789237
