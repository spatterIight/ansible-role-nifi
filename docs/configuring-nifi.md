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

Apache NiFi is a easy to use, powerful, and reliable system to process and distribute data.

See the project's [documentation](https://nifi.apache.org/docs/nifi-docs/) to learn what Apache NiFi does and why it might be useful to you.

## Prerequisites

To deploy Apache NiFi using this role it is necessary that the [community.general](https://github.com/ansible-collections/community.general) be installed. This is needed to support modifying XML configuration files.

## Adjusting the playbook configuration

To enable Apache NiFi with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# nifi                                                                 #
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

### Adjusting the Traefik configuration

Since the Apache NiFi container only supports listening via HTTPS it is neccesary to configure Traefik to skip verifying Apache NiFi's HTTPS certificate (since it self-signed).

To do this a custom `serversTransports` must be defined in Traefik's **static** configuration.

```yaml
serversTransports:
  insecure-nifi-transport:
    insecureSkipVerify: true
```

Or, if you are using the [MASH Traefik](https://github.com/mother-of-all-self-hosting/mash-playbook) role.

```yaml
traefik_configuration_extension_yaml: |
  serversTransports:
    {{ nifi_container_labels_traefik_serverstransport }}:
      insecureSkipVerify: true
```

### Adjusting the Apache NiFi configuration

There are some additional things you may wish to configure about the component.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for the `nifi_conf_*` variables that you can customize via your `vars.yml` file.

To configure a default admin username and password add the following configuration to your `vars.yml` file. Please note that the password must be 12 characters minimum.

```yaml
nifi_environment_variables_single_user_credentials_username: "my-admin-username"
nifi_environment_variables_single_user_credentials_password: "my-secure-admin-password"
```

#### Supported environment variables

The complete list of Apache NiFi's config options that you could put in `nifi_environment_variables_additional_variables` are as follows:

JVM Configuration

- NIFI_JVM_HEAP_INIT
- NIFI_JVM_HEAP_MAX
- NIFI_JVM_DEBUGGER

Web Server & Network

- NIFI_WEB_HTTPS_PORT (default: 8443)
- NIFI_WEB_HTTPS_HOST (default: hostname)
- NIFI_WEB_PROXY_HOST
- NIFI_WEB_PROXY_CONTEXT_PATH
- NIFI_REMOTE_INPUT_HOST (default: hostname)
- NIFI_REMOTE_INPUT_SOCKET_PORT (default: 10000)

Clustering

- NIFI_CLUSTER_IS_NODE (default: false)
- NIFI_CLUSTER_ADDRESS (default: hostname)
- NIFI_CLUSTER_NODE_PROTOCOL_PORT
- NIFI_CLUSTER_NODE_PROTOCOL_MAX_THREADS (default: 50)
- NIFI_CLUSTER_LOAD_BALANCE_HOST
- NIFI_CLUSTER_LEADER_ELECTION_IMPLEMENTATION (default: CuratorLeaderElectionManager)
- NIFI_CLUSTER_LEADER_ELECTION_KUBERNETES_LEASE_PREFIX

ZooKeeper

- NIFI_ZK_CONNECT_STRING
- NIFI_ZK_ROOT_NODE (default: /nifi)
- NIFI_ELECTION_MAX_WAIT (default: 5 mins)
- NIFI_ELECTION_MAX_CANDIDATES

State Management

- NIFI_STATE_MANAGEMENT_PROVIDER_CLUSTER (default: zk-provider)
- NIFI_KUBERNETES_CONFIGMAP_NAME_PREFIX

Analytics

- NIFI_ANALYTICS_PREDICT_ENABLED (default: false)
- NIFI_ANALYTICS_PREDICT_INTERVAL (default: 3 mins)
- NIFI_ANALYTICS_QUERY_INTERVAL (default: 5 mins)
- NIFI_ANALYTICS_MODEL_IMPLEMENTATION
- NIFI_ANALYTICS_MODEL_SCORE_NAME (default: rSquared)
- NIFI_ANALYTICS_MODEL_SCORE_THRESHOLD (default: .90)

NAR Libraries

- NIFI_NAR_LIBRARY_PROVIDER_NIFI_REGISTRY_URL

Security & Authentication

- NIFI_SENSITIVE_PROPS_KEY
- AUTH

Single User Authentication

- SINGLE_USER_CREDENTIALS_USERNAME
- SINGLE_USER_CREDENTIALS_PASSWORD

TLS/SSL (when AUTH=tls)

- KEYSTORE_PATH (Required)
- KEYSTORE_TYPE (Required)
- KEYSTORE_PASSWORD (Required)
- KEY_PASSWORD (defaults to KEYSTORE_PASSWORD)
- TRUSTSTORE_PATH (Required)
- TRUSTSTORE_TYPE (Required)
- TRUSTSTORE_PASSWORD (Required)
- NIFI_SECURITY_USER_AUTHORIZER (default: managed-authorizer)
- NIFI_SECURITY_USER_LOGIN_IDENTITY_PROVIDER
- INITIAL_ADMIN_IDENTITY
- INITIAL_ADMIN_GROUP
- NODE_IDENTITY

LDAP (when AUTH=ldap)

- LDAP_AUTHENTICATION_STRATEGY
- LDAP_MANAGER_DN
- LDAP_MANAGER_PASSWORD
- LDAP_TLS_KEYSTORE
- LDAP_TLS_KEYSTORE_PASSWORD
- LDAP_TLS_KEYSTORE_TYPE
- LDAP_TLS_TRUSTSTORE
- LDAP_TLS_TRUSTSTORE_PASSWORD
- LDAP_TLS_TRUSTSTORE_TYPE
- LDAP_TLS_PROTOCOL
- LDAP_URL
- LDAP_USER_SEARCH_BASE
- LDAP_USER_SEARCH_FILTER
- LDAP_IDENTITY_STRATEGY
- LDAP_REFERRAL_STRATEGY

OIDC (when AUTH=oidc)

- NIFI_SECURITY_USER_OIDC_DISCOVERY_URL
- NIFI_SECURITY_USER_OIDC_CONNECT_TIMEOUT
- NIFI_SECURITY_USER_OIDC_READ_TIMEOUT
- NIFI_SECURITY_USER_OIDC_CLIENT_ID
- NIFI_SECURITY_USER_OIDC_CLIENT_SECRET
- NIFI_SECURITY_USER_OIDC_PREFERRED_JWSALGORITHM
- NIFI_SECURITY_USER_OIDC_ADDITIONAL_SCOPES
- NIFI_SECURITY_USER_OIDC_CLAIM_IDENTIFYING_USER
- NIFI_SECURITY_USER_OIDC_CLAIM_GROUPS
- NIFI_SECURITY_USER_OIDC_FALLBACK_CLAIMS_IDENTIFYING_USER
- NIFI_SECURITY_USER_OIDC_TRUSTSTORE_STRATEGY
- NIFI_SECURITY_USER_OIDC_TOKEN_REFRESH_WINDOW

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, Apache NiFi becomes available at the specified hostname like `https://example.com`.

To get started, open the URL with a web browser to log in to the instance.

To log in to the instance use your configured credentials, or the randomly generated ones printed to the log.

Log Example:

```bash
Nov 20 13:20:24 server mash-nifi[1990462]: Generated Username [d4a56f91-fcaf-4f6b-b83f-b21ea177c25b]
Nov 20 13:20:24 server mash-nifi[1990462]: Generated Password [tlRhxiDso0cvWrLROwtkagpmk1Qwx1Rt]
```

## Troubleshooting

User guide is available on [this page](https://nifi.apache.org/docs/nifi-docs/html/user-guide.html).

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu nifi` (or how you/your playbook named the service, e.g. `mash-nifi`).
