mockery
=========

Installs mockery mock generation tool for Go

Requirements
------------

- Ansible 2.9 or higher
- Supported operating systems: Linux

Role Variables
--------------

| Variable | Default | Description |
|----------|---------|-------------|
| mockery_version | `2.27.1` | Configure mockery version |

Dependencies
------------

None

Example Playbook
----------------

```yaml
- hosts: servers
  become: yes
  roles:
    - mockery
```

With custom variables:

```yaml
- hosts: servers
  become: yes
  roles:
    - role: mockery
      vars:
        mockery_version: 2.27.1
```

Tags
----

- `mockery`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
