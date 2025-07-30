# Introduction
Ansible has support for Jinja2 filters or coalescing values. This is useful for setting a custom order of precendence within a given playbook or just providing default values in absence of optional parameters.

## Use cases

### Allow override for hosts
Variablizing the host field on a playbook but defaulting to an intended group lets you easily override this via a survey.

```Ansible
- name: RHEL updates
  become: true
  become_user: root
  hosts: "{{ survey_rhel_server | default('rhel_web') }}"
  tasks: 
  ...
```

### Supporting optional parameters of modules
You may want to add support for an optional parameter of a module via survey input but use the modules default in it's anscence. This lets you avoid creating multiple tasks with conditional statements to accound for different combinations.

```Ansible
- name: Update all installed packages
  ansible.builtin.dnf:
    name: '*'
    state: latest
    update_cache: "{{ survey_update_cache | default('false') }}"
```