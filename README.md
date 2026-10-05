Baseline
=========

Baseline playbook to update and install all the packages needed for a server

[![Build Status](https://circleci.com/gh/lucascbeyeler/baseline.svg?style=shield)](https://circleci.com/gh/lucascbeyeler/baseline)
![CentOS / Red Hat](https://img.shields.io/badge/platform-CentOS%20%7C%20Red%20Hat-green.svg)
![Ubuntu](https://img.shields.io/badge/platform-Ubuntu-green.svg)
[![Ansible Version](https://img.shields.io/badge/Ansible-2.18.0-green.svg)](https://www.ansible.com/)


Requirements
------------

* [Ansible](https://github.com/ansible/ansible) 2.18.0 or superior.
* This playbook now depends on the `community.general` collection. You can install it using:
  ```
  ansible-galaxy collection install -r requirements.yml
  ```
  More information can be found [here](https://galaxy.ansible.com/ui/repo/published/community/general/?extIdCarryOver=true&sc_cid=701f2000001OH7YAAW).


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
ansible-galaxy install -f lucascbeyeler.baseline
```

Features
--------------

* Update the system and install some basic packages (like vim, unzip, and ca-certificates), plus EPEL on RedHat;
* Configure chrony as the NTP client and change the timezone to what you want;
* Change the hostname and update the /etc/hosts to include 127.0.1.1 to answer when the hostname is resolved;
* Enable the NTP service to start during the boot (SystemD);
* Including hushlogin in /etc/skel (new users) and root's home to hide the MOTD;
* Change the SSH default port (also allowed in SELinux when it is enabled);
* Disable Root login throught SSH.

Every change to `/etc/ssh/sshd_config` is validated with `sshd -t` before it is written, and sshd is reloaded afterwards.

> **Warning:** when changing `ssh_port`, make sure the new port is open in your firewall (firewalld, ufw, security groups) and update `ansible_port` for the next runs, otherwise you will lose SSH access.


Tested Platforms
----------------
This playbook has been tested against:
* Rocky Linux 9
* Ubuntu Jammy Jellyfish


Role Variables
--------------

* **hostname:** set the hostname of your server **WITHOUT** the domain;
* **domain:** set the domain for the server and the primary domain for your server;
* **timezone:** inform the timezone the playbook should set in your server;
* **enable_hushlogin:** set to `true` to enable hush login (default `false`);
* **ssh_port:** define the default port for OpenSSH Server (unset keeps the current port);
* **baseline_ssh_permit_root_login:** value for `PermitRootLogin` (default `no`);
* **baseline_full_upgrade:** upgrade all the packages on every run (default `true`);
* **baseline_install_epel:** install EPEL on RedHat systems (default `true`);
* **baseline_ntp_servers:** list of NTP servers used by chrony (default `pool.ntp.org`);
* **multidist:** packages installed on every distribution;
* **debian / centos:** extra packages installed only on Debian/Ubuntu or CentOS/RedHat;

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

```
- hosts: all
  become: yes
  become_method: sudo
  roles:
     - role: lucascbeyeler.baseline
       hostname: pikachu
       domain: johto.com
       timezone: America/Sao_Paulo
       enable_hushlogin: true
       ssh_port: 8080
```

License
-------

[![GNU GPL v3.0](http://www.gnu.org/graphics/gplv3-127x51.png)](http://www.gnu.org/licenses/gpl.html)

View official GNU site <http://www.gnu.org/licenses/gpl.html>.

Author Information
------------------

* [Lucas Costa Beyeler](https://github.com/lucascbeyeler) - sysdevbey@gmail.com
