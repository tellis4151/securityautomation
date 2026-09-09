# Ansible Role: Check Point Remove Decommission Server Objects
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
Retrieves all references of where the server object is being used.

1. Identify access rules where the server object is listed in source or destination
Iterate through rules were decommissioned server object is sole source or sole destination

1. Display rules to disable
Display identified rules that will be disabled.

1. Disable Identified Rules
Disable rules where target server is sole source or sole destination
  
1. Policy Evaluation & Rule Disabling
Iterates through all returned access-rules. If target_server is the only item in source or the only item in destination, the rule's uid and layer are extracted, and cp_mgmt_access_rule sets enabled: false.

1. Identify object groups where the decommissioned server object is the ONLY member
Iterate through all server group objects to identify groups where the server object is the only member.

1. Display Groups to be deleted
Display identified groups that will be disabled.

1. Delete single-member groups
Delete groups where decommissioned server object is the ONLY member.

1. Publish changes to Management Server

1. Re-Check usage after cleanup
Run another check to see if the decommissioned server object is being used anywhere else before deletion.

1. Delete the server host object and Publish to Management Server

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


