Baseline
=========

Baseline playbook to update and install all the packages needed for a server

It is the server preparation used by [lucascbeyeler.zimbra](https://github.com/lucascbeyeler/zimbra), which supports the latest version of Zimbra (10.1), on the distributions Zimbra 10.1 runs on (Ubuntu 22.04 and Rocky Linux / Red Hat 8 and 9).

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
When a new version of baseline is released, you will need to run the install process again, but with the "-f" or "--force" parameter.

```
ansible-galaxy install -f lucascbeyeler.baseline
```

Features
--------------

* Update the system and install some basic packages (like vim, unzip, and ca-certificates), plus EPEL on RedHat;
* Refresh the APT cache before installing packages, even when the full upgrade is turned off, so it works on fresh Ubuntu hosts and images with an empty cache;
* Configure chrony as the NTP client (on Ubuntu and RedHat) and change the timezone to what you want;
* Change the hostname and manage a single 127.0.1.1 line in /etc/hosts to answer when the hostname is resolved;
* Enable the NTP service to start during the boot (SystemD);
* Including hushlogin in /etc/skel (new users) and root's home to hide the MOTD (only when `enable_hushlogin` is `true`);
* Change the SSH default port (also allowed in SELinux when it is enabled);
* Disable Root login through SSH;
* Safe to run many times: nothing is changed when the server already matches the configuration.

Every change to the SSH configuration is validated with `sshd -t` before it is written, and sshd is reloaded afterwards.
When `/etc/ssh/sshd_config` includes `/etc/ssh/sshd_config.d/*.conf` (Ubuntu 22.04+, RHEL 9), `PermitRootLogin` is written to `/etc/ssh/sshd_config.d/00-baseline.conf`, so drop-ins from the installer or the cloud image (like RHEL's `01-permitrootlogin.conf`) cannot override it.

> **Warning:** when changing `ssh_port`, make sure the new port is open in your firewall (firewalld, ufw, security groups) and update `ansible_port` for the next runs, otherwise you will lose SSH access.


Tested Platforms
----------------
This playbook has been tested against:
* Rocky Linux 9
* Ubuntu Jammy Jellyfish

Both are tested with Molecule in CircleCI on every push.


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

Testing
-------

The role has unit tests (`tests/unit/main.yml`), which render the per-distribution variables and the chrony template with mocked facts, and a Molecule scenario that runs the role in Ubuntu Jammy and Rocky Linux 9 containers. CircleCI runs ansible-lint, a syntax check, the unit tests and Molecule. See [Development.md](Development.md) to run them on your machine.

License
-------

[![GNU GPL v3.0](http://www.gnu.org/graphics/gplv3-127x51.png)](http://www.gnu.org/licenses/gpl.html)

View official GNU site <http://www.gnu.org/licenses/gpl.html>.

Author Information
------------------

* [Lucas Costa Beyeler](https://github.com/lucascbeyeler) - sysdevbey@gmail.com
