# Troubleshooting

Please follow steps below if you encounter issues when using Windows Contrail.

## 1. Use `contrail-status`

**TODO**

`contrail-status` is not implemented yet. Please move on to next troubleshooting steps.

## 2. Diagnostic check

Run `Invoke-DiagnosticCheck.ps1` script from [tools repository](https://github.com/Juniper/contrail-windows-tools)
on Windows Computes that cause problems. Refer to README in root of the repository
for script configuration info.

**Note**: the script can be ran with or without Administrator privileges. However, some checks
will not be performed without them.

## 3. Check the logs

Diagnostic check already looks for issues in logs of Contrail Components running on the node.
However, you might want to check them manually.

Logs are located in `$Env:ProgramData\contrail\var\logs\contrail\`.

## 4. Verify the configuration

You might want to verify the configuration of all Contrail services running on the node.

Config files are located in `$Env:ProgramData\contrail\etc\contrail\`

## 5. Known issues

Search [known issues section](./Known_issues.md) for symptoms of your issue.
The section contains recovery procedures.

## 6. Use Contrail utility tools

**Note**: the following steps require experience using Contrail utils.

Behaviour of Windows Contrail utility tools is similar to their Linux counterparts. They are: `vif`, `nh`, `rt`,
`flow`, `mpls`, `vrfstats`, `dropstats`, `vxlan`, `vrouter`, `vrmemstats`.

They should be in `$Path`. If not, they can be found under `$Env:ProgramFiles\Juniper Networks\vRouter utilities\`.

## 7. Crashdumps

If you encounter any kernel panics or service crashes, debugging dump files may prove useful:

* Kernel crashdumps: `$Env:SystemRoot\MEMORY.DMP` and `$Env:SystemRoot\Minidump\*dmp`
* Usermode crashdumps: `$Env:LocalAppData\CrashDumps`

## 8. Total cleanup

Run `Clear-ComputeNode.ps1` script from [tools repository](https://github.com/Juniper/contrail-windows-tools)
on Windows Computes that cause problems. Refer to README in root of the repository
for script configuration info.

**Warning**: The script tries to clear all traces of Windows Contrail along with
any leftover state. This includes any workload instances (containers).

**Note**: For ease of use, you can use `Invoke-ScriptInRemoteSessions.ps1` script that will run
the cleanup on a remote machine.

Redeployment of Windows Compute node is required after runnig this script. Please
refer to [deployment instructions](../Quick_start/deployment.md).
