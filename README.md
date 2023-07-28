# Add-VM-DRS
To create an Ansible playbook that adds a virtual machine (VM) to an existing DRS (Distributed Resource Scheduler) VM rule in VMware vSphere, you can use the "vmware_vm_vm_drs_vm_rule" module from the "community.vmware.vmware_vm_vm_drs_rule_info" collection. This module allows you to manage DRS VM rules for VMware virtual machines.

Before using the playbook, make sure you have installed the "community.vmware.vmware_vm_vm_drs_rule_info" collection. You can install it using the following command:
```bash
ansible-galaxy collection install community.vmware
```

Once the collection is installed, create the following Ansible playbook:

**add_vm_to_drs_rule.yml:**
```yaml
---
- name: Add VM to DRS VM Rule
  hosts: localhost
  gather_facts: no

  tasks:
    - name: Get DRS VM Rule Info
      community.vmware.vmware_vm_vm_drs_rule_info:
        hostname: "{{ vcenter_hostname }}"
        username: "{{ vcenter_username }}"
        password: "{{ vcenter_password }}"
        cluster_name: "{{ cluster_name }}"
        name: "{{ drs_vm_rule_name }}"
      register: drs_rule_info

    - name: Add VM to DRS VM Rule
      community.vmware.vmware_vm_vm_drs_vm_rule:
        hostname: "{{ vcenter_hostname }}"
        username: "{{ vcenter_username }}"
        password: "{{ vcenter_password }}"
        cluster_name: "{{ cluster_name }}"
        name: "{{ drs_vm_rule_name }}"
        enabled: "{{ drs_rule_info.drs_rule.enabled | default(True) }}"
        affinity_rule: "{{ drs_rule_info.drs_rule.affinity_rule | default(False) }}"
        vm_list: "{{ drs_rule_info.drs_rule.vm_list | default([]) + [vm_name] }}"
      when: drs_vm_rule_name in drs_rule_info.drs_rule.name
```

Replace the following variables with your environment-specific values:

- `vcenter_hostname`: The hostname or IP address of your VMware vCenter server.
- `vcenter_username`: The username of your vCenter server with sufficient privileges.
- `vcenter_password`: The password for the vCenter user.
- `cluster_name`: The name of the cluster where the DRS VM rule exists.
- `drs_vm_rule_name`: The name of the existing DRS VM rule to which you want to add the VM.
- `vm_name`: The name of the VM that you want to add to the DRS VM rule.

Save the playbook and run it using the `ansible-playbook` command:
```bash
ansible-playbook add_vm_to_drs_rule.yml
```

The playbook will add the specified VM to the existing DRS VM rule in the specified cluster. If the VM is already present in the rule or if the rule does not exist, the playbook will have no effect.

Please ensure you have a backup or take necessary precautions before running any playbook that modifies critical infrastructure settings.
