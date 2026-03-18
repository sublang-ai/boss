<!-- SPDX-License-Identifier: Apache-2.0 -->
<!-- SPDX-FileCopyrightText: 2026 SubLang International <https://sublang.ai> -->

# SAND: Sandbox Image Build and Configuration

## Intent

This spec defines implementation requirements for the local
Boss sandbox image.

## Build Inputs

### SAND-1

Where the local sandbox image is built, the Dockerfile shall use
`node:22-bookworm-slim`
([DR-001 §1](../../decisions/001-sandbox-architecture.md#1-oci-container-as-the-sandbox-boundary)).

### SAND-2

Where agent runtimes are installed, the build shall install
baseline agent CLIs (claude, codex) via mise using npm and
github backends. On-demand agent CLIs (gemini, opencode) shall
be declared in a separate on-demand config, locked at build time,
but not pre-installed
([DR-001 Context](../../decisions/001-sandbox-architecture.md#context),
[DR-004 §3](../../decisions/004-user-tool-provisioning.md)).

### SAND-3

Where agent names are resolved for CLI sessions, the mapping
shall be:
`claude -> claude`,
`codex -> codex`,
`gemini -> gemini`,
`opencode -> opencode`
([DR-002 Workspace Model](../../decisions/002-iteron-cli-commands.md#workspace-model)).

## Runtime Defaults

### SAND-4

Where the image is built, runtime defaults shall include user
`boss` (`uid=1000`, `gid=1000`), `tini` as PID 1, `bash`
as the default command, and `en_US.UTF-8` locale (`LANG` and
`LC_ALL`)
([DR-001 §1](../../decisions/001-sandbox-architecture.md#1-oci-container-as-the-sandbox-boundary)).

### SAND-5

Where the image is built, it shall remove SUID/SGID bits and
provision default config files for agents and tmux at:
`/home/boss/.claude.json`,
`/home/boss/.claude/settings.json`,
`/home/boss/.claude/CLAUDE.md`,
`/home/boss/.codex/config.toml`,
`/home/boss/.codex/AGENTS.md`,
`/home/boss/.gemini/settings.json`,
`/home/boss/.gemini/GEMINI.md`,
`/home/boss/.config/opencode/opencode.json`,
`/home/boss/.config/opencode/AGENTS.md`,
`/etc/tmux.conf`, and
`/home/boss/.tmux.conf`
([DR-001 §1](../../decisions/001-sandbox-architecture.md#1-oci-container-as-the-sandbox-boundary),
[DR-001 §3](../../decisions/001-sandbox-architecture.md#3-authentication)).

## Build Script

### SAND-6

Where `scripts/build-image.sh` runs a native build, it shall
select a functional runtime (Podman or Docker) and build
`boss-sandbox:<tag>` from `image/`.

### SAND-7

Where `scripts/build-image.sh` runs in multi-arch mode, the
script shall require `--push` and publish a
`linux/amd64` + `linux/arm64` manifest.

## Image Size

### SAND-8

Where a sandbox image is published, the release process shall
enforce a per-architecture compressed image size budget of at
most 700 MiB, measured as the sum of
compressed layer sizes from the registry manifest for each target
platform.

## Headless Authentication

### SAND-9

Where the sandbox image is built, container defaults shall set
`NO_BROWSER=true` for headless Gemini OAuth
([DR-001 §3](../../decisions/001-sandbox-architecture.md#3-authentication)).

### SAND-10

Where Boss resolves host OpenCode credentials, the supported
source path shall be
`$XDG_DATA_HOME/opencode/auth.json` when `XDG_DATA_HOME` is set,
otherwise `~/.local/share/opencode/auth.json`
([DR-001 §3](../../decisions/001-sandbox-architecture.md#3-authentication)).

### SAND-11

Where the supported host OpenCode credential file exists at
container start, launch behavior shall include a mapped credential
file at `/home/boss/.local/share/opencode/auth.json` that is
readable and writable by the container runtime user, including
credential refresh
([DR-001 §3](../../decisions/001-sandbox-architecture.md#3-authentication)).

### SAND-12

Where the supported host OpenCode credential file is absent at
container start, launch behavior shall omit any OpenCode
credential file mapping
([DR-001 §3](../../decisions/001-sandbox-architecture.md#3-authentication)).

## User-Local Tool Layer

### SAND-13

Where the image is built, the Dockerfile shall create
`/home/boss/.local/bin` owned by `boss:boss` and set `PATH`
to `~/.local/share/mise/shims:~/.local/bin:~/.local/share/npm-global/bin:~/.local/share/cargo/bin:$PATH` via `ENV`
([DR-001 §6](../../decisions/001-sandbox-architecture.md#6-user-local-tool-layer),
[DR-004 §3](../../decisions/004-user-tool-provisioning.md),
[DR-005 §1](../../decisions/005-package-manager-environment.md#1-xdg-environment-variables)).

### SAND-14

Where `boss start` launches a container, the start sequence
shall run `mkdir -p /home/boss/.local/bin` inside the container
after volume mount, ensuring the directory exists on pre-existing
`boss-data` volumes
([DR-001 §6](../../decisions/001-sandbox-architecture.md#6-user-local-tool-layer)).

## Vulnerability Scanning

### SAND-15

Where the image is built, globally installed npm packages and their
transitive dependencies shall have no known CRITICAL or HIGH CVEs
except as permitted by [SAND-16](#sand-16).

### SAND-16

Where a CRITICAL or HIGH CVE has no available fix in the base
distribution, or where a fix exists upstream but cannot be applied
without breaking the dependent package (e.g., npm internal
dependencies referencing private registries), the CVE ID, a
justification, and a concrete reassessment trigger (e.g., next
upstream release, base image upgrade) shall be recorded in the
accepted-CVE list at `image/.trivyignore`.

### SAND-17

Where the image CI workflow runs after a successful build, a
vulnerability scanner shall fail the pipeline on any CRITICAL or HIGH
CVE not listed in the accepted-CVE list.

### SAND-18

Where a local vulnerability scan is needed, `scripts/scan-image.sh`
shall accept an optional image tag, apply the accepted-CVE list, and
exit non-zero on any unaccepted CRITICAL or HIGH CVE.

## Tmux Configuration

### SAND-19

Where the image is built, tmux system configuration at `/etc/tmux.conf`
shall enable OSC 52 clipboard passthrough (`set-clipboard on`,
`allow-passthrough on`).

### SAND-20

Where the image is built, tmux system configuration shall set
`default-terminal` to `screen-256color` for agent TUI compatibility
([DR-001 §2](../../decisions/001-sandbox-architecture.md#2-tmux-mapping)).

### SAND-21

Where the image is built, tmux user configuration at `~/.tmux.conf`
shall contain only override directives, deferring defaults to the
system configuration.

### SAND-22

Where the image is built, tmux system configuration shall bind
`MouseDragEnd1Pane` to `copy-selection-and-cancel` in both
`copy-mode` and `copy-mode-vi`, and bind a prefix key to toggle
`mouse` mode on/off. The toggle shall persist the choice to
`~/.boss-prefs`.

### SAND-23

Where user preferences are persisted inside the container,
they shall be stored in `~/.boss-prefs` using `key=value`
format (one entry per line). This file lives on the
`boss-data` volume and survives container restarts and image
updates. Image-level defaults (e.g., `/etc/tmux.conf`) read
this file on startup to restore saved preferences.

## User-Space Tool Provisioning

### SAND-24

Where the image is built, `mise` shall be installed at a pinned
version and its binary shall be on `PATH`
([DR-004 §2](../../decisions/004-user-tool-provisioning.md)).

### SAND-25

Where the image is built, `/etc/mise/config.toml` shall declare
baseline agent CLIs (claude, codex) and enforce a backend denylist
allowing only `npm` and `github` backends
([DR-004 §3, §8](../../decisions/004-user-tool-provisioning.md)).

### SAND-26

Where the image is built, `/etc/mise/mise.lock` shall contain
resolved versions for all declared baseline tools
([DR-004 §6](../../decisions/004-user-tool-provisioning.md)).

### SAND-27

Where the image is built, baseline agent CLI binaries shall be
invocable via mise shims on `PATH`
([DR-004 §3](../../decisions/004-user-tool-provisioning.md)).

### SAND-28

Where the image is built, `/etc/mise/ondemand.toml` shall
declare on-demand agent CLIs (gemini, opencode) with the same
backend denylist as the baseline config. `/etc/mise/ondemand.lock`
shall contain resolved versions locked at build time. These
tools shall not be pre-installed during the image build.

### SAND-29

Where a user runs `boss open` with an on-demand agent, if the
agent binary is not yet installed, the CLI shall install it
from the on-demand config+lockfile before launching the tmux
session. A progress message shall be displayed during
installation.

### SAND-30

Where on-demand agents have been previously installed, the
entrypoint reconciliation shall reconcile them using the
on-demand config+lockfile, following the same locked-install
pattern as baseline system tools.

## DR-005 Package Manager Environment

### SAND-31

Where the image is built, Dockerfile `ENV` shall define the DR-005
package-manager paths:
`XDG_CONFIG_HOME`, `XDG_CACHE_HOME`, `XDG_DATA_HOME`, `XDG_STATE_HOME`,
`PYTHONUSERBASE`, `PIP_USER`, `NPM_CONFIG_PREFIX`, `GOPATH`, `GOBIN`,
`CARGO_HOME`, and `RUSTUP_HOME`
([DR-005 §1](../../decisions/005-package-manager-environment.md#1-xdg-environment-variables)).

### SAND-32

Where the image is built, `/usr/local/bin/sudo` shall be a root-owned
mock shim that runs allowed forms unprivileged with a context line and
rejects user/group switching and interactive shell forms
([DR-005 §2](../../decisions/005-package-manager-environment.md#2-mock-sudo-shim)).

### SAND-33

Where the image is built, `/opt/defaults/` shall contain image-owned
default dotfiles/configs, and container startup entrypoint behavior
shall seed missing files from `/opt/defaults/` into `$HOME` without
overwriting existing files
([DR-005 §3](../../decisions/005-package-manager-environment.md#3-entrypoint-defaults-seeding)).

### SAND-34

Where the image is built, Dockerfile `ENV` shall set
`BOSS_IMAGE_VERSION` from a required build argument. Startup entrypoint
behavior shall record this value at
`$XDG_STATE_HOME/.boss-image-version` and emit a diagnostic when the
stored value changes
([DR-005 §3](../../decisions/005-package-manager-environment.md#3-entrypoint-defaults-seeding)).

### SAND-35

Where the image is built, baseline developer CLIs shall be preinstalled
and executable on `PATH`: `gpg`, `tree`, `gh`, and `glab`
([DR-005](../../decisions/005-package-manager-environment.md)).

### SAND-36

Where the image startup entrypoint runs, it shall perform mise
reconciliation by:
- trusting system + user configs
- running a locked system phase from a writable temp project copy of
  `/etc/mise/config.toml` + `/etc/mise/mise.lock`
- running a locked user phase with `/etc/mise/config.toml` ignored when
  `~/.config/mise/mise.lock` exists
- emitting an actionable soft warning when user tools are declared without
  a user lockfile
and record reconciliation state at `$XDG_STATE_HOME/.boss-mise-reconcile.state`.
The state shall include a fingerprint derived from image version plus
config+lock hashes, and a `should_warn` flag deduped by
`(fingerprint, failed_step, error_class)`
([DR-004 §5](../../decisions/004-user-tool-provisioning.md#5-reconciliation-policy-on-container-start)).
