# List of all important VMs in Windows CI

1. Unknown:
    * `ci-vc` - What is this?
    * `ci-vc-um` - What is this?

2. Base OS templates:
    * `Windows` - Install OS (manually) + instruction (manually)
    * `CentOS` - No instruction? What need to be done? Install OS and configure accounts? Which one?

3. CI templates:
    * `builder` - Ansible, Jenkins job
    * `tester` - Ansible, Jenkins job
    * `testbed` - Ansible

4. Infra:
    * `builders` - Ansible, Jenkins job
    * `testers` - Ansible, Jenkins job
    * `mgmt-dhcp` - Created manually, DHCP server + configuration (172.17.0.0/24)
        * impact if down:
            * will cause failure in all production CI jobs due to lack of dhcp for testbed machines
            * will not affect demo-env, dev-env and other machines in public network
        * fix cost:
            * TODO
    * `winci-drive` - Created manually, network drive with (probably) temporary content
        * impact if down: 
            * no impact on production CI
            * degrades debuggability of CI (cannot upload or use ready artifacts)
            * impacts ability to deploy builders
        * fix cost:
            * will need to recreate directory structure and copy contents from one of the builders
            * this process is not documented to this degree of detail
    * `winci-graphite` - Created manually
        * impact if down:
            * no impact on production CI
            * degradres CI monitoring
        * fix cost:
            * configuration is not stored anywhere, so it will have to be manually redeployed and reconfigured
    * `winci-monitoring` - Created manually
        * impact if down:
            * TODO: does this affect production job (namely - post stage will fail, right?)
        * fix cost:
            * TODO
    * `winci-jenkins` - Created manually, plugins + configuration
    * `winci-mgmt` - Created manually, Python requirements.txt have to be installed
    * `winci-vyos-mgmt` - Created manually, what is inside?
    * `winci-zuulv2-production` - Created manually with Ansible script
    * `winci-dummy-1` - What is this?
    * `winci-purgatory` - Probably created manually? What is installed there?

5. Hosts:
    TODO
