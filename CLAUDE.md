# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Ansible role `ngine_io.tailscale` (published to Ansible Galaxy) that installs Tailscale on Debian (bookworm, trixie) from the official pkgs.tailscale.com apt repo, enables IP forwarding, and optionally joins the tailnet. Requires Ansible >= 2.17.

## Commands

```sh
# Lint (same as CI)
pip3 install ansible ansible-lint
ansible-lint .

# Molecule test (Docker driver, geerlingguy/docker-<distro>-ansible images)
python3 -m pip install ansible molecule "molecule-plugins[docker]" docker
molecule test                         # full create/converge/idempotence/destroy cycle
molecule converge                     # iterate on a running instance
MOLECULE_DISTRO=debian12 molecule test   # default is debian13
```

The molecule converge playbook references the role by its Galaxy name `ngine_io.tailscale`; CI checks the repo out into a directory with that name (`path: ngine_io.tailscale`), so role resolution relies on it.

## Architecture / conventions

- Everything lives in `tasks/main.yml`; there are no sub-task files or templates.
- Variable naming: public variables use the `tailscale__` prefix (double underscore) and are defined in `defaults/main.yml`; internal/registered facts use a leading underscore (`_tailscale__...`).
- Flow: apt key + deb822 repo (`/etc/apt/sources.list.d/tailscale.sources`; the legacy `tailscale.list` is removed) → install package (pinned via `tailscale__version` if set) → sysctl IPv4/IPv6 forwarding in `/etc/sysctl.d/99-tailscale.conf` (trixie has no `/etc/sysctl.conf`; notifies `Restart tailscaled` handler) → start service → read `tailscale status --json` → run `tailscale up` only when `tailscale__auth_token` is set and `BackendState == 'NeedsLogin'` (this keeps the role idempotent). `tailscale__subnets` are passed as `--advertise-routes`.
- Only external collection is `ansible.posix` (sysctl). There is no collections `requirements.yml`; CI relies on the full `ansible` package bundling it.
- `when` conditions must evaluate to real booleans (ansible-core 2.19+ errors otherwise), hence `is truthy` on string vars. The `tailscale up` task is `no_log` because it carries the auth key.
- Network-dependent tasks use the `register`/`until result is succeeded`/`retries: 5` pattern; keep it for new download/install tasks.

## Release

Publishing a GitHub release triggers `.github/workflows/release.yml`, which imports the role into Galaxy (needs the `ANSIBLE_GALAXY_API_KEY` secret). Tags follow `vX.Y.Z`.
