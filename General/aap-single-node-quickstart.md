# Ansible Automation Platform 2.5 QuickStart - Single Node Containerized Installer

## Overview 
This is an unofficial guide to assist with installing the Ansible Automation Platform on a single node using the containerized installer. 

There are currently three methods to install the Ansible Automation platform components

1. **Containerized Installer (recommended)**
2. RPM Installer 
3. Operator Installer (for OpenShift)

The containerized installer will deploy the following AAP components as containers on one or more hosts. See the [Ansible Automation Platform Architecture](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.5/html/planning_your_installation/aap_architecture#aap_architecture) docs for more details.
- Platform Gateway
- Private Automation Hub
- Ansible Controller
- Event Driven Ansible Controller
- Database

Helpful Links
- [Planning your AAP Installation](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.5/html/planning_your_installation/index)
- [AAP Containerized Installer](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.5/html/containerized_installation/index)
- [Video: AAP Containerized Installer Walkthrough](https://www.youtube.com/watch?v=wUcCeyrCvyg&t=24s&ab_channel=RedHatAnsibleAutomation)

## Prerequisites

**Minimum System Requirements**
- RAM: 16GB 
- CPUs: 4
- Local Disk: 60GB
- Disk IOPS: 3000

**OS Prerequisites**
- A host VM running RHEL 9.2 or later
- A non-root user for the Red Hat Enterprise Linux host, with sudo or other Ansible supported privilege escalation (sudo recommended). This user is responsible for the installation of containerized Ansible Automation Platform.
- The appropriate network ports are open if a firewall is in place. For more information about the ports to open, see [Container topologies](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.5/html/tested_deployment_models/container-topologies#network_ports_5) in *Tested deployment models*.

## Step 1: Preparing your RHEL Host

1. Set a hostname that is a fully qualified domain name (FQDN) 
    ```shell 
    sudo hostnamectl set-hostname <your_hostname>
    ```

2. Register your RHEL Host
    ```shell
    sudo subscription-manager register
    ```

3. Ensure that BaseOS and AppStream repos are enabled on the host

    ```shell
    sudo dnf repolist
    ```
4. Ensure that the host has DNS configured and can resolve host names and IP addresses by using a fully qualified domain name (FQDN)
   
5. Install the ```ansible-core``` package
   ```shell
   sudo dnf install -y ansible-core
   ```
6. (Optional) Install additional utilities for troubleshooting purposes
   ```shell
   sudo dnf install -y wget git-core rsync vim
   ```

## Step 2: Downloading the AAP Installer

1. Download the latest **Ansible Automation Platform 2.5 Containerized Setup Bundle** .tar file from [Ansible Automation Platform download page](https://access.redhat.com/downloads/content/480/ver=2.5/rhel---9/2.5/x86_64/product-software)
   
2.  Copy the installation program .tar file onto your RHEL host. In this case we are using scp to copy the installer from a workstation to our RHEL host.
    ```shell
    scp <username>:<source_host>:~/Downloads/<aap_setup_bundle_file> <username>:<remote_host>:/target/path
    ```
3. Unpack the bundled installer

    ```shell
    tar xfvz ansible-automation-platform-containerized-setup-bundle-<version>-<arch_name>.tar.gz
    ```


## Step 3: Configuring the inventory file

1. On your RHEL host, cd into the unpacked installer directory

    ```shell
    cd ansible-automation-platform-containerized-setup-bundle-<version>-<arch_name>
    ```

2. Edit the provided **inventory-growth** file or use the following example if installing locally on a single host. 
  
    **NOTES** 
    - SSH keys are only required when installing on remote hosts. If doing a self contained local VM based installation, you can use ansible_connection=local.
    - You do have the option to leverage ansible-vault to secure sensitive information in your inventory file ([docs with examples](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.5/html/planning_your_installation/about_the_installer_inventory_file#proc-securing_secrets_in_inventory_planning))

    ```shell
    vim inventory
    ```
    
    ```ini 
    # This is the AAP installer inventory file intended for the Container growth deployment topology.
    # This inventory file expects to be run from the host where AAP will be installed.
    # Please consult the Ansible Automation Platform product documentation about this topology's tested hardware configuration.
    # https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.5/html/tested_deployment_models/container-topologies
    #
    # Please consult the docs if you're unsure what to add
    # For all optional variables please consult the included README.md
    # or the Ansible Automation Platform documentation:
    # https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.5/html/containerized_installation

    # This section is for your AAP Gateway host(s)
    # -----------------------------------------------------
    [automationgateway]
    aap.example.org

    # This section is for your AAP Controller host(s)
    # -----------------------------------------------------
    [automationcontroller]
    aap.example.org

    # This section is for your AAP Automation Hub host(s)
    # -----------------------------------------------------
    [automationhub]
    aap.example.org

    # This section is for your AAP EDA Controller host(s)
    # -----------------------------------------------------
    [automationeda]
    aap.example.org

    # This section is for the AAP database
    # -----------------------------------------------------
    [database]
    aap.example.org

    [all:vars]
    # Ansible
    ansible_connection=local

    # Common variables
    # https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.5/html/containerized_installation/appendix-inventory-files-vars#ref-general-inventory-variables
    # -----------------------------------------------------
    postgresql_admin_username=postgres
    postgresql_admin_password=<set your own>

    bundle_install=true
    # The bundle directory must include /bundle in the path
    bundle_dir='{{ lookup("ansible.builtin.env", "PWD") }}/bundle'


    redis_mode=standalone

    # AAP Gateway
    # https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.5/html/containerized_installation/appendix-inventory-files-vars#ref-gateway-variables
    # -----------------------------------------------------
    gateway_admin_password=<set your own>
    gateway_pg_host=aap.example.org
    gateway_pg_password=<set your own>

    # AAP Controller
    # https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.5/html/containerized_installation/appendix-inventory-files-vars#ref-controller-variables
    # -----------------------------------------------------
    controller_admin_password=<set your own>
    controller_pg_host=aap.example.org
    controller_pg_password=<set your own>
    controller_percent_memory_capacity=0.5

    # AAP Automation Hub
    # https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.5/html/containerized_installation/appendix-inventory-files-vars#ref-hub-variables
    # -----------------------------------------------------
    hub_admin_password=<set your own>
    hub_pg_host=aap.example.org
    hub_pg_password=<set your own>
    hub_seed_collections=false

    # AAP EDA Controller
    # https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.5/html/containerized_installation/appendix-inventory-files-vars#event-driven-ansible-controller
    # -----------------------------------------------------
    eda_admin_password=<set your own>
    eda_pg_host=aap.example.org
    eda_pg_password=<set your own>
    ```

3. Run the installer

    ```shell
    ansible-playbook -i <inventory_file_name> ansible.containerized_installer.install -K -vvv
    ```



