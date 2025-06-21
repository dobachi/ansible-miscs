clion
=========

Installs JetBrains CLion IDE for C/C++ development

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
| clion_version | `2025.1.1` | Configure clion version |
| clion_directory | `2025.1.1` | Configure clion directory |

Dependencies
------------

None

Example Playbook
----------------

```yaml
- hosts: servers
  become: yes
  roles:
    - clion
```

With custom variables:

```yaml
- hosts: servers
  become: yes
  roles:
    - role: clion
      vars:
        clion_version: 2025.1.1
        clion_directory: 2025.1.1
```

Tags
----

- `clion`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
