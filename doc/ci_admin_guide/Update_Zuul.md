# Update Windows CI Zuulv2

This document describes a procedure required to update Windows CI Zuulv2 configuration.
This procedure updates configuration using `development` branch from [contrail-windows-ci][c-w-ci] repository.

## Prerequisites

- Ubuntu 16.04 or Windows Subsystem for Linux with Ubuntu
    - It will serve as Ansible control maching
- User's public SSH key installed on Zuulv2 instance
    - Please contact Windows CI team
- Access to ansible vault key
    - Please contact Windows CI team

## Steps

1.  Install required `apt` dependencies on Ansible control machine

    ```bash
    apt-get install git python3 python3-pip python3-virtualenv
    ```

1.  Clone `contrail-windows-ci` repository

    ```bash
    cd ~/
    git clone https://github.com/Juniper/contrail-windows-ci.git
    cd contrail-windows-ci
    ```

1.  Verify that `development` branch contains PRs with required changes to Zuul configuration

    - Assuming `PR#2` and `PR#1` are required PRs, run the following command:

    ```bash
    git log --oneline
    ```

    - Output will contain a list of merged PRs, from newest to oldest. Required PRs should be at the top:

    ```
    abcdabc PR#2
    abcd123 PR#1
    e8691f5 Some other PR
    3af841e Some other PR, part 2
    # ... omitted
    ```

1.  Move to `ansible` directory

    ```bash
    cd ansible
    ```

1.  Create a virtualenv and install `pip` dependencies

    ```bash
    python3 -m virtualenv -p /usr/bin/python3 venv
    source venv/bin/activate
    pip install -r python-requirements.txt
    ```

1.  Create a file for Ansible vault key and populate it with the key

    ```bash
    touch ~/ansible-vault-key
    vim ~/ansible-vault-key  # enter ansible vault key into a file
    ```

1.  Test connection to Zuul with Ansible

    ```bash
    ansible -i inventory.prod --private-key=YOUR_PRIVATE_KEY zuul -m ping
    ```

    - where `YOUR_PRIVATE_KEY` is a path to your SSH private key

1.  Run `setup-zuul-server.yml` playbook

    ```bash
    ansible-playbook -i inventory.prod --private-key=YOUR_PRIVATE_KEY setup-zuul-server.yml
    ```

    Verify that the run completed successfully.
    The output should be following and `failed` task count must equal zero.

    ```
    #
    # task list omitted for brevity
    #

    PLAY RECAP ******************************************************************************************
    10.84.12.75                : ok=22   changed=2    unreachable=0    failed=0

    #
    # ... - task run time omitted for brevity
    #
    ```

    - `failed` task count should be equal to zero

[c-w-ci]: https://github.com/Juniper/contrail-windows-ci
