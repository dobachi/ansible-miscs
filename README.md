# README

This is an example of Ansible playbooks to configure servers and WSL of Windows.

## Homeserver

* playbooks/conf/fileserver/fileserver.yml
  * Configure the file server.

## WSL

* playbooks/conf/wsl/wsl.yml
  * Configure WSL
* playbooks/conf/wsl/git_clone_sources.yml
  * Git clone sources like Spark, Kafka, and so on
* playbooks/conf/wsl/docker.yml
  * Install docker
  * You should run wsl as an Administrator
