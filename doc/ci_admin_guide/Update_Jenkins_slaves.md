# Updating Jenkins slaves

## Rolling out new slaves

TODO
- update templates
- create templates
- spawn new slaves
- decomission old slaves
- tags
- how to test if it worked safely

## Updating existing slaves

Some minor maintenance work does not require rollout of new slaves. An example might be an
upgrade of some library or other dependency. In this case, one can update the running cluster
without having to create a new template and rolling out every machine. However, the template must
be updated at the end of the procedure, so that future fresh machines will also have the
required changes.

The upgrade procedure consists of:
1. Manually upgrade a subset of nodes
1. Test if CI job pass on the new set of nodes
1. Upgrade the rest of the nodes

### 1. Manually upgrade a subset of nodes

1. Select one Jenkins slave to use.
    1. Make sure that there are other slaves that perform the same function (they have the same tags and are up).
    1. In Jenkins UI, press `Mark this node as temporarily offline`, providing the reason.
    1. Note the IP address of the slave.
1. Manually run ansible against the node.

    > NOTE: this is a one-off role. It doesn't necessarily need to be commited to
    version control. Therefore, any "hacks" are allowed. For example, to upgrade
    a chocolatey package, in addition to updating the `version` parameter of
    `win_chocolatey` role, one must also run an additional task of uninstalling
    the package first. See examples below.

    1. Checkout `github.com/Juniper/contrail-windows-ci` repository.
    1. Enter `ansible/` directory.
    1. Prepare your environment by going through steps described in `README.md`.
    1. Find the ansible role that is responsible for the upgrade.
        1. Modify the role to suit your needs.
        1. If modification could cause a slave restart, manually disable auto start of `jenkins_swarm_client`:

            ```powershell
            Stop-Service jenkins_swarm_client
            Set-Service jenkins_swarm_client -StartupType Manual
            ```

        1. __NOTE__: Auto start of `jenkins_swarm_client` after a reboot removes the `Mark this node as temporarily offline` status from the slave.
        1. Add a tag to every task that needs to be executed.
    1. Prepare inventory file.
        1. Edit file `inventory.devel/groups`.
        1. Add IP of your selected slave under the correct group.
    1. Create one-off playbook.
        1. Create an yml file in the root of `ansible/` dir.
        1. Specify hosts and roles to be executed.
    1. Run the playbook, specifying:
        - ansible vault key location,
        - inventory,
        - task tags to execute,
        - prepared playbook.
1. Test if the change worked.
    1. Remote into the machine and verify if change was successful.
    1. Test if job passes.
        1. In Jenkins, go into the selected node view. Remove all tags it has. Add some temporary tag (e.g. 'temp-upgrade').
        1. Open a test pull request to `github.com/Juniper/contrail-windows-ci`
            1. In its Jenksinfile, replace tags of nodes that you removed on t he node with the temporary tag.
            1. Open the pull request with "do not merge" in the title.
            1. Wait for CI to be triggered normally. Wait for it to pass.
    1. If autostart of `jenkins_swarm_client` was disabled:

        ```powershell
        Set-Service jenkins_swarm_client -StartupType Automatic
        Start-Service jenkins_swarm_client
        ```

    > NOTE: the following steps may change depending on case-by-case basic.

1. Open a normal pull request with cleaned up changes to ansible roles.
1. Get it merged.
1. Create a new builder template. (see [this](#rolling-out-new-slaves))
1. Rollout the change to other slaves by repeating steps 1-2 but specifying larger subsets of nodes.

#### Examples.

1. Upgrade golang version.

ansible/roles/builder/tasks/main.yml:
```
...
- name: Uninstall golang
  win_chocolatey:
    name: golang
    state: absent
  tags: bump
 
 - name: Install golang
   win_chocolatey:
     name: golang
     version: 1.10.0
     state: present
  tags: bump
...
```

ansible/mysite.yml
```
---
- name: 'bump golang'
  hosts: 
    - builder
  roles: 
    - builder
```

inventory.devel/groups:
```
...
[builder]
10.84.12.87
...
```

Command to run:
```
ansible-playbook -i inventory.devel --vault-password-file ~/.ansible-vault --tags "bump" mysite.yml
```
