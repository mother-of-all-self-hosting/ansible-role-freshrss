<!--
SPDX-FileCopyrightText: 2018-2026 Slavi Pantaleev
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

## What the scenarios verify

FreshRSS is configured through a web installation wizard, and this role
deliberately leaves that step to the administrator: it prints the database
credentials to type in (see [`tasks/print_db_credentials.yml`](../tasks/print_db_credentials.yml))
rather than performing the installation.

That makes FreshRSS an awkward thing to test honestly. An instance that has
never been installed still answers on every endpoint a test would reach for:
`/` redirects, `/i/` returns `200` (serving the installation wizard rather than
the login form), and `/api/greader.php` returns a bare `OK`. A test that checks
the service is active and that HTTP responds therefore passes against an
instance nobody could actually use.

So the scenarios install FreshRSS first, non-interactively, by setting the
`FRESHRSS_INSTALL` and `FRESHRSS_USER` environment variables that the container
image's entrypoint reads and turns into `cli/do-install.php` and
`cli/create-user.php` invocations. Both are no-ops once they have run, so
converging repeatedly stays safe. The database host is spelled exactly the way
this role tells administrators to spell it, so these tests fail if that advice
is ever wrong.

Against the resulting instance, each scenario asserts that:

- the systemd service is active and FreshRSS answers over HTTP
- FreshRSS is installed - `/i/` serves the login form rather than the
  installation wizard
- the version FreshRSS reports matches `freshrss_version` from the role's defaults
- the Google Reader API rejects a wrong password, and issues a token for the
  right one
- the Google Reader API refuses to serve articles without authentication
- articles fetched from a feed come back out through the authenticated API,
  and are counted in the database

The feed is served by an nginx sidecar container attached to the FreshRSS
container network, so no scenario depends on an internet feed being reachable
or on its contents staying put. `side_effect.yml` starts the sidecar,
subscribes FreshRSS to it via `cli/import-for-user.php`, and fetches it via
`cli/actualize-user.php`; `verify.yml` then asserts the articles arrived.

## Scenarios

Currently these testing scenarios are available:

### `default`

Tests a FreshRSS installation backed by SQLite, and asserts that the SQLite
database was created.

### `mariadb`

Tests a FreshRSS installation backed by MariaDB, connected over a Unix socket.

### `postgres`

Tests a FreshRSS installation backed by Postgres, connected over a Unix socket.

A scenario named after a database is worth nothing unless it proves that
database is the one in use - FreshRSS falls back to SQLite readily. So the
`mariadb` and `postgres` scenarios look for the articles in that database
itself, querying it through the respective role's own `cli-non-interactive`
helper, and require the SQLite database a fallback would have produced to be
absent.

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
