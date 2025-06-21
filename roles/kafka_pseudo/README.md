kafka_pseudo
=========

Installs Apache Kafka in pseudo-distributed mode

Requirements
------------

- Ansible 2.9 or higher
- Supported operating systems: Linux

Role Variables
--------------

| Variable | Default | Description |
|----------|---------|-------------|
| kafka_pseudo_baseurl | `http://ftp.riken.jp/net/apache/kafka` | Configure kafka pseudo baseurl |
| kafka_pseudo_version | `2.6.0` | Configure kafka pseudo version |
| kafka_pseudo_scala_version | `2.12` | Configure kafka pseudo scala version |

Dependencies
------------

None

Example Playbook
----------------

```yaml
- hosts: servers
  become: yes
  roles:
    - kafka_pseudo
```

With custom variables:

```yaml
- hosts: servers
  become: yes
  roles:
    - role: kafka_pseudo
      vars:
        kafka_pseudo_baseurl: http://ftp.riken.jp/net/apache/kafka
        kafka_pseudo_version: 2.6.0
```

Tags
----

- `kafka_pseudo`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
