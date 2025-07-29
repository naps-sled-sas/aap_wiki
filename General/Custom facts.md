# Introduction
Ansible supports "custom facts" that are static details about a machine that will be picked up into Ansible facts every run. These differ from host vars because they live on the host not the inventory so they will be picked up by any system querying for facts such as AAP, Ansible-core, or Satellite.

Official docs can be found [here](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_vars_facts.html#facts-d-or-local-facts)
## Use cases
Some common use cases include but are not limited to house keeping or inventory data such as:
1) Application owner contact information
2) ticket # tracking creation

## How to use
### Manually
1) Create a json file called example.fact at /etc/ansible/facts.d
```json
{
	"some_data":"some value"
}
```
2) That json information will appear under ansible_facts['ansible_local']['example'] in your playbook.

### With a playbook
```ansible
- name: "Create custom fact directory"
  file:
    path: "/etc/ansible/facts.d"
    state: "directory"
- name: "Insert custom fact file"
  copy:
    src: files/custom.fact
    dest: /etc/ansible/facts.d/custom.fact
    mode: 0755
```