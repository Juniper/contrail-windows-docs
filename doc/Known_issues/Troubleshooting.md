# Troubleshooting

Please follow steps below if you encounter issues when using Windows Contrail.

## 1. Use `contrail-status`

Open the PowerShell command prompt (with the Administrator priviledge) on the compute node.
Run contrail-status.ps1 from https://github.com/Juniper/contrail-windows/tree/master/scripts
Make sure that the following services listed by the script are running: Docker, contrail-cnm-plugin, contrail-vrouter-nodemgr and contrail-vrouter-agent.
Also make sure that the vrouter extension that is listed by the script is enabled and running. 
You may also run the script using the optional -Verbose option to get more details of the system as well as get the location of several important files such as logs etc. 

## 2. Diagnostic check

Run `Invoke-DiagnosticCheck.ps1` script from [tools repository](https://github.com/Juniper/contrail-windows-tools)
on Windows Computes that cause problems. Refer to README in root of the repository
for script configuration info.

**Note**: the script can be ran with or without Administrator privileges. However, some checks
will not be performed without them.

## 3. Microsoft's diagnostic check

Run Microsoft's `Debug-ContainerHost.ps1` script:

```
Invoke-WebRequest https://aka.ms/Debug-ContainerHost.ps1 -UseBasicParsing | Invoke-Expression
```

For other problems with Windows Containers, consult the [official troubleshooting docs](https://docs.microsoft.com/en-us/virtualization/windowscontainers/troubleshooting).

## 4. Check the logs

Diagnostic check already looks for issues in logs of Contrail Components running on the node.
However, you might want to check them manually.

Logs are located in `$Env:ProgramData\contrail\var\logs\contrail\`.

## 5. Verify the configuration

You might want to verify the configuration of all Contrail services running on the node.

Config files are located in `$Env:ProgramData\contrail\etc\contrail\`

## 6. Known issues

Search [known issues section](./Known_issues.md) for symptoms of your issue.
The section contains recovery procedures.

## 7. Use Contrail utility tools

**Note**: the following steps require experience using Contrail utils.

Behaviour of Windows Contrail utility tools is similar to their Linux counterparts. They are: `vif`, `nh`, `rt`,
`flow`, `mpls`, `vrfstats`, `dropstats`, `vxlan`, `vrouter`, `vrmemstats`.

They should be in `$Path`. If not, they can be found under `$Env:ProgramFiles\Juniper Networks\vRouter utilities\`.

## 8. Crashdumps

If you encounter any kernel panics or service crashes, debugging dump files may prove useful:

* Kernel crashdumps: `$Env:SystemRoot\MEMORY.DMP` and `$Env:SystemRoot\Minidump\*dmp`
* Usermode crashdumps: `$Env:LocalAppData\CrashDumps`

## 9. PDB files

Debugging symbols for Contrail Components can be found in PDB files in Artifacts folder, next to MSI files.
Artifacts folder is unpacked from docker containers on test beds and can be found directly on C: drive.
For debugging you can use WinDbg or DbgShell (if you know PowerShell).

**Warning**: Remember, that every PDB file is bound to executable which was built with it
(verified by checksum).

## 10. Total cleanup

Run `Clear-ComputeNode.ps1` script from [tools repository](https://github.com/Juniper/contrail-windows-tools)
on Windows Computes that cause problems. Refer to README in root of the repository
for script configuration info.

**Warning**: The script tries to clear all traces of Windows Contrail along with
any leftover state. This includes any workload instances (containers).

**Note**: For ease of use, you can use `Invoke-ScriptInRemoteSessions.ps1` script that will run
the cleanup on a remote machine:

```
\Invoke-ScriptInRemoteSessions.ps1 -ScriptFileName ".\Clear-ComputeNode.ps1" -Addresses "<IP1>,<IP2>" -Credential (Get-Credential)
```

Redeployment of Windows Compute node is required after runnig this script. Please
refer to [deployment instructions](../Quick_start/deployment.md).
