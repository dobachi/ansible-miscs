win_hostname
=========

Configures Windows hostname

Requirements
------------

- Ansible 2.9 or higher
- Supported operating systems: Linux
- Windows operating system
- WinRM configured for Ansible

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
    - win_hostname
```

Tags
----

- `win_hostname`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
