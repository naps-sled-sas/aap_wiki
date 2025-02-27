# Patching with Windows Update
Most patching use cases our outlined in the ansible.windows.win_updates docs [here](https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_updates_module.html).

## Additional Considerations
1) Note that by default updates will be pulled from Microsoft's CDN. The server_selection parameter may be used to specify an on prem WSUS server but cannot target a SCCM server.

2) The ability to specify a given patch to install by KB# can easily be parameterized behind a survey for targeted remediation. To provide multiple IDs look at the examples [here](/General/List%20survey.md).
```ansible
- name: Install only particular updates based on the KB numbers
  ansible.windows.win_updates:
    category_names:
      - SecurityUpdates
    accept_list: "{{ patchid }}"
```

3) Reboot timeouts may need adjusted for physical servers. The default timeouts are typically good enough for VMs but to avoid false failures adjust the reboot timeout for physical machines. This can be accomplished by having separate playbooks or by taking advantage of ansible_facts to dynamically adjust the value.
```
- name: Set default reboot_timeout
  ansible.builtin.set_fact:
    reboot_timeout: 1200

- name: Adjust reboot_timeout for physical nodes
  ansible.builtin.set_fact:
    reboot_timeout: 2400
  when: (ansible_facts['ansible_virtualization_role'] is not defined)

- name: Install all security updates with automatic reboots
  ansible.windows.win_updates:
    category_names:
      - SecurityUpdates
    reboot: true
    reboot_timeout: "{{ reboot_timeout }}"
```
