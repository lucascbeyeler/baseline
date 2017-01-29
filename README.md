ansible-commons
=========

Default playbook to prepare you environment to receive any kind of service

[![Build Status](https://travis-ci.org/lucascbeyeler/ansible-commons.svg?branch=master)](https://travis-ci.org/lucascbeyeler/ansible-commons)

Requirements
------------

* [Ansible](https://github.com/ansible/ansible) 2.2.0 or superior. Less than this and you will have problems running Zimbra's Playbook. See ansible-modules-core Bug #4202

- Configure de file /etc/ansible/ansible.cfg (create if don't exist) and set this options - not required if you using key and the ssh user is "root" already:
```
[defaults]
host_key_checking=False
stdout_callback=skippy

[ssh_connection]
pipelining=True
```

Role Variables
--------------

* **hostname:** set the hostname of your server **WITHOUT** the domain;
* **domain:** set the domain for the server and the primary domain for your server;

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

- hosts: all
  become: yes
  become_method: sudo
  roles:
     - role: lucascbeyeler.ansible-commons
       hostname: warudo
       domain: hollowbastion.com

License
-------

GNU GENERAL PUBLIC LICENSE

Author Information
------------------

https://github.com/lucascbeyeler
