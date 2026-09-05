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
| `target_server_objects` | individual or list of server objects | Server's that have been decommissioned and need to be deleted from the firewalls |

## Dependencies

Execution Environment with checkpoint_mgmt certifed collection.  
Here are tested EE definition files ([Firewall_EE](https://github.com/tellis4151/securityautomation/tree/main/Firewall_EE))

## Execution Workflow
Step-by-Step Runtime Logic

1. Normalization
tasks/main.yml evaluates target_server_objects. If passed as a single string or comma-separated string from an AAP survey, it converts the input into a standard list (['ServerA', 'ServerB']) and initiates the loop over process_server.yml.

1. Object Inspection (cp_mgmt_where_used)
Retrieves all references across object groups and access rule bases where target_server is referenced.

1. Group Evaluation & Deletion
Iterates through all returned object references of type group. If a group's members list contains exactly one item (the target server), that group is deleted before deleting the server itself.

1. Policy Evaluation & Rule Disabling
Iterates through all returned access-rules. If target_server is the only item in source or the only item in destination, the rule's uid and layer are extracted, and cp_mgmt_access_rule sets enabled: false.

1. Host Deletion & Session Publish
The host object is purged from Check Point via cp_mgmt_host. Once complete, cp_mgmt_publish commits the session changes to the Management Server database.

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


