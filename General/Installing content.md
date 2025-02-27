# Introduction
If you are not interested in creating a custom execution environment but need to add additional collections this can be done via some standard files within your project.

## Configuring Galaxy sources
1) If you only wish to use the community Ansible Galaxy you do not need to do anything, if not continue.

2) Create a credential of type "Ansible Galaxy/Automation Hub API Token". If you're adding the RedHat Automation Hub the information can be found [here](https://console.redhat.com/ansible/automation-hub/token)

3) Add the credential on a per-organization level under Access Management -> Organizations -> <org> edit and set the order of preference. When content is installed with the process below it will attempt to search at each source top to bottom. It is recommended to put the "RH Automation Hub" first to make sure you're grabbing the certified versions of your content.
![image](/Images/install_content-1.png)

## Installing content
1) Within your git repository you will need to create some files at some standard locations. No need to create the "roles" directory if roles are not required, same goes for "collections"
    - collections/requirements.yml
    - roles/requirements.yml

2) The syntax for these files can be found in the ansible docs [here](https://docs.ansible.com/ansible/latest/galaxy/user_guide.html). 

3) Every time your project syncs the content will be installed. Note that this will increase the sync time which may become problematic if you have "Update revision on launch" enabled on the project. If this is the case a custom execution environment is suggested.
