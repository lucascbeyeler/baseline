# Baseline Role Specification

`lucascbeyeler.baseline` prepares a fresh Linux server: it updates the
system, installs common packages, sets the clock, the timezone and the
hostname, and hardens OpenSSH. It is the server preparation that
[lucascbeyeler.zimbra](https://github.com/lucascbeyeler/zimbra) expects to
have run first, but it is generic and can be used before any role.

## 1. Scope

**In scope**

* Package updates and a base package set;
* EPEL on the Red Hat family;
* Timezone and NTP (chrony);
* Hostname and its `127.0.1.1` line in `/etc/hosts`;
* `.hushlogin` for root and new users;
* OpenSSH port and `PermitRootLogin`.

**Out of scope**

* Firewalls (firewalld, ufw, cloud security groups);
* Users, sudoers and SSH keys;
* Application software (that is the job of roles such as `zimbra`).

## 2. Requirements

| Item | Value |
|---|---|
| Ansible | 2.18.0 or later (`meta/main.yml` → `min_ansible_version`) |
| Collections | `community.general` (`requirements.yml`) — used by `timezone` and `seport` |
| Privileges | The play must run with `become: true` |
| Galaxy name | `lucascbeyeler.baseline` |

## 3. Supported platforms

| Family | Distributions | Tested in CI |
|---|---|---|
| Debian | Ubuntu 22.04 (Jammy) | Yes (Molecule, `ubuntu:jammy`) |
| RedHat | Red Hat / Rocky Linux / AlmaLinux 8 and 9 | Rocky Linux 9 (Molecule, `rockylinux:9`) |

These match the distributions Zimbra 10.1 runs on.

## 4. Variables

### 4.1 Public variables (`defaults/main.yml`)

| Variable | Default | Description |
|---|---|---|
| `hostname` | `localhost` | Short host name, **without** the domain |
| `domain` | `localdomain` | Domain; the FQDN is `{{ hostname }}.{{ domain }}` |
| `timezone` | `UTC` | IANA timezone, e.g. `America/Sao_Paulo` |
| `baseline_full_upgrade` | `true` | Upgrade every package on each run (`apt full-upgrade` / `dnf update *`) |
| `baseline_install_epel` | `true` | Install EPEL on the Red Hat family |
| `multidist` | `vim, wget, unzip, sysstat, sudo, ca-certificates, git` | Packages installed on every distribution |
| `debian` | `[]` | Extra packages for Debian/Ubuntu only |
| `centos` | `[]` | Extra packages for the Red Hat family only |
| `baseline_ntp_servers` | `0-3.pool.ntp.org` | Servers written to chrony's configuration |
| `enable_hushlogin` | `false` | Create `.hushlogin` in `/etc/skel` and `/root` |
| `baseline_ssh_permit_root_login` | `"no"` | Value of `PermitRootLogin` (`yes`, `no`, `prohibit-password`) |
| `ssh_port` | *undefined* | OpenSSH port. When undefined the current port is kept |

`hostname`, `domain`, `timezone`, `multidist`, `debian`, `centos`,
`enable_hushlogin` and `ssh_port` predate the `baseline_` prefix convention.
They are kept for compatibility with existing playbooks, which is why
`.ansible-lint` skips `var-naming[no-role-prefix]`. **New variables must use
the `baseline_` prefix.**

### 4.2 Internal variables (`vars/main.yml`)

Not meant to be overridden. They hold per-OS-family values in
`baseline_os_vars` and derive:

| Variable | Debian | RedHat |
|---|---|---|
| `baseline_family_packages` | `chrony` | `chrony`, `policycoreutils-python-utils` |
| `baseline_chrony_conf` | `/etc/chrony/chrony.conf` | `/etc/chrony.conf` |
| `baseline_chrony_service` | `chrony` | `chronyd` |
| `baseline_sshd_service` | `ssh` | `sshd` |

* `baseline_epel_package` — `epel-release` from the distribution's repos on
  RHEL clones; the Fedora URL
  `https://dl.fedoraproject.org/pub/epel/epel-release-latest-<major>.noarch.rpm`
  on Red Hat itself, which has no `epel-release` package.
* `baseline_manage_services` — `true` only when the service manager is
  systemd **and** the host is not a Docker/Podman container. Services and
  handlers are skipped otherwise.

## 5. Behaviour

Tasks run in this order (`tasks/main.yml`):

1. **EPEL** (RedHat, when `baseline_install_epel`).
2. **Full upgrade** (when `baseline_full_upgrade`): `apt upgrade=full` with a
   cache refresh, or `dnf name=* state=latest`.
3. **APT cache refresh** when the full upgrade is off, so a fresh image with
   an empty cache can still install packages. The cache is considered valid
   for one hour (`cache_valid_time: 3600`).
4. **Packages**: `multidist` + (`debian` or `centos`) + `baseline_family_packages`.
5. **Timezone** via `community.general.timezone`.
6. **chrony** configuration from `templates/chrony.conf.j2`
   (`root:root 0644`), notifying *Restart chrony*. The template writes one
   `server <name> iburst` line per NTP server, the family's drift file,
   `makestep 1.0 3`, `rtcsync`, `cmdport 0` and `logdir /var/log/chrony`.
7. **Hostname** set to the FQDN with `ansible.builtin.hostname`; inside
   Docker/Podman `/etc/hostname` is written instead (with `unsafe_writes`,
   because it is a bind mount).
8. **`/etc/hosts`**: a single line `127.0.1.1 <fqdn> <hostname>`, replacing
   any existing `127.0.1.1` line. Skipped in Docker, which manages the file.
9. **chrony service** started and enabled (when `baseline_manage_services`).
10. **`.hushlogin`** in `/etc/skel` and `/root` (`root:root 0600`, times
    preserved), when `enable_hushlogin` is true.
11. **SELinux**: the custom `ssh_port` is labelled `ssh_port_t` (RedHat with
    SELinux enabled, `ssh_port` defined).
12. **SSH port**: `Port <ssh_port>` in `/etc/ssh/sshd_config`, when defined.
13. **Root login**: if `sshd_config` includes `/etc/ssh/sshd_config.d/`
    (Ubuntu 22.04+, RHEL 9), `PermitRootLogin` is written to
    `/etc/ssh/sshd_config.d/00-baseline.conf` (`0600`), which sorts before
    other drop-ins such as RHEL's `01-permitrootlogin.conf` — sshd keeps the
    first value it reads. Otherwise it is set in `sshd_config` itself.

### Handlers

| Handler | Action | Condition |
|---|---|---|
| Restart chrony | restart `baseline_chrony_service` | `baseline_manage_services` |
| Reload sshd | reload `baseline_sshd_service` | `baseline_manage_services` |

## 6. Guarantees

These properties are part of the contract. A change that breaks one of them
is a breaking change.

* **Idempotent** — a second run reports no changes (checked by Molecule's
  `idempotence` step).
* **Safe SSH edits** — every change to the SSH configuration is validated
  with `sshd -t -f %s` before being written; an invalid file is never
  installed. sshd is reloaded, not restarted.
* **Secure defaults** — root login over SSH is disabled, hushlogin is off,
  and the SSH port is left untouched unless `ssh_port` is set.
* **Container-aware** — the role converges inside Docker/Podman without
  systemd.

> **Operator warning:** when changing `ssh_port`, open the new port in the
> firewall first and update `ansible_port` for the next run, or SSH access
> will be lost.

## 7. Testing

| Layer | Location | Runs | Needs |
|---|---|---|---|
| Lint | `.ansible-lint` | `ansible-lint` | Python |
| Syntax | `tests/test.yml` | `ansible-playbook -i tests/inventory tests/test.yml --syntax-check` | `ansible.cfg` with `roles_path=../` |
| Unit | `tests/unit/main.yml` | `ansible-playbook tests/unit/main.yml` | Nothing but Ansible |
| Integration | `molecule/default` | `molecule test` | Docker |

* **Unit tests** load `defaults/` and `vars/` with mocked facts and assert the
  per-family values, the EPEL source, when services are managed, the
  rendered `chrony.conf` for each family, and the secure defaults.
* **Molecule** creates `ubuntu:jammy` and `rockylinux:9` containers. `prepare.yml`
  installs Python and OpenSSH, adds a `01-permitrootlogin.conf` drop-in that
  allows root (like RHEL's installer) and empties the APT cache (like a fresh
  image). The role then runs with `ssh_port: 2222`, hushlogin on and the full
  upgrade off. `verify.yml` checks the packages, the timezone (by file
  content), chrony's servers, `/etc/hostname`, `/etc/hosts`, the `.hushlogin`
  files and their mode, and `sshd -T` reports only `port 2222` and
  `permitrootlogin no`.

See [Development.md](../Development.md) to set up the environment locally.

## 8. CI/CD

CircleCI (`.circleci/config.yml`), workflow `build-and-test`:

| Job | Runs on | Steps |
|---|---|---|
| `lint` | `cimg/python:3.12` | ansible-lint, syntax check, unit tests |
| `molecule` | `ubuntu-2404` machine | `molecule test` |
| `ansible-galaxy` | `cimg/python:3.12` | `ansible-galaxy role import` — **only on `master`**, after `lint` and `molecule` pass |

Every merge to `master` is therefore a release to Ansible Galaxy.
