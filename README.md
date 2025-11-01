# Cisco Network Device Backup Automation

An enterprise-grade Ansible playbook for automated backup of Cisco network devices including IOS, IOS-XE, and NX-OS platforms with timestamped storage and automated retention management.

## Features

- ✅ **Multi-Platform Support**: IOS, IOS-XE, and NX-OS devices
- ✅ **Timestamped Backups**: Organized by date/time for easy tracking
- ✅ **Automated Cleanup**: Configurable retention policy
- ✅ **Thread-Safe Logging**: Prevents concurrent write conflicts
- ✅ **Error Handling**: Comprehensive failure tracking and recovery
- ✅ **Audit Trail**: Detailed summary logs for each backup run
- ✅ **Self-Protection**: Prevents deletion of current backup

## Prerequisites

### System Requirements
- Ansible 2.9 or higher
- Python 3.6 or higher
- Network connectivity to target devices

### Required Ansible Collections
```bash
ansible-galaxy collection install cisco.ios
ansible-galaxy collection install cisco.nxos
```

### Network Device Requirements
- SSH enabled on all devices
- Valid credentials with appropriate privileges
- Enable mode access (for IOS/IOS-XE devices)

## Installation

1. **Clone or download the playbook files:**
```bash
mkdir cisco-backup && cd cisco-backup
```

2. **Install required collections:**
```bash
ansible-galaxy collection install cisco.ios cisco.nxos
```

3. **Configure your inventory** (see Configuration section)

## Configuration

### Inventory File (`inventory.ini`)

```ini
[ios_devices]
router1 ansible_host=192.168.1.10
router2 ansible_host=192.168.1.11
switch1 ansible_host=192.168.1.20

[nxos_devices]
nexus1 ansible_host=192.168.1.30
nexus2 ansible_host=192.168.1.31

[cisco_devices:children]
ios_devices
nxos_devices

[ios_devices:vars]
ansible_network_os=cisco.ios.ios
ansible_connection=network_cli

[nxos_devices:vars]
ansible_network_os=cisco.nxos.nxos
ansible_connection=network_cli

[cisco_devices:vars]
ansible_user=admin
ansible_password=your_password
ansible_become=yes
ansible_become_method=enable
ansible_become_password=your_enable_password
```

### Playbook Variables

Edit `cisco_backup.yaml` to customize:

| Variable | Default | Description |
|----------|---------|-------------|
| `backup_dir` | `{{ playbook_dir }}/backups` | Backup storage location |
| `retention_days` | `30` | Days to keep old backups |

### Ansible Configuration (`ansible.cfg`)

```ini
[defaults]
inventory = ./inventory.ini
host_key_checking = False
timeout = 60
retry_files_enabled = False
gathering = explicit

[persistent_connection]
command_timeout = 60
connect_timeout = 60
```

## Usage

### Basic Backup Run

```bash
ansible-playbook cisco_backup.yaml
```

### Dry Run (Check Mode)

```bash
ansible-playbook cisco_backup.yaml --check
```

### Backup Specific Devices

```bash
ansible-playbook cisco_backup.yaml --limit router1,nexus1
```

### Backup Only IOS Devices

```bash
ansible-playbook cisco_backup.yaml --limit ios_devices
```

### Verbose Output

```bash
ansible-playbook cisco_backup.yaml -v
```

### Using Different Inventory

```bash
ansible-playbook cisco_backup.yaml -i production_inventory.ini
```

## Directory Structure

After running, backups are organized as follows:

```
backups/
├── 20251101_143022/
│   ├── backup_summary.log
│   ├── router1_20251101_143022.cfg
│   ├── router2_20251101_143022.cfg
│   ├── switch1_20251101_143022.cfg
│   ├── nexus1_20251101_143022.cfg
│   └── nexus2_20251101_143022.cfg
├── 20251101_090015/
│   └── ...
└── 20251031_235959/
    └── ...
```

## Backup Summary Log

Each backup run creates a summary log with the following format:

```
====================================
Backup Run Started: 2025-11-01T14:30:22Z
====================================
2025-11-01T14:30:25Z - router1 - cisco.ios.ios - SUCCESS
2025-11-01T14:30:28Z - router2 - cisco.ios.ios - SUCCESS
2025-11-01T14:30:31Z - switch1 - cisco.ios.ios - SUCCESS
2025-11-01T14:30:35Z - nexus1 - cisco.nxos.nxos - SUCCESS
2025-11-01T14:30:38Z - nexus2 - cisco.nxos.nxos - FAILED - Connection timeout
```

## Security Best Practices

### 1. Use Ansible Vault for Credentials

**Encrypt sensitive variables:**
```bash
ansible-vault create vault.yaml
```

**vault.yml content:**
```yaml
vault_ansible_user: admin
vault_ansible_password: SecurePassword123
vault_ansible_become_password: EnablePassword456
```

**Update inventory to use vault variables:**
```ini
[cisco_devices:vars]
ansible_user={{ vault_ansible_user }}
ansible_password={{ vault_ansible_password }}
ansible_become_password={{ vault_ansible_become_password }}
```

**Run playbook with vault:**
```bash
ansible-playbook cisco_backup.yaml --ask-vault-pass
```

### 2. Use SSH Keys (Recommended)

Configure SSH key authentication on devices and use:
```ini
ansible_ssh_private_key_file=/path/to/private/key
```

### 3. Restrict File Permissions

```bash
chmod 600 inventory.ini
chmod 600 vault.yaml
chmod 700 backups/
```

## Automated Scheduling

### Using Cron (Linux)

```bash
# Edit crontab
crontab -e

# Daily backup at 2 AM
0 2 * * * cd /path/to/cisco-backup && /usr/bin/ansible-playbook cisco_backup.yaml >> /var/log/cisco-backup.log 2>&1

# Weekly backup on Sundays at 3 AM
0 3 * * 0 cd /path/to/cisco-backup && /usr/bin/ansible-playbook cisco_backup.yaml
```

### Using Ansible Tower/AWX

1. Create a new Job Template
2. Set the playbook to `cisco_backup.yaml`
3. Configure schedule (daily/weekly/monthly)
4. Set credentials from vault

### Using systemd Timer (Linux)

**Create service file** (`/etc/systemd/system/cisco-backup.service`):
```ini
[Unit]
Description=Cisco Device Backup
After=network.target

[Service]
Type=oneshot
User=ansible
WorkingDirectory=/opt/cisco-backup
ExecStart=/usr/bin/ansible-playbook backup_playbook.yml
```

**Create timer file** (`/etc/systemd/system/cisco-backup.timer`):
```ini
[Unit]
Description=Daily Cisco Backup Timer

[Timer]
OnCalendar=daily
OnCalendar=02:00
Persistent=true

[Install]
WantedBy=timers.target
```

**Enable and start:**
```bash
systemctl enable cisco-backup.timer
systemctl start cisco-backup.timer
systemctl list-timers
```

## Troubleshooting

### Common Issues

#### 1. Connection Timeout
**Error:** `Failed to connect to the host via ssh`

**Solutions:**
- Verify device IP addresses in inventory
- Check SSH connectivity: `ssh admin@192.168.1.10`
- Increase timeout in `ansible.cfg`:
  ```ini
  [defaults]
  timeout = 120
  ```

#### 2. Authentication Failed
**Error:** `Authentication failed`

**Solutions:**
- Verify credentials in inventory
- Check enable password for IOS devices
- Test manual login to device
- Use `-vvv` for detailed debug output

#### 3. Permission Denied
**Error:** `Permission denied creating directory`

**Solutions:**
- Check file system permissions on backup directory
- Ensure user has write access to playbook directory
- Run: `chmod 755 backups/`

#### 4. Module Not Found
**Error:** `ERROR! couldn't resolve module/action 'cisco.ios.ios_config'`

**Solutions:**
- Install collections: `ansible-galaxy collection install cisco.ios cisco.nxos`
- Verify installation: `ansible-galaxy collection list`

#### 5. No Backups Being Cleaned Up
**Issue:** Old backups remain after retention period

**Solutions:**
- Verify `retention_days` variable is set
- Check directory name format matches: `YYYYMMDD_HHMMSS`
- Run with `-v` to see cleanup process

### Debug Commands

```bash
# Test connectivity to devices
ansible cisco_devices -m ping -i inventory.ini

# List all devices in inventory
ansible-inventory -i inventory.ini --list

# Test single device backup
ansible-playbook cisco_backup.yaml --limit router1 -vvv

# Check Ansible version
ansible --version

# Verify collections
ansible-galaxy collection list | grep cisco
```

## Backup Verification

### Manual Verification

```bash
# Check backup file exists and is not empty
ls -lh backups/20251101_143022/router1_20251101_143022.cfg

# View backup summary
cat backups/20251101_143022/backup_summary.log

# Count successful backups
grep SUCCESS backups/20251101_143022/backup_summary.log | wc -l
```

### Automated Verification Script

```bash
#!/bin/bash
LATEST_BACKUP=$(ls -td backups/*/ | head -1)
FAILED=$(grep -c FAILED "$LATEST_BACKUP/backup_summary.log" || echo 0)

if [ $FAILED -gt 0 ]; then
    echo "⚠️  $FAILED device(s) failed backup!"
    exit 1
else
    echo "✅ All backups successful"
    exit 0
fi
```

## Restore Procedures

To restore a configuration to a device:

### IOS/IOS-XE Devices

```yaml
---
- name: Restore IOS Configuration
  hosts: router1
  gather_facts: no
  tasks:
    - name: Copy config to device
      cisco.ios.ios_config:
        src: backups/20251101_143022/router1_20251101_143022.cfg
        replace: config
```

### NX-OS Devices

```yaml
---
- name: Restore NX-OS Configuration
  hosts: nexus1
  gather_facts: no
  tasks:
    - name: Copy config to device
      cisco.nxos.nxos_config:
        src: backups/20251101_143022/nexus1_20251101_143022.cfg
        replace: config
```

**⚠️ Warning:** Always test restore procedures in a lab environment first!

## Integration Examples

### Git Version Control

```bash
#!/bin/bash
cd backups
git add .
git commit -m "Backup $(date +%Y-%m-%d)"
git push origin main
```

### Email Notifications

Add to playbook:
```yaml
- name: Send email on failure
  mail:
    host: smtp.example.com
    port: 587
    username: alerts@example.com
    password: "{{ email_password }}"
    to: netops@example.com
    subject: "Cisco Backup Failed - {{ inventory_hostname }}"
    body: "Backup failed for {{ inventory_hostname }} at {{ ansible_date_time.iso8601 }}"
  when: ansible_failed_task is defined
  delegate_to: localhost
```

### Slack Notifications

```yaml
- name: Send Slack notification
  uri:
    url: https://hooks.slack.com/services/YOUR/WEBHOOK/URL
    method: POST
    body_format: json
    body:
      text: "✅ Cisco backup completed successfully at {{ timestamp }}"
  delegate_to: localhost
  run_once: true
```

## Performance Tuning

### Parallel Execution

Increase forks for faster execution:
```ini
[defaults]
forks = 10
```

### Connection Pooling

Enable persistent connections:
```ini
[persistent_connection]
connect_timeout = 30
command_timeout = 30
```

### Reduce Verbosity

For production runs, use minimal output:
```bash
ansible-playbook cisco_backup.yaml > /dev/null 2>&1
```

## Maintenance

### Regular Tasks

- **Weekly:** Review backup summary logs for failures
- **Monthly:** Verify backups are restorable (test restore)
- **Quarterly:** Update Ansible and collections
- **Annually:** Review and update credentials

### Health Check Script

```bash
#!/bin/bash
# Check last backup age
LAST_BACKUP=$(ls -td backups/*/ | head -1)
BACKUP_AGE=$((($(date +%s) - $(date -r "$LAST_BACKUP" +%s)) / 86400))

if [ $BACKUP_AGE -gt 1 ]; then
    echo "⚠️  Last backup is $BACKUP_AGE days old!"
else
    echo "✅ Backups are current"
fi
```

## Support and Contribution

### Getting Help

- Check the troubleshooting section above
- Review Ansible documentation: https://docs.ansible.com
- Cisco collection docs: https://docs.ansible.com/ansible/latest/collections/cisco/

### Reporting Issues

When reporting issues, include:
- Ansible version (`ansible --version`)
- Playbook output with `-vvv` flag
- Inventory configuration (sanitized)
- Device platform and version

## License

This playbook is provided as-is for network automation purposes.

## Changelog

### Version 1.0 (2025-11-01)
- Initial release
- Support for IOS, IOS-XE, NX-OS
- Automated retention management
- Thread-safe logging
- Comprehensive error handling

---

**Author:** Ehsan Momeni Bashusqeh, Network Automation Engineer  
**Last Updated:** November 2025  
**Tested Platforms:** IOS 15.x, IOS-XE 16.x/17.x, NX-OS 9.x