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
SPDX-FileCopyrightText: 2023 Felix Stupp
SPDX-FileCopyrightText: 2023 Julian-Samuel Gebühr
SPDX-FileCopyrightText: 2023 Pierre 'McFly' Marty
SPDX-FileCopyrightText: 2024 Thomas Miceli
SPDX-FileCopyrightText: 2024 - 2025 Suguru Hirahara

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Setting up Gotify

This is an [Ansible](https://www.ansible.com/) role which installs [Gotify](https://gotify.net/) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

Gotify is a simple server for sending and receiving messages.

See the project's [documentation](https://gotify.net/docs/) to learn what Gotify does and why it might be useful to you.

## Prerequisites

To run a Gotify instance it is necessary to prepare a database. You can use a [MySQL](https://www.mysql.com/) compatible database server, [Postgres](https://www.postgresql.org/), or [SQLite](https://www.sqlite.org/). The SQLite database file will be automatically created by the service if it is enabled.

If you are looking for Ansible roles for a MySQL compatible server or Postgres, you can check out [ansible-role-mariadb](https://github.com/mother-of-all-self-hosting/ansible-role-mariadb) and [ansible-role-postgres](https://github.com/mother-of-all-self-hosting/ansible-role-postgres), both of which are maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team.

## Adjusting the playbook configuration

To enable Gotify with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# gotify                                                               #
#                                                                      #
########################################################################

gotify_enabled: true

########################################################################
#                                                                      #
# /gotify                                                              #
#                                                                      #
########################################################################
```

### Set the hostname

To enable Gotify you need to set the hostname as well. To do so, add the following configuration to your `vars.yml` file. Make sure to replace `example.com` with your own value.

```yaml
gotify_hostname: "example.com"
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your server.

**Note**: hosting Gotify under a subpath (by configuring the `gotify_path_prefix` variable) does not seem to be possible due to Gotify's technical limitations.

### Specify database

It is necessary to select database used by Gotify from a MySQL compatible database, Postgres, and SQLite.

To use Postgres, add the following configuration to your `vars.yml` file:

```yaml
gotify_database_type: postgres
```

Set `mysql` to use a MySQL compatible database, and `sqlite` to use SQLite. The SQLite database is stored in the directory specified with `gotify_data_path`.

For other settings, check variables such as `gotify_database_*` on [`defaults/main.yml`](../defaults/main.yml).

### Specify username and password for the first user

You also need to set the initial username and password for the first user by adding the following configuration to your `vars.yml` file:

```yaml
gotify_environment_variables_defaultuser_name: YOUR_USERNAME_HERE
gotify_environment_variables_defaultuser_pass: YOUR_PASSWORD_HERE
```

>[!NOTE]
> These values will only be used on the first run. To edit the user thereafter, you'll have to use the web UI.

### Extending the configuration

There are some additional things you may wish to configure about the component.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `gotify_environment_variables_additional_variables` variable

See [the official documentation](https://gotify.net/docs/config) for a complete list of Gotify's config options that you could put in `gotify_environment_variables_additional_variables`.

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, Gotify becomes available at the specified hostname like `https://example.com`.

To get started, open the URL with a web browser to log in to the instance.

### Enabling user registration

By default the role is configured to disable user registration. To enable it, add the following configuration to your `vars.yml` file:

```yaml
gotify_environment_variables_registration: true
```

### Integrating services

Gotify is not quite useful unless it is integrated with other services. You can find a list of integrations on [this repository](https://github.com/gotify/contrib).

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu gotify` (or how you/your playbook named the service, e.g. `mash-gotify`).
