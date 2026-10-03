[![CI](https://github.com/ngine-io/ansible-role-tailscale/actions/workflows/ci.yml/badge.svg)](https://github.com/ngine-io/ansible-role-tailscale/actions/workflows/ci.yml)

# Ansible Role: Tailscale

Installs [Tailscale](https://tailscale.com/) on Debian Linux (12 bookworm, 13 trixie).

## Requirements

Ansible >= 2.17 with the `ansible.posix` collection.

## Installation

Via `requirements.yml`:

```yaml
---
# file: requirements.yml
roles:
  - name: ngine_io.tailscale
    version: v0.2.0
```

To install:

```
ansible-galaxy install -r requirements.yml
```
## License

MIT / Apache2

## Author Information

This role was created in 2024 by [René Moser](https://renemoser.net).
