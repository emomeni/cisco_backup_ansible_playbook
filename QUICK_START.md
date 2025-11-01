# Quick Start Guide

## Initial Setup

1. **Install Ansible Collections:**
   ```bash
   ansible-galaxy collection install cisco.ios cisco.nxos
   ```

2. **Configure Inventory:**
   - Edit `inventory.ini` with your device IPs
   - Update credentials (or use vault - see below)

3. **Test Connectivity:**
   ```bash
   ansible cisco_devices -m ping -i inventory.ini
   ```

4. **Run Backup:**
   ```bash
   ansible-playbook cisco_backup.yaml
   ```

## Using Ansible Vault (Recommended)

1. **Encrypt vault file:**
   ```bash
   ansible-vault encrypt vault.yaml
   ```

2. **Edit encrypted vault:**
   ```bash
   ansible-vault edit vault.yaml
   ```

3. **Run with vault:**
   ```bash
   ansible-playbook cisco_backup.yaml --ask-vault-pass
   ```

4. **Use vault password file:**
   ```bash
   echo "your-vault-password" > .vault_pass
   chmod 600 .vault_pass
   ansible-playbook cisco_backup.yaml --vault-password-file .vault_pass
   ```

## Directory Structure

```
cisco-backup/
├── ansible.cfg                  # Ansible configuration
├── inventory.ini                # Device inventory
├── cisco_backup.yaml            # Main backup playbook
├── vault.yaml                   # Encrypted credentials
├── group_vars/                  # Group variables
│   ├── all.yaml                 # Global variables
│   ├── ios_devices.yaml         # IOS-specific variables
│   └── nxos_devices.yaml        # NX-OS-specific variables
├── host_vars/                   # Host-specific variables
│   ├── core-router-01.yaml      # Example host variables
│   └── nexus-core-01.yaml       # Example host variables
├── backups/                     # Backup storage (auto-created)
│   └── YYYYMMDD_HHMMSS/         # Timestamped backup folders
└── logs/                        # Log files
    └── ansible.log              # Ansible execution logs
```

## Common Commands

### Backup specific devices
```bash
ansible-playbook cisco_backup.yaml --limit core-router-01
```

### Backup only IOS devices
```bash
ansible-playbook cisco_backup.yaml --limit ios_devices
```

### Dry run (check mode)
```bash
ansible-playbook cisco_backup.yaml --check
```

### Verbose output
```bash
ansible-playbook cisco_backup.yaml -vvv
```

## Troubleshooting

### View logs
```bash
tail -f ansible.log
```

### Test single device
```bash
ansible core-router-01 -m ping -i inventory.ini -vvv
```

### Verify collections
```bash
ansible-galaxy collection list | grep cisco
```

For detailed documentation, see README.md