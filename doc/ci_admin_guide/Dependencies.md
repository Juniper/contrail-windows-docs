## How to check what is installed on builders

## Prerequisites:
1. Checkout `github.com/Juniper/contrail-windows-ci` repository.
1. Enter `ansible/` directory.
1. Prepare your environment by going through steps described in `README.md`. 
1. Log into Jenkins
1. Inspect every builder node and note its IP address
1. Create inventory file for [builder] group, that contains their IPs:

```
[builder]
10.84.12.A
10.84.12.B
10.84.12.C
...
```

### Check version of psget package:

Run the following command:
```
ansible builder -i inventory --vault-password-file ~/.ansible-vault -m win_shell -a "Get-Package -Name PSScriptAnalyzer | Select -Expand Version"
```

### Check versions of chocolatey packages:

Run the following command:
```
ansible builder -i inventory --vault-password-file ~/.ansible-vault -m win_shell -a "choco list --local-only"
```
