spark
=========

Installs Apache Spark distributed computing framework

Requirements
------------

- Ansible 2.9 or higher
- Supported operating systems: Linux

Role Variables
--------------

| Variable | Default | Description |
|----------|---------|-------------|
| spark_version | `3.1.1` | Configure spark version |
| spark_hadoop_version | `3.2` | Configure spark hadoop version |

Dependencies
------------

None

Example Playbook
----------------

```yaml
- hosts: servers
  become: yes
  roles:
    - spark
```

With custom variables:

```yaml
- hosts: servers
  become: yes
  roles:
    - role: spark
      vars:
        spark_version: 3.1.1
        spark_hadoop_version: 3.2
```

Tags
----

- `spark`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
