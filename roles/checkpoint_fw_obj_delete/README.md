# Ansible Role: Check Point Clean Empty Groups 
An Ansible role to identify, report, and safely purge empty group objects from a Check Point Management Server when they are not referenced in any rules or objects.

## Requirements

* Minimum Ansible version: `2.16`
* Ansible Collection: check_point.mgmt >= `6.9.0`

## Role Variables

Available variables are defined in `defaults/main.yml`:

| Variable | Default | Description |
| --- | --- | --- |
| `checkpoint_group_fetch_limit` | `500` | PMaximum number of group objects to retrieve from the Management Server. |
| `checkpoint_enable_delete` | false | Dry-run safeguard toggle. Set to true to execute group deletions and publish changes. |

## Dependencies

Execution Environment with checkpoint_mgmt certifed collection.  
Here are tested EE definition files ([Firewall_EE](https://github.com/tellis4151/securityautomation/tree/main/Firewall_EE))

## Execution Workflow
Retrieve Group Objects: Uses check_point.mgmt.cp_mgmt_group_facts to fetch details for groups up to checkpoint_group_fetch_limit.

Filter Empty Groups: Parses returned facts to identify groups where the members list is defined and empty.

Query Usage Dependencies: Iterates over empty groups using check_point.mgmt.cp_mgmt_where_used to evaluate object and rulebase references.

Isolate Candidates: Filters results for empty groups where used-in.total equals 0.

Report Candidates: Outputs the list of target groups using ansible.builtin.debug.

Delete and Publish: If checkpoint_enable_delete is true, deletes the candidate groups via check_point.mgmt.cp_mgmt_group and commits the session changes using check_point.mgmt.cp_mgmt_publish.

## Ansible Automation Platform
**Credentials**:
Create a Check Point credential type directly to the Job Template. AAP handles API authentication tokens and session keys automatically at runtime.

**Host/Inventory Variables**
Define connection options on the host or inventory group level within AAP:
```yaml
---
ansible_connection: httpapi
ansible_network_os: check_point.mgmt.checkpoint
ansible_httpapi_use_ssl: true
ansible_httpapi_validate_certs: false
```

## Example Playbook

```yaml
---
- name: Audit and remove empty/unused Check Point groups
  hosts: checkpoint_mgmt
  gather_facts: false
  roles:
    - role: checkpoint_fw_obj_delete
      vars:
        checkpoint_enable_delete: false  # Change to true to enable actual deletion
        checkpoint_group_fetch_limit: 500
```


