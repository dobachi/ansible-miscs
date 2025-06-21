gvim
=========

Installs GVim (GUI version of Vim)

Requirements
------------

- Ansible 2.9 or higher
- Supported operating systems: CentOS, Debian, Ubuntu

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
    - gvim
```

Tags
----

- `gvim`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
