docker
=========

Installs Docker CE and Docker Compose

Requirements
------------

- Ansible 2.9 or higher
- Supported operating systems: CentOS, Debian, Rocky Linux, Ubuntu
- Requires root/sudo privileges

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
    - docker
```

Tags
----

- `docker`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
