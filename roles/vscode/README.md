vscode
=========

Installs Visual Studio Code editor

Requirements
------------

- Ansible 2.9 or higher
- Supported operating systems: Rocky Linux, Ubuntu

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
    - vscode
```

Tags
----

- `vscode`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
