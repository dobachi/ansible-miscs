register_home
=========

Registers home directory settings

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
    - register_home
```

Tags
----

- `register_home`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
