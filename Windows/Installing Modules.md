# Installing PowerShell modules with Ansible

## Prerequisites
Windows PowerShell 5.1 does not have adequate configuration out of the box to install modules from the PSGallery the following commands either need done on your base image or via an Ansible configuration.

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
Install-PackageProvider -Name NuGet
```

```ansible
- name: Setup PsGallery
  ansible.windows.win_powershell:
    script: |
      [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
      Install-PackageProvider -Name NuGet
```

## Tasks
There will install a given module from PSGallery. Docs can be found [here](https://docs.ansible.com/ansible/latest/collections/community/windows/win_psmodule_module.html).

```ansible
- name: Setup the SecurityPolicyDSC module
      community.windows.win_psmodule:
        name: SecurityPolicyDSC
        module_version: 2.10.0.0
        state: present
        accept_license: true
```
