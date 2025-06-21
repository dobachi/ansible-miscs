go
=========

Installs Go programming language

Requirements
------------

- Ansible 2.9 or higher
- Supported operating systems: Linux

Role Variables
--------------

| Variable | Default | Description |
|----------|---------|-------------|
| go_version | `1.19.13` | Configure go version |

Dependencies
------------

None

Example Playbook
----------------

```yaml
- hosts: servers
  become: yes
  roles:
    - go
```

With custom variables:

```yaml
- hosts: servers
  become: yes
  roles:
    - role: go
      vars:
        go_version: 1.19.13
```

Tags
----

- `go`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
