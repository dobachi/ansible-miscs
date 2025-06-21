goland
=========

Installs JetBrains GoLand IDE for Go development

Requirements
------------

- Ansible 2.9 or higher
- Supported operating systems: Linux
- Requires X11/GUI environment
- Sufficient disk space for IDE installation

Role Variables
--------------

| Variable | Default | Description |
|----------|---------|-------------|
| goland_version | `2024.2.3` | Configure goland version |

Dependencies
------------

None

Example Playbook
----------------

```yaml
- hosts: servers
  become: yes
  roles:
    - goland
```

With custom variables:

```yaml
- hosts: servers
  become: yes
  roles:
    - role: goland
      vars:
        goland_version: 2024.2.3
```

Tags
----

- `goland`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
