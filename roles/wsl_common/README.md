wsl_common
=========

Common configurations for WSL environments

Requirements
------------

- Ansible 2.9 or higher
- Supported operating systems: Linux
- Windows Subsystem for Linux (WSL) environment

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
    - wsl_common
```

Tags
----

- `wsl_common`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
