android_studio
=========

Installs Android Studio IDE for Android development

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
| android_studio_version | `2024.3.1.14` | Configure android studio version |
| android_studio_directory | `241.18034.62` | Configure android studio directory |

Dependencies
------------

None

Example Playbook
----------------

```yaml
- hosts: servers
  become: yes
  roles:
    - android_studio
```

With custom variables:

```yaml
- hosts: servers
  become: yes
  roles:
    - role: android_studio
      vars:
        android_studio_version: 2024.3.1.14
        android_studio_directory: 241.18034.62
```

Tags
----

- `android_studio`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
