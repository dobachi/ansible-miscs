clone_sources
=========

Clones various source code repositories

Requirements
------------

- Ansible 2.9 or higher
- Supported operating systems: Linux

Role Variables
--------------

This role has no configurable variables.

Dependencies
------------

None

Example Playbook
----------------

```yaml
- hosts: servers
  become: yes
  roles:
    - clone_sources
```

Tags
----

- `clone_sources`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
