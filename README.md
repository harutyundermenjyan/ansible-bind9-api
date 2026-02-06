# Ansible Role: bind9_api

Ansible role for deploying BIND9 DNS server with REST API management.

## Requirements

- Ubuntu 22.04+ (Debian-based)
- Ansible 2.14+

## Installation

### Via requirements.yml (recommended)

```yaml
# requirements.yml
roles:
  - name: bind9_api
    src: https://github.com/harutyundermenjyan/ansible-bind9-api.git
    version: main
    scm: git
```

```bash
ansible-galaxy install -r requirements.yml
```

### Manual

```bash
ansible-galaxy install git+https://github.com/harutyundermenjyan/ansible-bind9-api.git,main
```

## Role Variables

### API Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `bind9_api_port` | `8080` | API listen port |
| `bind9_api_host` | `0.0.0.0` | API listen address |
| `bind9_api_workers` | `4` | Uvicorn worker count |
| `bind9_api_log_level` | `INFO` | API log level |
| `bind9_api_repo` | GitHub URL | Git repository for API source |
| `bind9_api_version` | `main` | Git branch/tag to deploy |
| `bind9_api_key` | auto-generated | API authentication key |

### BIND9 Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `bind9_config_dir` | `/etc/bind` | BIND9 config directory |
| `bind9_zones_dir` | `/var/lib/bind` | Zone files directory |
| `bind9_allow_new_zones` | `true` | Enable dynamic zone management |
| `bind9_stats_port` | `8053` | Statistics channel port |
| `bind9_tsig_algorithm` | `hmac-sha256` | TSIG key algorithm |
| `bind9_configure_apparmor` | `true` | Configure AppArmor for BIND9 |

### BIND9 Options

```yaml
bind9_options:
  allow_query:
    - "any"
  allow_recursion:
    - "localhost"
    - "localnets"
  dnssec_validation: "auto"
  listen_on:
    - "any"
```

See `defaults/main.yml` for all available variables.

## Usage

```yaml
# playbook.yml
- hosts: dns_servers
  become: true
  roles:
    - bind9_api
```

### With custom variables

```yaml
- hosts: dns_servers
  become: true
  roles:
    - role: bind9_api
      vars:
        bind9_api_port: 9090
        bind9_options:
          allow_query:
            - "10.0.0.0/8"
          allow_recursion:
            - "localhost"
```

## What Gets Deployed

- **BIND9** DNS server with dynamic zone support
- **BIND9 REST API** (Python/FastAPI) for programmatic DNS management
- **AppArmor** configuration for BIND9
- **UFW** firewall rules (DNS + API ports)
- **RNDC** and **TSIG** keys for secure management
- **Systemd** service for the API

## After Deployment

- **API Endpoint**: `http://<server-ip>:<bind9_api_port>`
- **API Docs**: `http://<server-ip>:<bind9_api_port>/docs`
- **Credentials**: `/opt/bind9-api/.env` on the server

## Related Projects

- [bind9-api](https://github.com/harutyundermenjyan/bind9-api) - REST API source
- [terraform-provider-bind9](https://github.com/harutyundermenjyan/terraform-provider-bind9) - Terraform provider

## License

MIT
