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

# Setting up Apache Answer

This is an [Ansible](https://www.ansible.com/) role which installs [Apache Answer](https://answer.apache.org/) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

Apache Answer is a Q&A community platform software for teams.

See the project's [documentation](https://answer.apache.org/docs/) to learn what Apache Answer does and why it might be useful to you.

## Prerequisites

To run a Apache Answer instance it is necessary to prepare a database. You can use a [MySQL](https://www.mysql.com/) compatible database server, [Postgres](https://www.postgresql.org/), or [SQLite](https://www.sqlite.org/). The SQLite database file will be automatically created by the service if it is enabled.

If you are looking for Ansible roles for a MySQL compatible server or Postgres, you can check out [ansible-role-mariadb](https://github.com/mother-of-all-self-hosting/ansible-role-mariadb) and [ansible-role-postgres](https://github.com/mother-of-all-self-hosting/ansible-role-postgres), both of which are maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team.

## Adjusting the playbook configuration

To enable Apache Answer with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# answer                                                               #
#                                                                      #
########################################################################

answer_enabled: true

########################################################################
#                                                                      #
# /answer                                                              #
#                                                                      #
########################################################################
```

### Set the hostname

To enable Apache Answer you need to set the hostname as well. To do so, add the following configuration to your `vars.yml` file. Make sure to replace `example.com` with your own value.

```yaml
answer_hostname: "example.com"
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your server.

**Note**: hosting Apache Answer under a subpath (by configuring the `answer_path_prefix` variable) does not seem to be possible due to Apache Answer's technical limitations.

### Automatic installation with environment variables

By default the role is configured to install the service with environment variables automatically when running the installation command.

To disable automatic installation, add the following configuration to your `vars.yml` file:

```yaml
answer_environment_variables_auto_install: false
```

#### Specify database

For automatic installation, it is necessary to select database used by Apache Answer from a MySQL compatible database, Postgres, and SQLite.

To use Postgres, add the following configuration to your `vars.yml` file:

```yaml
answer_database_type: postgres
```

Set `mysql` to use a MySQL compatible database, and `sqlite` to use SQLite. The SQLite database is stored in the directory specified with `answer_data_path`.

For other settings, check variables such as `answer_database_*` on [`defaults/main.yml`](../defaults/main.yml).

#### Specify details for the administrator

You also need to set the name, email address, and password for the administrator by adding the following configuration to your `vars.yml` file:

```yaml
answer_environment_variables_admin_name: ADMIN_NAME_HERE
answer_environment_variables_admin_email: ADMIN_EMAIL_ADDRESS_HERE
answer_environment_variables_admin_password: ADMIN_PASSWORD_HERE
```

#### Set email address for contact

It is necessary to specify the email address for the contact who is responsible for the instance as well. Add the following configuration to your `vars.yml` file to set it:

```yaml
answer_environment_variables_contact_email: CONTACT_EMAIL_ADDRESS_HERE
```

### Extending the configuration

There are some additional things you may wish to configure about the component.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `answer_environment_variables_additional_variables` variable

See [the official documentation](https://answer.apache.org/docs/env/) for a complete list of Apache Answer's config options that you could put in `answer_environment_variables_additional_variables`.

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, Apache Answer becomes available at the specified hostname like `https://example.com`.

To get started, open the URL with a web browser to log in to the instance. If automatic installation is disabled, please follow the set up wizard.

After logging in as the administrator, you can configure settings of the instance (account registrations , SMTP settings, etc.) at `https://example.com/admin/dashboard`.

## Troubleshooting

FAQ is available on [this page](https://answer.apache.org/docs/faq).

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu answer` (or how you/your playbook named the service, e.g. `mash-answer`).
