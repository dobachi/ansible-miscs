pip3
=========

Installs pip3 Python package manager

Requirements
------------

- Ansible 2.9 or higher
- Supported operating systems: CentOS, Rocky Linux, Ubuntu

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
    - pip3
```

Tags
----

- `pip3`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
