tmux
=========

Installs tmux terminal multiplexer

Requirements
------------

- Ansible 2.9 or higher
- Supported operating systems: CentOS, Debian, Rocky Linux, Ubuntu

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
    - tmux
```

Tags
----

- `tmux`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
