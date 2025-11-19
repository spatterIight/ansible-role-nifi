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
SPDX-FileCopyrightText: 2025 spatterlight

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Setting up Apache NiFi

This is an [Ansible](https://www.ansible.com/) role which installs [Apache NiFi](https://nifi.apache.org/) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

Apache NiFi is a Q&A community platform software for teams.

See the project's [documentation](https://nifi.apache.org/docs/) to learn what Apache NiFi does and why it might be useful to you.

## Prerequisites

To run a Apache NiFi instance it is necessary to prepare a database. You can use a [MySQL](https://www.mysql.com/) compatible database server, [Postgres](https://www.postgresql.org/), or [SQLite](https://www.sqlite.org/). The SQLite database file will be automatically created by the service if it is enabled.

If you are looking for Ansible roles for a MySQL compatible server or Postgres, you can check out [ansible-role-mariadb](https://github.com/mother-of-all-self-hosting/ansible-role-mariadb) and [ansible-role-postgres](https://github.com/mother-of-all-self-hosting/ansible-role-postgres), both of which are maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team.

## Adjusting the playbook configuration

To enable Apache NiFi with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# nifi                                                               #
#                                                                      #
########################################################################

nifi_enabled: true

########################################################################
#                                                                      #
# /nifi                                                              #
#                                                                      #
########################################################################
```

### Set the hostname

To enable Apache NiFi you need to set the hostname as well. To do so, add the following configuration to your `vars.yml` file. Make sure to replace `example.com` with your own value.

```yaml
nifi_hostname: "example.com"
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your server.

**Note**: hosting Apache NiFi under a subpath (by configuring the `nifi_path_prefix` variable) does not seem to be possible due to Apache NiFi's technical limitations.

### Automatic installation with environment variables

By default the role is configured to install the service with environment variables automatically when running the installation command.

To disable automatic installation, add the following configuration to your `vars.yml` file:

```yaml
nifi_environment_variables_auto_install: false
```

#### Specify database

For automatic installation, it is necessary to select database used by Apache NiFi from a MySQL compatible database, Postgres, and SQLite.

To use Postgres, add the following configuration to your `vars.yml` file:

```yaml
nifi_database_type: postgres
```

Set `mysql` to use a MySQL compatible database, and `sqlite` to use SQLite. The SQLite database is stored in the directory specified with `nifi_data_path`.

For other settings, check variables such as `nifi_database_*` on [`defaults/main.yml`](../defaults/main.yml).

#### Specify details for the administrator

You also need to set the name, email address, and password for the administrator by adding the following configuration to your `vars.yml` file:

```yaml
nifi_environment_variables_admin_name: ADMIN_NAME_HERE
nifi_environment_variables_admin_email: ADMIN_EMAIL_ADDRESS_HERE
nifi_environment_variables_admin_password: ADMIN_PASSWORD_HERE
```

#### Set email address for contact

It is necessary to specify the email address for the contact who is responsible for the instance as well. Add the following configuration to your `vars.yml` file to set it:

```yaml
nifi_environment_variables_contact_email: CONTACT_EMAIL_ADDRESS_HERE
```

### Extending the configuration

There are some additional things you may wish to configure about the component.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `nifi_environment_variables_additional_variables` variable

See [the official documentation](https://nifi.apache.org/docs/env/) for a complete list of Apache NiFi's config options that you could put in `nifi_environment_variables_additional_variables`.

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, Apache NiFi becomes available at the specified hostname like `https://example.com`.

To get started, open the URL with a web browser to log in to the instance. If automatic installation is disabled, please follow the set up wizard.

After logging in as the administrator, you can configure settings of the instance (account registrations , SMTP settings, etc.) at `https://example.com/admin/dashboard`.

## Troubleshooting

User guide is available on [this page](https://nifi.apache.org/docs/nifi-docs/html/user-guide.html).

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu nifi` (or how you/your playbook named the service, e.g. `mash-nifi`).

### wadawd

● Based on my analysis of the scripts in the current directory, here are all the environment variables supported by this NiFi Docker
  container:

  JVM Configuration

  - NIFI_JVM_HEAP_INIT - Initial JVM heap size
  - NIFI_JVM_HEAP_MAX - Maximum JVM heap size
  - NIFI_JVM_DEBUGGER - Enable JVM debugger (any value enables it)

  Web Server & Network

  - NIFI_WEB_HTTPS_PORT - HTTPS port (default: 8443)
  - NIFI_WEB_HTTPS_HOST - HTTPS host (default: hostname)
  - NIFI_WEB_PROXY_HOST - Proxy host for accessing NiFi UI
  - NIFI_WEB_PROXY_CONTEXT_PATH - Web proxy context path
  - NIFI_REMOTE_INPUT_HOST - Remote input host (default: hostname)
  - NIFI_REMOTE_INPUT_SOCKET_PORT - Remote input socket port (default: 10000)

  Clustering

  - NIFI_CLUSTER_IS_NODE - Whether instance is a cluster node (default: false)
  - NIFI_CLUSTER_ADDRESS - Cluster node address (default: hostname)
  - NIFI_CLUSTER_NODE_PROTOCOL_PORT - Cluster protocol port
  - NIFI_CLUSTER_NODE_PROTOCOL_MAX_THREADS - Max protocol threads (default: 50)
  - NIFI_CLUSTER_LOAD_BALANCE_HOST - Load balance host
  - NIFI_CLUSTER_LEADER_ELECTION_IMPLEMENTATION - Leader election implementation (default: CuratorLeaderElectionManager)
  - NIFI_CLUSTER_LEADER_ELECTION_KUBERNETES_LEASE_PREFIX - Kubernetes lease prefix

  ZooKeeper

  - NIFI_ZK_CONNECT_STRING - ZooKeeper connection string
  - NIFI_ZK_ROOT_NODE - ZooKeeper root node (default: /nifi)
  - NIFI_ELECTION_MAX_WAIT - Election max wait time (default: 5 mins)
  - NIFI_ELECTION_MAX_CANDIDATES - Election max candidates

  State Management

  - NIFI_STATE_MANAGEMENT_PROVIDER_CLUSTER - State management provider (default: zk-provider)
  - NIFI_KUBERNETES_CONFIGMAP_NAME_PREFIX - Kubernetes ConfigMap name prefix

  Analytics

  - NIFI_ANALYTICS_PREDICT_ENABLED - Enable analytics prediction (default: false)
  - NIFI_ANALYTICS_PREDICT_INTERVAL - Prediction interval (default: 3 mins)
  - NIFI_ANALYTICS_QUERY_INTERVAL - Query interval (default: 5 mins)
  - NIFI_ANALYTICS_MODEL_IMPLEMENTATION - Model implementation class
  - NIFI_ANALYTICS_MODEL_SCORE_NAME - Model score name (default: rSquared)
  - NIFI_ANALYTICS_MODEL_SCORE_THRESHOLD - Model score threshold (default: .90)

  NAR Libraries

  - NIFI_NAR_LIBRARY_PROVIDER_NIFI_REGISTRY_URL - NiFi Registry URL for NAR provider

  Security & Authentication

  - NIFI_SENSITIVE_PROPS_KEY - Sensitive properties encryption key
  - AUTH - Authentication mode: tls, ldap, or oidc

  Single User Authentication

  - SINGLE_USER_CREDENTIALS_USERNAME - Single user username
  - SINGLE_USER_CREDENTIALS_PASSWORD - Single user password

  TLS/SSL (when AUTH=tls)

  - KEYSTORE_PATH - Required - Absolute path to keystore
  - KEYSTORE_TYPE - Required - Keystore type (JKS, PKCS12, PEM)
  - KEYSTORE_PASSWORD - Required - Keystore password
  - KEY_PASSWORD - Key password (defaults to KEYSTORE_PASSWORD)
  - TRUSTSTORE_PATH - Required - Absolute path to truststore
  - TRUSTSTORE_TYPE - Required - Truststore type (JKS, PKCS12, PEM)
  - TRUSTSTORE_PASSWORD - Required - Truststore password
  - NIFI_SECURITY_USER_AUTHORIZER - User authorizer (default: managed-authorizer)
  - NIFI_SECURITY_USER_LOGIN_IDENTITY_PROVIDER - Login identity provider
  - INITIAL_ADMIN_IDENTITY - Initial admin user identity
  - INITIAL_ADMIN_GROUP - Initial admin group
  - NODE_IDENTITY - Node identity for cluster communication

  LDAP (when AUTH=ldap)

  - LDAP_AUTHENTICATION_STRATEGY - LDAP authentication strategy
  - LDAP_MANAGER_DN - LDAP manager DN
  - LDAP_MANAGER_PASSWORD - LDAP manager password
  - LDAP_TLS_KEYSTORE - LDAP TLS keystore path
  - LDAP_TLS_KEYSTORE_PASSWORD - LDAP TLS keystore password
  - LDAP_TLS_KEYSTORE_TYPE - LDAP TLS keystore type
  - LDAP_TLS_TRUSTSTORE - LDAP TLS truststore path
  - LDAP_TLS_TRUSTSTORE_PASSWORD - LDAP TLS truststore password
  - LDAP_TLS_TRUSTSTORE_TYPE - LDAP TLS truststore type
  - LDAP_TLS_PROTOCOL - LDAP TLS protocol
  - LDAP_URL - LDAP server URL
  - LDAP_USER_SEARCH_BASE - LDAP user search base
  - LDAP_USER_SEARCH_FILTER - LDAP user search filter
  - LDAP_IDENTITY_STRATEGY - LDAP identity strategy
  - LDAP_REFERRAL_STRATEGY - LDAP referral strategy

  OIDC (when AUTH=oidc)

  - NIFI_SECURITY_USER_OIDC_DISCOVERY_URL - OIDC discovery URL
  - NIFI_SECURITY_USER_OIDC_CONNECT_TIMEOUT - OIDC connect timeout
  - NIFI_SECURITY_USER_OIDC_READ_TIMEOUT - OIDC read timeout
  - NIFI_SECURITY_USER_OIDC_CLIENT_ID - OIDC client ID
  - NIFI_SECURITY_USER_OIDC_CLIENT_SECRET - OIDC client secret
  - NIFI_SECURITY_USER_OIDC_PREFERRED_JWSALGORITHM - Preferred JWS algorithm
  - NIFI_SECURITY_USER_OIDC_ADDITIONAL_SCOPES - Additional OIDC scopes
  - NIFI_SECURITY_USER_OIDC_CLAIM_IDENTIFYING_USER - Claim identifying user
  - NIFI_SECURITY_USER_OIDC_CLAIM_GROUPS - Claim for groups
  - NIFI_SECURITY_USER_OIDC_FALLBACK_CLAIMS_IDENTIFYING_USER - Fallback claims
  - NIFI_SECURITY_USER_OIDC_TRUSTSTORE_STRATEGY - Truststore strategy
  - NIFI_SECURITY_USER_OIDC_TOKEN_REFRESH_WINDOW - Token refresh window

  The main entry point is start.sh:105-124, which branches based on the AUTH variable to configure the appropriate authentication
  method.
