# When to make a custom EE?
There are a few considerations when determining if you should make a custom EE vs the requirements.yaml process outlined [here](/General/Installing%20content.md). Consider the following and make the right choice for your situation, there is no objective answer.

- Is the additional project sync time proving disruptive?
- Do I have environment specific requirements such a krb5.conf or additional trusted CAs?
- Are there additional system packages needed outside of Ansible collections?

# How to create a custom EE on the command line?
This is done via the ansible-builder utility and outlined in the blog [here](https://www.redhat.com/en/blog/introduction-to-ansible-builder). This is referencing an older build but the process is still accurate. In a nutshell ansible-builder is a wrapper for podman that takes a detailed list of requirements and generates a containerfile to build the new EE from. It is recommended to use the supported "ee-minimal-*" image for your version of AAP. You can find this in product at (Automation Execution -> Infrastructure -> Execution Environments) you can ignore the hash following the URI for this purpose. These are typically available in rhel8 or rhel9 options. Unless you have a reason to be on 8 choose 9.
![image](/Images/custom-ee-1.png)

# How to create a custom EE via playbook within AAP?
