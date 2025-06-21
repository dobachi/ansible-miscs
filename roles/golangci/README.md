golangci
=========

Installs golangci-lint for Go code analysis

Requirements
------------

- Ansible 2.9 or higher
- Supported operating systems: Linux

Role Variables
--------------

| Variable | Default | Description |
|----------|---------|-------------|
| golangci_version | `v1.50.1` | Configure golangci version |

Dependencies
------------

None

Example Playbook
----------------

```yaml
- hosts: servers
  become: yes
  roles:
    - golangci
```

With custom variables:

```yaml
- hosts: servers
  become: yes
  roles:
    - role: golangci
      vars:
        golangci_version: v1.50.1
```

Tags
----

- `golangci`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
