jan
=========

Installs Jan AI assistant application

Requirements
------------

- Ansible 2.9 or higher
- Supported operating systems: Linux

Role Variables
--------------

| Variable | Default | Description |
|----------|---------|-------------|
| jan_version | `0.5.17` | Configure jan version |

Dependencies
------------

None

Example Playbook
----------------

```yaml
- hosts: servers
  become: yes
  roles:
    - jan
```

With custom variables:

```yaml
- hosts: servers
  become: yes
  roles:
    - role: jan
      vars:
        jan_version: 0.5.17
```

Tags
----

- `jan`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
