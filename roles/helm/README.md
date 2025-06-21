helm
=========

Installs Helm package manager for Kubernetes

Requirements
------------

- Ansible 2.9 or higher
- Supported operating systems: Linux

Role Variables
--------------

| Variable | Default | Description |
|----------|---------|-------------|
| helm_helmfile_version | `1.0.0-rc.7` | Configure helm helmfile version |

Dependencies
------------

None

Example Playbook
----------------

```yaml
- hosts: servers
  become: yes
  roles:
    - helm
```

With custom variables:

```yaml
- hosts: servers
  become: yes
  roles:
    - role: helm
      vars:
        helm_helmfile_version: 1.0.0-rc.7
```

Tags
----

- `helm`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
