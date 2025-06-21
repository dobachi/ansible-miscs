wsl_gui
=========

Configures GUI support for WSL

Requirements
------------

- Ansible 2.9 or higher
- Supported operating systems: CentOS, Ubuntu
- Windows Subsystem for Linux (WSL) environment

Role Variables
--------------

| Variable | Default | Description |
|----------|---------|-------------|
| wsl_gui_2 | `True` | Configure wsl gui 2 |

Dependencies
------------

None

Example Playbook
----------------

```yaml
- hosts: servers
  become: yes
  roles:
    - wsl_gui
```

With custom variables:

```yaml
- hosts: servers
  become: yes
  roles:
    - role: wsl_gui
      vars:
        wsl_gui_2: True
```

Tags
----

- `wsl_gui`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
