# Patching with Windows Update
Most patching use cases our outlined in the ansible.windows.win_updates docs [here](https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_updates_module.html).

## Additional Considerations
1) Note that by default updates will be pulled from Microsoft's CDN. The server_selection parameter may be used to specify an on prem WSUS server but cannot target a SCCM server.

2) The ability to specify a given patch to install by KB# can easily be parameterized behind a survey for targeted remediation. The below example will take input from a "textarea" answer type with one KB# per line.
```ansible
- name: Install only particular updates based on the KB numbers
  ansible.windows.win_updates:
    category_names:
      - SecurityUpdates
    accept_list: "{{ test_data.split('\n') }}"
```
