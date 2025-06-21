sbt
=========

Installs SBT (Scala Build Tool)

Requirements
------------

- Ansible 2.9 or higher
- Supported operating systems: Linux

Role Variables
--------------

| Variable | Default | Description |
|----------|---------|-------------|
| sbt_version | `1.3.13` | Configure sbt version |

Dependencies
------------

None

Example Playbook
----------------

```yaml
- hosts: servers
  become: yes
  roles:
    - sbt
```

With custom variables:

```yaml
- hosts: servers
  become: yes
  roles:
    - role: sbt
      vars:
        sbt_version: 1.3.13
```

Tags
----

- `sbt`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
