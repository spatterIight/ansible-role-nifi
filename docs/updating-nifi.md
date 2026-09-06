<!--
SPDX-FileCopyrightText: 2025 spatterlight

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Updating the role for a new Apache NiFi release

Bumping `nifi_version` is not a one-line change. The role ships upstream NiFi
config files verbatim in [`files/conf/`](../files/conf/) and a pair of custom
startup scripts in [`files/scripts/`](../files/scripts/). On a new release the
config files usually need to be refreshed from the new image, but the scripts
deliberately diverge from upstream and should only be touched under specific
conditions (see [Step 4](#step-4--reconcile-filesscripts)).

The role's installer (`tasks/install.yml`) applies customizations on top of
`files/conf/` at install time via `community.general.xml` (for XML files) and
`ansible.builtin.lineinfile` (for properties files), driven by the
`nifi_*_replacements` variables in `defaults/main.yml`. That means upstream
changes to the shipped config files do not need to be merged by hand — they
just need to land in `files/conf/` and the Ansible-side overlays will be
reapplied on next install.

## When to run this procedure

On each new upstream Apache NiFi **minor or major** release. Renovate is
configured to watch the `apache/nifi` Docker tag (see the `# renovate:` comment
in [`defaults/main.yml`](../defaults/main.yml)) and opens the version-bump pull
request automatically; the conf/script reconciliation below is what still has to
be done by hand on top of that pull request.

**Patch releases do not normally need any of this**, and are automerged once CI
is green (see [`.github/renovate.json`](../.github/renovate.json)). That is not
a matter of trust: the Molecule scenario compares the files this role ships in
[`files/conf/`](../files/conf/) against the image `nifi_version` pins and fails
the build if they have drifted apart. So if upstream ever does change a shipped
configuration file in a patch release, the automerge is blocked and this
procedure has to be run after all. Measured across `2.7.0` → `2.7.1` → `2.7.2`
and `1.28.0` → `1.28.1`, upstream has not changed either the configuration files
or the startup scripts in a patch release.

## Prerequisites

- Docker installed locally
- A working clone of this role

## Step 1 — Bump `nifi_version`

Edit [`defaults/main.yml`](../defaults/main.yml) and set `nifi_version` to the
new release (or let Renovate's PR stand in for this step). This drives the
container tag via `nifi_container_image_tag`.

## Step 2 — Stage the new upstream image for extraction

Create (but do not run) a container from the target image so files can be
copied out:

```sh
docker create --name nifi-extract apache/nifi:<new-version>
mkdir -p /tmp/nifi-new
```

## Step 3 — Reconcile `files/conf/`

For each file the role ships — the same list as in `tasks/install.yml`:

- `authorizers.xml`
- `bootstrap.conf`
- `logback.xml`
- `login-identity-providers.xml`
- `nifi.properties`
- `state-management.xml`
- `zookeeper.properties`

Copy the upstream version out and diff it against the role's copy:

```sh
docker cp nifi-extract:/opt/nifi/nifi-current/conf/<file> /tmp/nifi-new/
diff -u files/conf/<file> /tmp/nifi-new/<file>
```

**Default action: copy the new file over verbatim.** The role reapplies its
customizations on top at install time, so there is no need to port changes by
hand.

### Sanity-check the default replacements

After refreshing `files/conf/`, cross-check the keys the role pins against the
new upstream files:

- `nifi_properties_replacements_default` in
  [`defaults/main.yml`](../defaults/main.yml) — every key referenced there
  should still exist in (or still be a valid insertion target for) the new
  `nifi.properties`. If upstream renames or removes a key the role pins, the
  default replacements need to be updated to match.
- `nifi_login_identity_providers_xml_replacements_default` in
  [`defaults/main.yml`](../defaults/main.yml) — the XPath expressions there
  must still resolve against the new `login-identity-providers.xml` structure.

## Step 4 — Reconcile `files/scripts/`

```sh
docker cp nifi-extract:/opt/nifi/scripts/common.sh /tmp/nifi-new/
docker cp nifi-extract:/opt/nifi/scripts/start.sh  /tmp/nifi-new/
diff -u files/scripts/common.sh /tmp/nifi-new/common.sh
diff -u files/scripts/start.sh  /tmp/nifi-new/start.sh
```

**Do not copy these over verbatim.** The role's copies are deliberately
stripped down: upstream's scripts contain `prop_replace` calls that mutate
`nifi.properties` at container start from environment variables, for example:

```sh
prop_replace 'nifi.web.proxy.host' "${NIFI_WEB_PROXY_HOST}"
```

The role's installer already applies those changes statically from Ansible
variables, so pulling those calls back in would cause the container to rewrite
files the role has just configured.

### Decision rule when diffs appear

- Changes that are purely about dynamic/runtime configuration driven by
  `NIFI_*` environment variables — anything invoking `prop_replace`,
  `xmlstarlet`, `sed -i` against files under `${NIFI_HOME}/conf/`, or similar
  — **skip**. The role handles these at install time.
- Changes unrelated to runtime config — bug fixes, new `trap` handlers,
  JVM-launch plumbing, logging, `PATH` setup, and the like — port them into
  the role's copy by hand.

When in doubt, err toward skipping and note the divergence in the PR
description.

## Step 5 — Clean up

```sh
docker rm nifi-extract
rm -rf /tmp/nifi-new
```

## Step 6 — Verify

- `just prek-run-on-all` (runs ansible-lint, markdownlint, codespell and reuse
  via [prek](https://prek.j178.dev/)) must pass. Note that `files/conf/` and
  `files/scripts/` are excluded from these hooks (see
  [`.pre-commit-config.yaml`](../.pre-commit-config.yaml)), so refreshed
  upstream files are left byte-for-byte as shipped.
- End-to-end: run the Molecule scenario, which installs the role, starts the
  container, logs in with the configured credentials, asserts that the running
  process reports the pinned version, and asserts that `files/conf/` matches the
  pinned image — so a forgotten Step 3 fails here rather than in production
  (see [`molecule/README.md`](../molecule/README.md)):

  ```sh
  molecule test --scenario-name default
  ```

  See [`molecule/README.md`](../molecule/README.md) for setup, and
  [`molecule/default/molecule.yml`](../molecule/default/molecule.yml) for the
  `nifi_debug_service_enabled` flag, which dumps `journalctl -xeu nifi.service`
  when the scenario fails.
- Re-check `git diff defaults/main.yml files/conf/ files/scripts/` before
  committing — changes should be limited to the version bump, refreshed
  upstream conf, and any intentional script ports from
  [Step 4](#step-4--reconcile-filesscripts).
