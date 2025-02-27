# Using PowerShell DSC resources with Ansible

## What is DSC?
PowerShell DSC (Desired State Configuration) is a PowerShell native configuration management framework. Ansible has support to directly leverage this communities body of work via playbooks. There is a fair degree of overlap between the official Microsoft resources and Ansible.Windows but there are some notable community modules available that are very useful.

1) The [DSC Community](https://github.com/dsccommunity) maintains several modules for: SQLServer, IIS, Sharepoint, ActiveDirectory, Security/Audit policies, etc.

2) Microsoft also maintains [modules for O365](https://github.com/Microsoft/Microsoft365DSC)

## Prerequisites
1) Windows PowerShell 5.1 is installed on the node. (Default in Server 2016+)
2) Recommended but not required that there is no other source of DSC configuration (Azure Automation or pull server)
3) [The node is set up to install modules](/Windows/Installing%20Modules.md)
3) Ensure the resource you intend to use is not a "composite" as these are not supported. This can be verified by running the following on a Windows machine and checking the "ImplementedAs" value returned.
```powershell
$ModuleName = "Module Name"
$ResourceName = "Resource Name"
Install-Module -Name $ModuleName
Import-Module -Name $ModuleName
Get-DscResource -Name $ModuleName -Module $ModuleName
```
![image](/Images/dsc-1.png)

## Tasks
win_dsc is a bit unique in that the parameters are dynamic based on the specified resource_name since these are different for every resource.

```ansible
    - name: Setup the SecurityPolicyDSC module
      community.windows.win_psmodule:
        name: SecurityPolicyDSC
        module_version: 2.10.0.0
        state: present
        accept_license: true

    - name: Set minimum password length
      ansible.windows.win_dsc:
        resource_name: AccountPolicy
        Name: Minimum_Password_Length
        Maximum_Password_Age: 8
```
