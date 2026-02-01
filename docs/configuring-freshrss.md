<!--
SPDX-FileCopyrightText: 2020 - 2024 MDAD project contributors
SPDX-FileCopyrightText: 2020 - 2024 Slavi Pantaleev
SPDX-FileCopyrightText: 2020 Aaron Raimist
SPDX-FileCopyrightText: 2020 Chris van Dijk
SPDX-FileCopyrightText: 2020 Dominik Zajac
SPDX-FileCopyrightText: 2020 Mickaël Cornière
SPDX-FileCopyrightText: 2022 François Darveau
SPDX-FileCopyrightText: 2022 Julian Foad
SPDX-FileCopyrightText: 2022 Warren Bailey
SPDX-FileCopyrightText: 2023 Antonis Christofides
SPDX-FileCopyrightText: 2023 Alejandro AR
SPDX-FileCopyrightText: 2023 Felix Stupp
SPDX-FileCopyrightText: 2023 Julian-Samuel Gebühr
SPDX-FileCopyrightText: 2023 Pierre 'McFly' Marty
SPDX-FileCopyrightText: 2024 Thomas Miceli
SPDX-FileCopyrightText: 2024 - 2025 Suguru Hirahara

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Setting up FreshRSS

This is an [Ansible](https://www.ansible.com/) role which installs [FreshRSS](https://freshrss.org/) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

FreshRSS is a self-hosted RSS and Atom feed aggregator, which is lightweight, easy to work with, powerful, and customizable.

See the project's [documentation](https://freshrss.github.io/FreshRSS/) to learn what FreshRSS does and why it might be useful to you.

## Prerequisites

To run a FreshRSS instance it is necessary to prepare a database. You can use a [MySQL](https://www.mysql.com/) compatible database server, [Postgres](https://www.postgresql.org/), or [SQLite](https://www.sqlite.org/). The SQLite database file will be automatically created by the service if it is enabled.

If you are looking for Ansible roles for a MySQL compatible server or Postgres, you can check out [ansible-role-mariadb](https://github.com/mother-of-all-self-hosting/ansible-role-mariadb) and [ansible-role-postgres](https://github.com/mother-of-all-self-hosting/ansible-role-postgres), both of which are maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team.

## Adjusting the playbook configuration

To enable FreshRSS with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# freshrss                                                             #
#                                                                      #
########################################################################

freshrss_enabled: true

########################################################################
#                                                                      #
# /freshrss                                                            #
#                                                                      #
########################################################################
```

### Set the hostname

To enable FreshRSS you need to set the hostname as well. To do so, add the following configuration to your `vars.yml` file. Make sure to replace `example.com` with your own value.

```yaml
freshrss_hostname: "example.com"
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your server.

**Note**: hosting FreshRSS under a subpath (by configuring the `freshrss_path_prefix` variable) does not seem to be possible due to FreshRSS's technical limitations. See [this issue](https://github.com/mother-of-all-self-hosting/mash-playbook/issues/116) for details.

### Specify database

It is necessary to select database used by FreshRSS from a MySQL compatible database, Postgres, and SQLite.

To use Postgres, add the following configuration to your `vars.yml` file:

```yaml
freshrss_database_type: postgres
```

Set `mysql` to use a MySQL compatible database, and `sqlite` to use SQLite. The SQLite database is stored in the directory specified with `freshrss_data_path`.

For other settings, check variables such as `freshrss_database_*` on [`defaults/main.yml`](../defaults/main.yml).

### Extending the configuration

There are some additional things you may wish to configure about the service.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `freshrss_environment_variables_additional_variables` variable

See [the official documentation](https://github.com/FreshRSS/FreshRSS/tree/edge/Docker#environment-variables) for a complete list of FreshRSS's config options that you could put in `freshrss_environment_variables_additional_variables`.

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, FreshRSS becomes available at the specified hostname like `https://example.com`.

To get started, open the URL with a web browser, and follow the set up wizard.

### Outputting database credentials

On the set up wizard, it is required to input database credentials to use a MySQL compatible database or Postgres. You can output its credentials by running the playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=print-freshrss-db-credentials
```

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu freshrss` (or how you/your playbook named the service, e.g. `mash-freshrss`).
