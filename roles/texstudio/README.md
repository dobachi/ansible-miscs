texstudio
=========

LaTeX 用 GUI エディタ TeXstudio を apt でインストールします。
TeX Live 本体は別途 `texlive` ロールで導入してください。

Requirements
------------

- Ansible 2.9 以上
- Debian / Ubuntu 系 (apt)

Role Variables
--------------

This role has no configurable variables.

Dependencies
------------

None. (実用には `texlive` ロールが先に必要)

Example Playbook
----------------

```yaml
- hosts: servers
  become: yes
  roles:
    - texlive
    - texstudio
```

Tags
----

- `texstudio`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
