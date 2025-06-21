gomigrate
=========

Installs gomigrate database migration tool for Go

Requirements
------------

- Ansible 2.9 or higher
- Supported operating systems: Linux

Role Variables
--------------

| Variable | Default | Description |
|----------|---------|-------------|
| gomigrate_version | `4.18.1` | Configure gomigrate version |

Dependencies
------------

None

Example Playbook
----------------

```yaml
- hosts: servers
  become: yes
  roles:
    - gomigrate
```

With custom variables:

```yaml
- hosts: servers
  become: yes
  roles:
    - role: gomigrate
      vars:
        gomigrate_version: 4.18.1
```

Tags
----

- `gomigrate`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
