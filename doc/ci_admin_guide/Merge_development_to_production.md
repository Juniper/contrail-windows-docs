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
        - `ZUUL_REF=None` (it should literally be `None`)
        - `ZUUL_URL=http://10.84.12.75/merge-warrior`
        - `ZUUL_UUID=$someRandomUUID` (e.g. $commitId with the first 8 characters replaced by `deadbeef`)
        - `ZUUL_CHANGE=` (can be left empty)
        - `ZUUL_PATCHSET=` (can be left empty)


## Merge development to production

1. Update production branch (following git operations require administrative privileges on GitHub)

    ```bash
    git fetch --all --prune
    git checkout development
    git pull
    git checkout production
    git merge development
    git push
    ```

2. Update Zuul configuration

    - Refer to [Update Windows CI Zuulv2][update-zuulv2] documentation


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

1. Check Zuul services:
    - `zuul-merger`
    - `zuul-server`

    ```bash
    localhost $ ssh winci-zuulv2-production # with provided credentials; current address 10.84.12.75
    systemctl status zuul-merger.service # should be active (running)
    systemctl status zuul-server.service # should be active (running)
    ```

2. Trigger Zuul job
    - Create a dummy PR on Gerrit
    - Post `recheck windows` on some existing PR


## Future considerations

- Slave VM templates creation and promotion
- Builder/tester deployment

[update-zuulv2]: https://github.com/Juniper/contrail-windows-docs/blob/master/doc/ci_admin_guide/Update_Zuul.md
