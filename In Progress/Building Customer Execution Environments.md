# When to make a custom EE?
There are a few considerations when determining if you should make a custom EE vs the requirements.yaml process outlined [here](/General/Installing%20content.md) and the official docs can be found [here](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.3/html-single/creating_and_consuming_execution_environments/index). Consider the following and make the right choice for your situation, there is no objective answer. 

- Is the additional project sync time proving disruptive?
- Do I have environment specific requirements such a krb5.conf or additional trusted CAs?
- Are there additional system packages needed outside of Ansible collections?

# How to create a custom EE on the command line?
This is done via the ansible-builder utility and outlined in the blog [here](https://www.redhat.com/en/blog/introduction-to-ansible-builder). This is referencing an older build but the process is still accurate. In a nutshell ansible-builder is a wrapper for podman that takes a detailed list of requirements and generates a containerfile to build the new EE from. It is recommended to use the supported "ee-minimal-*" image for your version of AAP. You can find this in product at (Automation Execution -> Infrastructure -> Execution Environments) you can ignore the hash following the URI for this purpose. These are typically available in rhel8 or rhel9 options. Unless you have a reason to be on 8 choose 9.
![image](/Images/custom-ee-1.png)

# How to create a custom EE via playbook within AAP?
Red Hat's community of practice maintains a role for building custom EE's within a playbook, [infra.ee_utilities](https://galaxy.ansible.com/ui/repo/published/infra/ee_utilities/content/role/ee_builder/). This role is at v4.0.2 at the time of writting.

## Prerequisites
- RHEL server to run the playbook against.
- A RH account to be used to pull images from registry.redhat.io
- A private automation hub to push our image too that also contains all the collections we want to install in our EE.
- A git repository to store these files in. With the below at /collections/requirements.yml
```
---
collections:
  - name: infra.ee_utilities
  - name: ansible.controller
  - name: containers.podman
```

## Sample playbook
First lets take a look at what the ee.yml vars file looks like that contains our EE definition. The example includes installing additional RPMs, a krb5.conf file, python packages, and ansible content.

```Ansible
---
# Base Image
ee_base_image: registry.redhat.io/ansible-automation-platform-25/ee-minimal-rhel9:latest
ee_aap_version: 2.5
ee_update_base_images: true

# download from access.redhat.com -> Downloads -> OpenShift Container Platform -> Packages
ocp_rpm_name: "openshift-clients-4.18.0-202504151332.p0.geb9bc9b.assembly.stream.el9.x86_64.rpm"

ee_list:
  - name: my-custom-ee
    # automatically tags the image with current date
    tag: "{{ now(fmt='%m-%d-%Y') }}"
    build_items:
      - "krb5.conf"
    build_files:
      - src: "krb5.conf"
        dest: rpms
    build_steps:
      prepend_base:
        - RUN $PYCMD -m pip install --upgrade pip setuptools
        - COPY _build/rpms/krb5.conf /etc/krb5.conf
        # dependencies for redhat.openshift_virtualization
        - COPY _build/rpms/openshift-clients*.rpm /tmp/openshift-clients.rpm
        - RUN $PKGMGR -y update && $PKGMGR -y install bash-completion && $PKGMGR clean all
        - RUN rpm -ivh /tmp/openshift-clients.rpm && rm /tmp/openshift-clients.rpm
    dependencies:
      python_interpreter:
        python_path: "/usr/bin/python3.11"
      python:
        # Zabbix
        - netaddr>=0.8.0
        - Jinja2>=3.1.2
        # Windows
        - pykerberos==1.2.4
        # Linux
        - firewall
      system:
        # Required for microsoft.ad, needs to match version of python_path
        - python3.11-devel [platform:rpm]
      galaxy:
        collections:
          # General
          - name: community.general
          # AAP
          - name: ansible.controller
          - name: ansible.platform
          # OCP
          - name: redhat.openshift_virtualization
          - name: redhat.openshift
          - name: kubernetes.core
          # Windows
          - name: microsoft.ad
          - name: chocolatey.chocolatey
          - name: ansible.windows
          - name: community.windows
          # zabbix
          - name: zabbix.zabbix
          - name: ansible.utils
          - name: ansible.posix
```

This playbook will install the necessary packages on the RHEL server and import the above vars file. The role will build the custom EE and publish it to our private automation hub.
```Ansible
---
- name: Build EE
  hosts: rhel-server
  environment:
    PATH: /sbin:/bin:/usr/sbin:/usr/bin:/usr/local/bin

  vars_files:
    - ee.yml

  pre_tasks:
    - name: Install Dependencies
      ansible.builtin.dnf:
        name:
          - podman
          - python3-pip

    - name: Install ansible-builder with pip
      ansible.builtin.pip:
        name:
          - ansible
          - ansible-builder

  roles:
    - infra.ee_utilities.ee_builder
```

infra.ee_utilities requires a lot of information that shouldn't be hard coded so here is an example of a custom credential type you can use to provide those values. Reference the role documentation above for what values to provide.

Input Configuration
```
fields:
  - id: base_registry_username
    type: string
    label: EE Base Registry Username
  - id: base_registry_password
    type: string
    label: EE Base Registry Password
    secret: true
  - id: registry_dest
    type: string
    label: Destination registry
  - id: registry_username
    type: string
    label: Destination registry Username
  - id: registry_password
    type: string
    label: Destination registry Password
    secret: true
  - id: ah_host
    type: string
    label: EE Automation Hub
  - id: ah_token
    type: string
    label: EE Automation Hub Token
    secret: true
required:
  - ee_base_registry_username
  - ee_base_registry_password
  - ee_registry_dest
  - ee_registry_username
  - ee_registry_password
  - ee_ah_host
  - ee_ah_token
```

Injector Configuration
```
extra_vars:
  ee_ah_host: '{{ ah_host }}'
  ee_ah_token: '{{ ah_token }}'
  ee_registry_dest: '{{ registry_dest }}'
  ee_registry_password: '{{ registry_password }}'
  ee_registry_username: '{{ registry_username }}'
  ee_base_registry_password: '{{ base_registry_password }}'
  ee_base_registry_username: '{{ base_registry_username }}'
```

# How to use a custom EE in AAP?
After publishing your custom EE to your private automation hub click the dots next your EE and select "Use in Controller"
![image](/Images/custom-ee-2.png)

After this has been added you can now select it anywhere an execution envrionemnt is applicable. Projects, inventory sources, job templates, etc.

> Note that when you update your EE you will need to update the tag in (Automation Execution -> Infrastructure -> Execution Environments -> my-custom-ee). Simply using the "lastest" tag will not work. It is reccomended to be explicit with your tags.