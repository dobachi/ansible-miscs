pycharm.org
=========

Alternative PyCharm installation method

Requirements
------------

- Ansible 2.9 or higher
- Supported operating systems: Linux

Role Variables
--------------

| Variable | Default | Description |
|----------|---------|-------------|
| pycharm_version | `2020.2.3` | Configure pycharm version |

Dependencies
------------

None

Example Playbook
----------------

```yaml
- hosts: servers
  become: yes
  roles:
    - pycharm.org
```

With custom variables:

```yaml
- hosts: servers
  become: yes
  roles:
    - role: pycharm.org
      vars:
        pycharm_version: 2020.2.3
```

Tags
----

- `pycharm.org`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
