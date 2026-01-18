# BIND9 + REST API Ansible Playbook

Fully automated deployment of BIND9 DNS server with REST API for Terraform/OpenTofu management.

## Features

- ✅ **Automated Installation** - BIND9 + Python API in one command
- ✅ **Secure Key Generation** - API key, RNDC key, TSIG/DDNS key auto-generated
- ✅ **Production Ready** - Proper permissions, logging, AppArmor
- ✅ **Terraform Ready** - Works with [terraform-provider-bind9](https://github.com/harutyundermenjyan/terraform-provider-bind9)

## Quick Start

### 1. Install Ansible

```bash
# macOS
brew install ansible

# Ubuntu/Debian
sudo apt install ansible

# pip
pip install ansible
```

### 2. Configure Inventory

Edit `inventories/production/hosts.yml`:

```yaml
all:
  children:
    dns_servers:
      hosts:
        dns1:
          ansible_host: 192.168.1.10    # ← Your DNS server IP
          ansible_user: ubuntu
```

### 3. Run Playbook

```bash
# Test connection
ansible -i inventories/production/hosts.yml all -m ping

# Deploy
ansible-playbook -i inventories/production/hosts.yml site.yml
```

### 4. Get Credentials

After deployment, credentials are saved to `credentials/<hostname>.txt`:

```bash
cat credentials/dns1.txt
```

## Configuration

### Default Variables

Override in `inventories/production/hosts.yml` or `group_vars/`:

| Variable | Default | Description |
|----------|---------|-------------|
| `bind9_api_port` | `8080` | API listen port |
| `bind9_api_key` | auto-generated | API authentication key |
| `bind9_zones_dir` | `/var/lib/bind` | Zone files directory |
| `bind9_allow_new_zones` | `true` | Enable dynamic zone creation |

### Custom API Key

To use a specific API key instead of auto-generating:

```yaml
# inventories/production/hosts.yml
dns1:
  ansible_host: 192.168.1.10
  bind9_api_key: "your-secure-key-here"
```

### Multiple Servers

```yaml
dns_servers:
  hosts:
    dns1:
      ansible_host: 192.168.1.10
    dns2:
      ansible_host: 192.168.1.11
    dns3:
      ansible_host: 192.168.1.12
```

## Directory Structure

```
ansible-bind9-api/
├── site.yml                    # Main playbook
├── ansible.cfg                 # Ansible configuration
├── inventories/
│   └── production/
│       └── hosts.yml           # Server inventory
├── group_vars/
│   └── all.yml                 # Global variables
├── credentials/                # Generated credentials (gitignored)
│   └── dns1.txt
└── roles/
    └── bind9_api/
        ├── defaults/main.yml   # Default variables
        ├── tasks/
        │   ├── main.yml        # Main task entry
        │   ├── keys.yml        # Key generation
        │   ├── bind9.yml       # BIND9 setup
        │   ├── api.yml         # API setup
        │   ├── firewall.yml    # Firewall rules
        │   └── validate.yml    # Validation tests
        ├── templates/          # Configuration templates
        └── handlers/main.yml   # Service handlers
```

## Using with Terraform

After deployment, configure your Terraform provider:

```hcl
terraform {
  required_providers {
    bind9 = {
      source  = "harutyundermenjyan/bind9"
      version = ">= 1.0.3"
    }
  }
}

provider "bind9" {
  endpoint = "http://192.168.1.10:8080"  # From credentials file
  api_key  = "YOUR_API_KEY"              # From credentials file
}
```

## GitLab CI (Manual Deploy)

This repo includes a production-ready GitLab CI pipeline with a **manual** deploy job.

### Required CI/CD variables

Create these variables in **GitLab → Settings → CI/CD → Variables**:

- `SSH_PRIVATE_KEY` (masked, protected)  
  Private key with access to your DNS servers.
- `SSH_KNOWN_HOSTS` (optional, masked)  
  Output of `ssh-keyscan -H <server_ip>` for your hosts.

The deploy job runs:

```bash
ansible-playbook -i inventories/production/hosts.yml site.yml
```

If your inventory is different for CI, set it in `inventories/production/hosts.yml`
or override it with a project variable and pass it via `ANSIBLE_INVENTORY`.

## Security Notes

1. **API Key** - Generated securely using `secrets.token_urlsafe(32)`
2. **TSIG Keys** - Generated with `tsig-keygen` using HMAC-SHA256
3. **File Permissions** - Sensitive files are mode 0600/0640
4. **Credentials** - Saved locally, not on server, gitignored

## Troubleshooting

### Check Service Status

```bash
# On server
sudo systemctl status named
sudo systemctl status bind9-api
```

### View Logs

```bash
# BIND9 logs
sudo journalctl -u named -f

# API logs
sudo journalctl -u bind9-api -f
```

### Validate Configuration

```bash
# BIND9 config
sudo named-checkconf

# Test API
curl -H "X-API-Key: YOUR_KEY" http://YOUR_SERVER:8080/health
```

## License

MIT
