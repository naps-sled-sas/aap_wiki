# Introduction
Special variables are automatically set variables about the current state of an execution. These are different than ansible_facts. ansible_facts is actually a special variable itself. Official docs can be found [here](https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html).

# Use cases

## Hostvars
You can reference details about a host without connecting to it. Any cached facts or inventory data about a host can be accessed in the variable {{ hostvars[hostname] }}. This is incredibly useful for interacting with supporting systems of a given host. For example populating the host information into another system or editing it's configuration in a hypervisor without needing to connect to the host itself and pass data along in a workflow.

```Ansible
- name: Register all disks
    ansible.builtin.set_fact:
    vm_disks: "{{ hostvars[vm_name]['vmi_volume_status'] }}"
    when: vm_name is defined
```

## Groups
You can check for additional group memberships beyond the one you're selecting for the specific execution. This is useful for further drilling down conditional tasks. For example a RHEL 9 baseline playbook targetting a "RHEL9" group may have a few additional tasks if the host is also in a "web servers" inventory group.

```Ansible
---
Hosts: "RHEL9"
...
- name: Install the latest version of Apache if also a web server
  ansible.builtin.dnf:
    name: httpd
    state: present
  when: inventory_hostname in groups["web servers"]
```