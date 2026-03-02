# QumuloSync

**Synchronize and backup configuration for Qumulo clusters**

QumuloSync provides three main commands:
- **sync**: Copy configuration from a source cluster to a destination cluster
- **save**: Backup cluster configuration to a JSON file
- **restore**: Restore cluster configuration from a backup file

---

## Requirements

- **Python** 3.10 or later
- **Qumulo Core** 7.4.4 or later (cluster firmware version)
- **qumulo_api** 7.4.4 or later (`pip install qumulo_api`)

The tool uses Qumulo REST API features (multitenancy, S3 bucket policies, strongly-typed dataclasses) that require Qumulo Core 7.4.4+. Older clusters are not supported.

---

## Quick Start

### Sync Between Clusters

```bash
# Sync all resources from source to destination
./QumuloSync.ubuntu sync \
  --source-address prod.company.com --source-username admin --source-password SECRET \
  --dest-address dr.company.com --dest-username admin --dest-password SECRET

# Sync only SMB shares
./QumuloSync.ubuntu sync \
  --source-address prod.company.com --source-username admin --source-password SECRET \
  --dest-address dr.company.com --dest-username admin --dest-password SECRET \
  --resources smb
```

### Save Configuration to File

```bash
# Save all configuration to JSON
./QumuloSync.ubuntu save \
  --address cluster.company.com --username admin --password SECRET \
  --output backup_20260127.json

# Save only SMB shares and quotas
./QumuloSync.ubuntu save \
  --address cluster.company.com --access-token "qqq_abc123..." \
  --output backup_smb_quotas.json --resources smb_shares quotas
```

### Restore Configuration from File

```bash
# Restore from JSON backup
./QumuloSync.ubuntu --force restore \
  --address new-cluster.company.com --username admin --password SECRET \
  --input backup_20260127.json

# Restore only NFS exports with path prefix
./QumuloSync.ubuntu --force restore \
  --address new-cluster.company.com --username admin --password SECRET \
  --input backup_20260127.json --resources nfs_exports --top-dir /migrated
```

---

## What Gets Synced

| Resource Type | Description | Sync | Save/Restore |
|---------------|-------------|------|--------------|
| **SMB Shares** | Windows file shares with permissions | ✓ | ✓ |
| **NFS Exports** | Unix/Linux exports with restrictions | ✓ | ✓ |
| **S3 Buckets** | Object storage buckets with policies | ✓ | ✓ |
| **Directory Quotas** | Storage limits per directory | ✓ | ✓ |
| **Snapshot Policies** | Automated snapshot schedules | ✓ | ✓ |
| **S3 Settings** | S3 server configuration | ✓ | ✓ |
| **Local Users** | Local user accounts | - | Save only |
| **Local Groups** | Local group accounts | - | Save only |

### Important Notes

- **Snapshot Policies**: Policies are restored with `enabled: false` to prevent them from immediately creating snapshots on the destination cluster. Enable them manually after verifying the configuration.
- **Local Users & Groups**: User and group accounts are saved for reference but are not created or modified during restore. During sync, any users or groups on the source cluster that are missing from the destination are logged as a warning.
- **S3 Buckets**: Bucket policies and versioning settings are preserved during save/restore.
- **S3 Settings**: S3 server settings (enabled, base_path) are synced before buckets to ensure S3 is enabled on the destination.

---

## Command Reference

### Global Options

```
-v, --version          Show version number
-l, --log LEVEL        Set log level: DEBUG, INFO, WARNING, ERROR (default: INFO)
-f, --force            Skip confirmation prompts and update existing resources
```

### sync Command

Sync configuration from source cluster to destination cluster.

```
QumuloSync sync [OPTIONS]

Source cluster:
  --source-address HOST      Source cluster hostname or IP (required)
  --source-port PORT         Source cluster port (default: 8000)
  --source-username USER     Source cluster username
  --source-password PASS     Source cluster password
  --source-token TOKEN       Source cluster access token

Destination cluster:
  --dest-address HOST        Destination cluster hostname or IP (required)
  --dest-port PORT           Destination cluster port (default: 8000)
  --dest-username USER       Destination cluster username
  --dest-password PASS       Destination cluster password
  --dest-token TOKEN         Destination cluster access token

Sync options:
  -r, --resources TYPE [...] Resource types to sync: all, smb, nfs, s3, s3_settings, quota, snapshot_policy
  -t, --top-dir DIR          Prefix path on destination cluster
  -c, --config-file FILE     Use configuration file
  -d, --delete-non-existing  Delete resources on destination not on source
  --hide                     Add $ suffix to SMB share names
  --additional-limit N       Increase quota limits by N percent
  -n, --dry-run              Show what would be synced without making changes
```

### save Command

Save cluster configuration to a file.

```
QumuloSync save [OPTIONS]

Required:
  --address HOST             Cluster hostname or IP
  -o, --output PATH          Output file (JSON)

Optional:
  --port PORT                Cluster port (default: 8000)
  --username USER            Cluster username
  --password PASS            Cluster password
  --access-token TOKEN       Cluster access token
  -r, --resources TYPE [...] Resource types to save:
                             local_users, local_groups, s3_settings,
                             smb_shares, nfs_exports, quotas,
                             s3_buckets, snapshot_policies, or all
```

### restore Command

Restore cluster configuration from a file.

```
QumuloSync [--force] restore [OPTIONS]

Required:
  --address HOST             Cluster hostname or IP
  -i, --input PATH           Input file (JSON)

Optional:
  --port PORT                Cluster port (default: 8000)
  --username USER            Cluster username
  --password PASS            Cluster password
  --access-token TOKEN       Cluster access token
  -r, --resources TYPE [...] Resource types to restore:
                             s3_settings, smb_shares, nfs_exports,
                             quotas, s3_buckets, snapshot_policies,
                             or all
  -t, --top-dir DIR          Prefix path for all restored resources
```

---

## Configuration File (for sync)

Create a `config.json` file for repeated sync operations:

```json
{
    "force": true,
    "top_dir": "",
    "delete_non_existing": false,
    
    "smb": {
        "shares": ["all"],
        "hide": false
    },
    "nfs": {
        "exports": ["all"]
    },
    "s3": {
        "buckets": ["all"]
    },
    "quota": {
        "directories": ["all"],
        "additional_limit": 0
    },
    "snapshot_policy": {
        "policies": ["all"]
    },
    
    "source": {
        "address": "source-cluster.company.com",
        "port": "8000",
        "username": "admin",
        "password": "your-password"
    },
    "destination": {
        "address": "dest-cluster.company.com",
        "port": "8000",
        "username": "admin",
        "password": "your-password"
    }
}
```

Then run:

```bash
./QumuloSync.ubuntu sync --config-file config.json
```

---

## Save/Restore Output Formats

### JSON Format

A single JSON file containing all configuration with metadata:

```json
{
  "version": "1.0",
  "cluster": "prod-cluster.company.com",
  "timestamp": "2026-01-27T15:30:00Z",
  "local_users": [...],
  "local_groups": [...],
  "s3_settings": {
    "enabled": true,
    "base_path": "/s3/",
    "multipart_upload_expiry_interval": "1days"
  },
  "smb_shares": [...],
  "nfs_exports": [...],
  "quotas": [...],
  "s3_buckets": [...],
  "snapshot_policies": [...]
}
```

---

## Examples

### Example 1: Full DR Site Sync

```bash
./QumuloSync.ubuntu --force sync \
  --source-address prod.company.com --source-username admin --source-password SECRET \
  --dest-address dr.company.com --dest-username admin --dest-password SECRET
```

### Example 2: Sync Only SMB Shares with Hidden Names

```bash
./QumuloSync.ubuntu --force sync \
  --source-address prod.company.com --source-username admin --source-password SECRET \
  --dest-address dr.company.com --dest-username admin --dest-password SECRET \
  --resources smb --hide
```

### Example 3: Sync to Subdirectory

```bash
./QumuloSync.ubuntu --force sync \
  --source-address prod.company.com --source-username admin --source-password SECRET \
  --dest-address dr.company.com --dest-username admin --dest-password SECRET \
  --top-dir /replicated
```

### Example 4: Save Full Backup

```bash
./QumuloSync.ubuntu save \
  --address cluster.company.com --username admin --password SECRET \
  --output full_backup_$(date +%Y%m%d).json
```

### Example 5: Restore with Path Prefix

```bash
./QumuloSync.ubuntu --force restore \
  --address new-cluster.company.com --username admin --password SECRET \
  --input backup.json --top-dir /restored
```

### Example 6: Using Access Tokens

```bash
./QumuloSync.ubuntu sync \
  --source-address prod.company.com --source-token "qqq_abc123..." \
  --dest-address dr.company.com --dest-token "qqq_xyz789..."
```

---

## Required Permissions

The user account needs these privileges on the cluster(s):

```
SMB_SHARE_READ, SMB_SHARE_WRITE
NFS_EXPORT_READ, NFS_EXPORT_WRITE
S3_BUCKETS_READ, S3_BUCKETS_WRITE
QUOTA_READ, QUOTA_WRITE
SNAPSHOT_POLICY_READ, SNAPSHOT_POLICY_WRITE
LOCAL_USER_READ                          # For save/sync user comparison
LOCAL_GROUP_READ                         # For save/sync group comparison
ANALYTICS_READ
```

---

## Troubleshooting

### Connection Errors
- Verify network connectivity to the cluster(s)
- Check that port 8000 is accessible
- Confirm credentials are correct

### Permission Errors
- Ensure the user has required privileges (see above)
- Check if the path exists on the destination cluster

### "Directory doesn't exist" Errors
- The source path must exist on the destination
- Use `--top-dir` if paths differ between clusters
- Ensure replication is complete before syncing

### Logs
Each run creates a timestamped log file:
```
qumulosync_20260127_143500.log
```

---

## Support

- **Issues**: [GitHub Issues](https://github.com/Qumulo/QumuloSync/issues)
- **Slack**: Qumulo Care Slack channel
- **Docs**: [Qumulo Documentation](https://docs.qumulo.com)

---

## License

MIT License - Copyright (c) 2022 Qumulo, Inc.
