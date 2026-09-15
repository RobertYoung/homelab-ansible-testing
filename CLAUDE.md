# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Single-host Ansible project for testing purposes, targeting `debian-test1.local.iamrobertyoung.co.uk`. Secrets are stored in AWS SSM Parameter Store.

## Key Commands

All commands require AWS credentials via aws-vault for SSM parameter lookups.

```bash
# Test connectivity
aws-vault exec iamrobertyoung:home-assistant-production:p -- ansible debian_test -m ping

# Run full playbook
aws-vault exec iamrobertyoung:home-assistant-production:p -- ansible-playbook playbooks/site.yml

# Run specific role only
aws-vault exec iamrobertyoung:home-assistant-production:p -- ansible-playbook playbooks/site.yml --tags <role-name>

# Install external dependencies (roles)
aws-vault exec iamrobertyoung:home-assistant-production:p -- ansible-galaxy install -r requirements.yml -p .roles

# Lint
yamllint .
ansible-lint
```

## Architecture

### Deployment Stack
The main playbook (`playbooks/site.yml`) applies roles in order. Add roles as needed.

### Secrets Management
Secrets are stored in AWS SSM Parameter Store (region: **eu-west-1**) under the
`/homelab/*` namespace. Ansible retrieves them via
`lookup('aws_ssm', '/homelab/<service>/<name>', region='eu-west-1')`.

This repo read `/home-server/*` in eu-west-2 until 2026-09-15, and was the last
of the twelve `homelab-*` repos to do so. That mattered rather than being
untidy: `home-server` is being decommissioned, and a `step-ca-client` lookup
that cannot resolve is a certificate that cannot be renewed. `debian-test1`'s
client certificate expired on 2025-08-12 and stayed expired for thirteen
months, during which the host shipped no logs anywhere — `step ca renew`
authenticates with the certificate it is replacing, so once one lapses it can
never recover itself. See RobertYoung/homelab-talos#219.

### External Dependencies
Dependencies in `requirements.yml` are installed to `.roles/` (gitignored). External roles use SSH git URLs requiring SSH keys.

### Tool Versions (mise.toml)
- ansible 13.1.0
- pipx 1.8.0
- uv 0.9.18
