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
    * `winci-drive` - Created manually, network drive with (probably) temporary content
    * `winci-graphite` - Created manually
    * `winci-monitoring` - Created manually
    * `winci-jenkins` - Created manually, plugins + configuration
    * `winci-mgmt` - Created manually, Python requirements.txt have to be installed
    * `winci-vyos-mgmt` - Created manually, what is inside?
    * `winci-zuulv2-production` - Created manually with Ansible script
    * `winci-dummy-1` - What is this?
    * `winci-purgatory` - Probably created manually? What is installed there?
