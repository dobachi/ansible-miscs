llama
=========

Installs Llama AI model and related tools

Requirements
------------

- Ansible 2.9 or higher
- Supported operating systems: Linux

Role Variables
--------------

| Variable | Default | Description |
|----------|---------|-------------|
| llama_version | `b5604` | Configure llama version |

Dependencies
------------

None

Example Playbook
----------------

```yaml
- hosts: servers
  become: yes
  roles:
    - llama
```

With custom variables:

```yaml
- hosts: servers
  become: yes
  roles:
    - role: llama
      vars:
        llama_version: b5604
```

Tags
----

- `llama`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
