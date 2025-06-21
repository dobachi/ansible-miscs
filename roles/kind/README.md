kind
=========

Installs Kubernetes IN Docker (kind) for local Kubernetes clusters

Requirements
------------

- Ansible 2.9 or higher
- Supported operating systems: Linux

Role Variables
--------------

| Variable | Default | Description |
|----------|---------|-------------|
| kind_version | `v0.20.0` | Configure kind version |

Dependencies
------------

None

Example Playbook
----------------

```yaml
- hosts: servers
  become: yes
  roles:
    - kind
```

With custom variables:

```yaml
- hosts: servers
  become: yes
  roles:
    - role: kind
      vars:
        kind_version: v0.20.0
```

Tags
----

- `kind`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
