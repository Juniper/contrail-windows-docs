# CredSSP errors bestiary

In this document the Windows machine from where you are trying the RDP is called "client" and the
Windows 2016 machine you are trying an RDP into is called a "server".

## I get authentication error "CredSSP encryption Oracle remediation" when trying to RDP into a Windows machine from another Windows machine

![CredSSP encryption Oracle remediation](CredSSPError.png)

If you see the above error, please configure either the client or the server.

### Changing the options in the server itself:

warning: Please note that this makes the server vulnerable.

This is the safer option as it does not make the client vulnerable.
On the Server, please start Server Manager. Go to Local Server and click on Remote Desktop.
Make sure that the checkbox that says "allow connections only from computers running remote desktop with Network Level Authentication" is not checked.

![System Properties](SystemProperties.png)

### Changing the options on the client:

Warning: Please note that this makes the client vulnerable.

On the client machine,

1.run gpedit.msc

2.Go to the "Encryption oracle remediation" option as shown in the image below.

![Local Group Policy Editor](LocalGroupPolicyEditor_EOR.png)

3.Set the "enabled" option and make the protection level "Vulnerable" as shown below.

![Encryption Oracle Remediation](EncryptionOracleRemediationDlg.png)

### References and further reading

Microsoft KB Article:

https://support.microsoft.com/en-us/help/4093492/credssp-updates-for-cve-2018-0886-march-13-2018.
Please note that the two options mentioned on this page are workarounds.

CVE-2018-0886 | CredSSP Remote Code Execution Vulnerability:

https://portal.msrc.microsoft.com/en-us/security-guidance/advisory/CVE-2018-0886