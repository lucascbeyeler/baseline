Baseline
=========

Baseline playbook to update and install all the packages needed for a server

[![Build Status](https://travis-ci.org/lucascbeyeler/baseline.svg?branch=master)](https://travis-ci.org/lucascbeyeler/baseline)
![Linux Distro](https://img.shields.io/badge/platform-CentOS%20%7C%20Red%20Hat%20%7C%20Ubuntu-blue.svg)
![Branch](https://img.shields.io/badge/Branch-Master-green.svg)
[![Ansible Version](https://img.shields.io/badge/Ansible-2.3.1.0-green.svg)](https://www.ansible.com/)


Requirements
------------

* [Ansible](https://github.com/ansible/ansible) 2.3.1.0 or superior.


Install
--------------
Baseline is already in Ansible Galaxy, so the only thing you need is to install this script in your machine is just use ansible-galaxy command:

```
ansible-galaxy install lucascbeyeler.baseline
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
* Including hushlogin to hide the MOTD;
* Change the SSH default port;
* Disable Root login throught SSH.


Role Variables
--------------

* **hostname:** set the hostname of your server **WITHOUT** the domain;
* **domain:** set the domain for the server and the primary domain for your server;
* **timezone:** inform the timezone the playbook should set in your server;
* **enable_hushlogin:** enable hush login for all users inside your server;
* **ssh_port:** define the default port for OpenSSH Server;

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

[![GNU GPL v3.0](http://www.gnu.org/graphics/gplv3-127x51.png)](http://www.gnu.org/licenses/gpl.html)

View official GNU site <http://www.gnu.org/licenses/gpl.html>.

Author Information
------------------

* [Lucas Costa Beyeler](https://github.com/lucascbeyeler) - lucas.costab@outlook.com
