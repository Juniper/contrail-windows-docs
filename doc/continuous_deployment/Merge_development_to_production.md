# Merge development to production

This document describes the steps required to perform a merge from `development` to `production` in Contrail Windows CI.


## Pre-merge check

Run production pipeline on `development` branch

1. Preferably freeze `$commitId` of `development` branch
2. Clone `winci-server2016-prod` to `winci-prod-test`
3. Change `winci-prod-test` configuration
    - Properties content: `BRANCH_NAME=$commitId`
    - Branches to build: `$commitId`
4. Run `winci-prod-test`
    - Parameters:
        - `ZUUL_PROJECT=Juniper/contrail-controller`
        - `ZUUL_BRANCH=master`
        - `ZUUL_REF=None`
        - `ZUUL_URL=http://10.84.12.75/merge-warrior`
        - `ZUUL_UUID=$someRandomUUID`
        - `ZUUL_CHANGE=` (can be left empty)
        - `ZUUL_PATCHSET=` (can be left empty)


## Merge development to production

1. Update production branch

    ```bash
    git fetch --all --prune
    git checkout development
    git pull
    git checkout production
    git merge development
    git push
    ```

2. Update Zuul configuration

    ```bash
    localhost $ ssh winci-mgmt
    winci-mgmt $ cd ~/ji/juniper-contrail-windows-ci
    winci-mgmt $ git checkout production
    winci-mgmt $ git pull
    winci-mgmt $ git log
    # apply patch to ansible/roles/zuul/defaults/main.yml with production url/key/user
    # - gerrit_server
    # - gerrit_user
    # - gerrit_keynam
    winci-mgmt $ cd ansible
    winci-mgmt $ ansible-playbook -i ~/ji/ansible.hosts play.yml --vault-password-file ~/.ansible-vault
    ```


### Zuul update playbook

```yaml
---
- hosts: zuul
  remote_user: ciadmin
  become: yes
  roles:
  - zuul
```


## Post checks

1. Check Zuul services
    - `zuul-merger`
    - `zuul-server`
2. Trigger Zuul job
    - Create a dummy PR on Gerrit
    - Post `recheck windows` on some existing PR


## Future considerations

- Slave VM templates creation and promotion
- Builder/tester deployment
