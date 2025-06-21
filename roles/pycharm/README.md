pycharm
=========

Installs JetBrains PyCharm Professional IDE

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
| pycharm_version | `2024.2.3` | Configure pycharm version |
| pycharm_directory | `2024.2.3` | Configure pycharm directory |

Dependencies
------------

None

Example Playbook
----------------

```yaml
- hosts: servers
  become: yes
  roles:
    - pycharm
```

With custom variables:

```yaml
- hosts: servers
  become: yes
  roles:
    - role: pycharm
      vars:
        pycharm_version: 2024.2.3
        pycharm_directory: 2024.2.3
```

Tags
----

- `pycharm`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
