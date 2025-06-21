pgadmin-docker
=========

Deploys pgAdmin using Docker

Requirements
------------

- Ansible 2.9 or higher
- Supported operating systems: Linux
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
    - pgadmin-docker
```

Tags
----

- `pgadmin-docker`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
