docker_legacy
=============

> **DEPRECATED / 互換用途**: 旧 `roles/docker` をリネームしたもの。
> Ubuntu 24.04 以降では `apt_key` が deprecated、`docker-compose` v1 も EOL の
> ため、現行 Ubuntu (24.04 / 26.04) では新しい [`roles/docker`](../docker/) を
> 使うこと。本ロールは jammy (22.04) / Debian / CentOS / Rocky の既存
> プレイブック互換のために残してある。

Installs Docker CE and Docker Compose v1 (legacy)

Requirements
------------

- Ansible 2.9 or higher
- Supported operating systems: CentOS, Debian, Rocky Linux, Ubuntu
- Requires root/sudo privileges

Role Variables
--------------

This role has no configurable variables.

Dependencies
------------

None

Example Playbook
----------------

```yaml
- hosts: servers
  become: yes
  roles:
    - docker_legacy
```

Tags
----

- `docker`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
