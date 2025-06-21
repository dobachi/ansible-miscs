intellij
=========

Installs JetBrains IntelliJ IDEA Ultimate IDE

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
| intellij_version | `2024.1.4` | Configure intellij version |
| intellij_directory | `241.18034.62` | Configure intellij directory |

Dependencies
------------

None

Example Playbook
----------------

```yaml
- hosts: servers
  become: yes
  roles:
    - intellij
```

With custom variables:

```yaml
- hosts: servers
  become: yes
  roles:
    - role: intellij
      vars:
        intellij_version: 2024.1.4
        intellij_directory: 241.18034.62
```

Tags
----

- `intellij`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
