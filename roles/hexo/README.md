hexo
=========

Installs Hexo static site generator

Requirements
------------

- Ansible 2.9 or higher
- Supported operating systems: CentOS, Ubuntu

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
    - hexo
```

Tags
----

- `hexo`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
