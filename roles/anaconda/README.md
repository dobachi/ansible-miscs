anaconda
=========

Installs Anaconda Python distribution for data science

Requirements
------------

- Ansible 2.9 or higher
- Supported operating systems: Linux

Role Variables
--------------

| Variable | Default | Description |
|----------|---------|-------------|
| anaconda_home | `/opt/Anaconda` | Configure anaconda home |
| anaconda_version | `5.3.0` | Configure anaconda version |

Dependencies
------------

None

Example Playbook
----------------

```yaml
- hosts: servers
  become: yes
  roles:
    - anaconda
```

With custom variables:

```yaml
- hosts: servers
  become: yes
  roles:
    - role: anaconda
      vars:
        anaconda_home: /opt/Anaconda
        anaconda_version: 5.3.0
```

Tags
----

- `anaconda`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
