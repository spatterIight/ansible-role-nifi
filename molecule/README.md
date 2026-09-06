<!--
SPDX-FileCopyrightText: 2018-2025 Slavi Pantaleev
SPDX-FileCopyrightText: 2019-2022 Aaron Raimist
SPDX-FileCopyrightText: 2019-2023 MDAD project contributors
SPDX-FileCopyrightText: 2023 QEDeD
SPDX-FileCopyrightText: 2024 Fabio Bonelli
SPDX-FileCopyrightText: 2024 Nikita Chernyi
SPDX-FileCopyrightText: 2024-2026 Suguru Hirahara
SPDX-FileCopyrightText: 2026 spatterlight

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Molecule Testing

This role supports [Molecule](https://docs.ansible.com/projects/molecule/), an Ansible testing framework designed for developing and testing Ansible collections, playbooks, and roles.

## Prerequisites

To utilize Molecule you need to prepare several requirements:

- **x86** computer running one of these operating systems that make use of [systemd](https://systemd.io/):
  - **Archlinux**
  - **CentOS**, **Rocky Linux**, **AlmaLinux**, or possibly other RHEL alternatives (although your mileage may vary)
  - **Debian** (10/Buster or newer)
  - **Ubuntu** (18.04 or newer, although [20.04 may be problematic](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/ansible.md#supported-ansible-versions) if you run the Ansible playbook on it)
- `root` access on the computer which Molecule runs against
- [Ansible](http://ansible.com/) program
- [Python](https://www.python.org/)
  - Most distributions install Python by default, but some don't (e.g. Ubuntu 18.04) and require manual installation (something like `apt-get install python3`)
- [Docker](https://www.docker.com)
  - Access to Docker UNIX socket (`/var/run/docker.sock`) is required by default

## Installation

To set up the environment for using Molecule, run the command below on the terminal:

```bash
python3 -m venv ./molecule/venv
source ./molecule/venv/bin/activate
pip3 install -r ./molecule/requirements.txt
```

## Scenarios

Currently there is one testing scenario available.

### `default`

Tests a standard Apache NiFi installation.

The verification does not stop at "the systemd service is active" — the unit is `Restart=always`, so a crash-looping container reports `active` too. It:

- waits for Apache NiFi's own web interface rather than for the unit, which for a JVM application of this size takes minutes rather than seconds
- establishes that the API refuses unauthenticated requests and that a wrong password is rejected, so that the two checks below are able to fail in the first place
- logs in with the credentials the role bcrypt-hashes into `conf/login-identity-providers.xml`. A stock Apache NiFi generates a random UUID username and a random password instead and answers this with `400`, so a container running on anything but the role's configuration cannot pass this
- asserts that the running process reports the version `nifi_version` pins, via `/nifi-api/flow/about`
- asserts that `files/conf/` still matches the pinned image, which is the by-hand reconciliation [docs/updating-nifi.md](../docs/updating-nifi.md) describes, turned into something CI enforces
- asserts that `nifi_container_additional_volumes` really reaches `docker create` as a `--mount`, and that no Traefik labels are emitted while Traefik is disabled

The scenario deliberately runs Apache NiFi on port 9443 rather than the 8443 its own image defaults to. The container only publishes the port the role was told about, so a NiFi that ignored the role's `nifi.properties` would be listening where nothing is mapped and the web interface check would time out.

## Running

By default it is configured to run the scenarios on Ubuntu 26.04.

```bash
molecule test --scenario-name default
```

You can utilize other distributions by setting one to the `MOLECULE_DISTRO` environment variable:

```bash
# Ubuntu 24.04
MOLECULE_DISTRO=ubuntu2404 molecule test --scenario-name default

# Debian 13
MOLECULE_DISTRO=debian13 molecule test --scenario-name default

# Debian 12
MOLECULE_DISTRO=debian12 molecule test --scenario-name default
```
