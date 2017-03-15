ansible-commons
=========

Default playbook to prepare you environment to receive any kind of service

[![Build Status](https://travis-ci.org/lucascbeyeler/ansible-commons.svg?branch=master)](https://travis-ci.org/lucascbeyeler/ansible-commons)

Requirements
------------

* [Ansible](https://github.com/ansible/ansible) 2.2.0 or superior. Less than this and you will have problems running Zimbra's Playbook. See ansible-modules-core Bug #4202

* Configure de file /etc/ansible/ansible.cfg (create if don't exist) and set this options - not required if you using key and the ssh user is "root" already:
```
[defaults]
host_key_checking=False
stdout_callback=skippy

[ssh_connection]
pipelining=True
```

Install
--------------
ansible-commons is already in Ansible Galaxy, so the only thing you need to install this script in your machine is just use ansible-galaxy command:

```
ansible-galaxy install lucascbeyeler.commons
```

Update
--------------
When a new version of ansible-commons is released, you will need to run the install process again, but with the "-f" or "--force" parameter.

```
ansible-galaxy install -f lucascbeyeler.commons
```

Features
--------------

* Update the system and install some basic packages (like vim, unzip, ntp, and ca-certificates);
* Configure a ntp client and change the timezone to America/Sao_Paulo;
* Change the hostname and update the /etc/hosts to include 127.0.0.1 to answer when the hostname is resolved;
* Enable some services, like ntp, to start during the boot (Upstart and SystemD);


Role Variables
--------------

* **hostname:** set the hostname of your server **WITHOUT** the domain;
* **domain:** set the domain for the server and the primary domain for your server;
* **timezone:** inform the timezone the playbook should set in your server;

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

```
- hosts: all
  become: yes
  become_method: sudo
  roles:
     - role: lucascbeyeler.commons
       hostname: warudo
       domain: hollowbastion.com
       timezone: America/Sao_Paulo
```

License
-------

GNU GENERAL PUBLIC LICENSE

Author Information
------------------

https://github.com/lucascbeyeler
