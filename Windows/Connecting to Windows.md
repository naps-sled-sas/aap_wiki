# Connecting to Windows hosts
This guide will be written assuming WinRM and Active Directory for authentication. Both SSH and Local Auth are possible but not widely adopted or recommended at this time.

## Networking
Ansible uses the same mechanisms as RHEL for discovering an AD realm. The [precheck steps](https://access.redhat.com/solutions/5444941) around DNS configuration and open ports on your domain controllers still apply.

## Ansible Vars
Ansible assumes SSH by default and needs to be informed a host requires WinRM and with what configuration. There are many flavors of WinRM access and they are outlined [here](https://docs.ansible.com/ansible/latest/os_guide/windows_winrm.html). It is recommended to set these variables at a level above host vars, such as inventory or group vars. Example below, adjust settings according to your environment per the docs previously linked.
![image](/Images/win_connection-1.png)

## Credentials
Windows will still use the standard "machine" credential type. You will need to ensure the user account has appropriate elevated permissions to carry out your tasks. Below is an example of a Windows AD credential, note the all uppercase domain name (required) and the privilege escalation method.
![image](/Images/win_connection-2.png)
