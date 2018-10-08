# Troubleshooting

Please follow steps below if you encounter issues when using Windows Contrail.

## 1. Diagnostic check

Run `Invoke-DiagnosticCheck.ps1` script from [tools repository](https://github.com/Juniper/contrail-windows-tools)
on Windows Computes that cause problems. Refer to README in root of the repository
for script configuration info.

**Note**: the script can be ran with or without Administrator privileges. However, some checks
will not be performed without them.

## 2. Known issues

Search [known issues section](./Known_issues.md) for symptoms of your issue.
The section contains recovery procedures.

## 3. Total cleanup

Run `Clear-ComputeNode.ps1` script from [tools repository](https://github.com/Juniper/contrail-windows-tools)
on Windows Computes that cause problems. Refer to README in root of the repository
for script configuration info.

**Warning**: The script tries to clear all traces of Windows Contrail along with
any leftover state. This includes any workload instances (containers).

**Note**: For ease of use, you can use `Invoke-ScriptInRemoteSessions.ps1` script that will run
the cleanup on a remote machine.

Redeployment of Windows Compute node is required after runnig this script. Please
refer to [deployment instructions](../Quick_start/deployment.md).
