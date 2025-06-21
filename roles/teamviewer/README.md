teamviewer
=========

Installs TeamViewer remote desktop software

Requirements
------------

- Ansible 2.9 or higher
- Supported operating systems: Debian, Ubuntu

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
    - teamviewer
```

Tags
----

- `teamviewer`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
