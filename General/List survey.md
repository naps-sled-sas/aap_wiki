# How to provide a list via survey
Ansible will pass survey data in as strings. Because of this there is a small additional step to pass multiple values in for a list/loop.

## Example
Using a "textarea" answer type specify unique values on separate lines. Then reference the variable using the splitlines() method.
![image](/Images/survey_loop-1.png)
```ansible
- name: Install only particular updates based on the KB numbers
  ansible.windows.win_updates:
    category_names:
      - SecurityUpdates
    accept_list: "{{ patchids.splitlines() }}"
```
```ansible
- name: Print my list
  ansible.builtin.debug:
    msg: "{{ item }}"
  with_items: "{{ patchids.splitlines() }}"
```
