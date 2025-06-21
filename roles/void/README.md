void
=========

Installs Void development tools

Requirements
------------

- Ansible 2.9 or higher
- Supported operating systems: Linux

Role Variables
--------------

| Variable | Default | Description |
|----------|---------|-------------|
| void_version | `1.99.30038` | Configure void version |

Dependencies
------------

None

Example Playbook
----------------

```yaml
- hosts: servers
  become: yes
  roles:
    - void
```

With custom variables:

```yaml
- hosts: servers
  become: yes
  roles:
    - role: void
      vars:
        void_version: 1.99.30038
```

Tags
----

- `void`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
