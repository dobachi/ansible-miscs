vim
=========

Installs Vim text editor

Requirements
------------

- Ansible 2.9 or higher
- Supported operating systems: Linux

Role Variables
--------------

| Variable | Default | Description |
|----------|---------|-------------|
| vim_clean | None | Configure vim clean |
| vim_force | None | Configure vim force |

Dependencies
------------

None

Example Playbook
----------------

```yaml
- hosts: servers
  become: yes
  roles:
    - vim
```

With custom variables:

```yaml
- hosts: servers
  become: yes
  roles:
    - role: vim
      vars:
        vim_clean: False
        vim_force: False
```

Tags
----

- `vim`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
