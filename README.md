# QumuloSync

**Synchronize configuration between Qumulo clusters**

QumuloSync copies SMB shares, NFS exports, S3 buckets, directory quotas, and snapshot policies from a source cluster to a destination cluster.

---

## Quick Start

### Sync Everything Using a Config File

```bash
./QumuloSync.ubuntu --config-file ./config/config.json
```

### Sync Specific Resources via Command Line

```bash
# Sync all SMB shares
./QumuloSync.ubuntu --force --skip-replication \
  source --address primary.company.com --port 8000 --username admin --password SECRET \
  destination --address dr-site.company.com --port 8000 --username admin --password SECRET \
  smb --shares all

# Sync specific NFS exports
./QumuloSync.ubuntu --force --skip-replication \
  source --address primary.company.com --port 8000 --username admin --password SECRET \
  destination --address dr-site.company.com --port 8000 --username admin --password SECRET \
  nfs --exports /home /projects /shared
```

---

## What Gets Synced

| Resource Type | Description | Config Key |
|---------------|-------------|------------|
| **SMB Shares** | Windows file shares with permissions | `smb.shares` |
| **NFS Exports** | Unix/Linux exports with restrictions | `nfs.exports` |
| **S3 Buckets** | Object storage buckets | `s3.buckets` |
| **Directory Quotas** | Storage limits per directory | `quota.directories` |
| **Snapshot Policies** | Automated snapshot schedules | `snapshot_policy.policies` |

---

## Configuration File

Create a `config.json` file:

```json
{
    "force": true,
    "skip_replication": true,
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

### Configuration Options

| Option | Description | Default |
|--------|-------------|---------|
| `force` | Skip confirmation prompts | `false` |
| `skip_replication` | Sync all resources, not just replicated ones | `false` |
| `top_dir` | Prefix path on destination (e.g., `/dr`) | `""` |
| `delete_non_existing` | Remove destination-only resources | `false` |
| `additional_limit` | Increase quota limits by percentage | `0` |

---

## Usage Examples

### Example 1: Full DR Site Sync

Sync all configurations from production to DR:

```bash
./QumuloSync.ubuntu -c config/dr_sync.json --force
```

### Example 2: Sync Only SMB Shares

```bash
./QumuloSync.ubuntu --force --skip-replication \
  source --address prod.company.com --port 8000 --username admin --password SECRET \
  destination --address dr.company.com --port 8000 --username admin --password SECRET \
  smb --shares all
```

### Example 3: Sync Specific Directories with Hidden Shares

```bash
./QumuloSync.ubuntu --force --skip-replication \
  source --address prod.company.com --port 8000 --username admin --password SECRET \
  destination --address dr.company.com --port 8000 --username admin --password SECRET \
  smb --shares Finance$ HR$ IT$ --hide
```

### Example 4: Sync Quotas with 10% Extra Capacity

```bash
./QumuloSync.ubuntu --force --skip-replication \
  source --address prod.company.com --port 8000 --username admin --password SECRET \
  destination --address dr.company.com --port 8000 --username admin --password SECRET \
  quota --directories all --additional-limit 10
```

### Example 5: Sync NFS Exports to Subdirectory

```bash
./QumuloSync.ubuntu --force --skip-replication --top-dir /replicated \
  source --address prod.company.com --port 8000 --username admin --password SECRET \
  destination --address dr.company.com --port 8000 --username admin --password SECRET \
  nfs --exports all
```

### Example 6: Using Access Tokens Instead of Passwords

```bash
./QumuloSync.ubuntu --force --skip-replication \
  source --address prod.company.com --port 8000 --access-token "qqq_abc123..." \
  destination --address dr.company.com --port 8000 --access-token "qqq_xyz789..." \
  smb --shares all
```

### Example 7: Delete Resources Not on Source

⚠️ **Use with caution** - this removes resources from destination that don't exist on source:

```json
{
    "force": true,
    "delete_non_existing": true,
    "smb": { "shares": ["all"] },
    ...
}
```

---

## Command Line Reference

```
QumuloSync [OPTIONS] source [SOURCE_OPTIONS] destination [DEST_OPTIONS] RESOURCE [RESOURCE_OPTIONS]

Global Options:
  -f, --force              Skip confirmation prompts
  -c, --config-file FILE   Use configuration file
  -t, --top-dir DIR        Prefix path on destination
  --skip-replication       Sync all resources (not just replicated)
  --log LEVEL              Log level: DEBUG, INFO, WARNING, ERROR

Source/Destination Options:
  --address HOST           Cluster hostname or IP
  --port PORT              API port (default: 8000)
  --username USER          Admin username
  --password PASS          Admin password
  --access-token TOKEN     Access token (alternative to user/pass)

Resource Commands:
  smb                      Sync SMB shares
    --shares NAME [NAME...]    Share names or "all"
    --hide                     Add $ suffix to hide shares
    
  nfs                      Sync NFS exports
    --exports PATH [PATH...]   Export paths or "all"
    
  s3                       Sync S3 buckets
    --buckets NAME [NAME...]   Bucket names or "all"
    
  quota                    Sync directory quotas
    --directories PATH [PATH...] Directory paths or "all"
    --additional-limit N       Increase limits by N percent
    
  snapshot_policy          Sync snapshot policies
    --policies NAME [NAME...]  Policy names or "all"
```

---

## Required Permissions

The user account needs these privileges on both clusters:

```
SMB_SHARE_READ, SMB_SHARE_WRITE
NFS_EXPORT_READ, NFS_EXPORT_WRITE
S3_BUCKETS_READ, S3_BUCKETS_WRITE
QUOTA_READ, QUOTA_WRITE
SNAPSHOT_POLICY_READ, SNAPSHOT_POLICY_WRITE
ANALYTICS_READ
```

---

## Troubleshooting

### Connection Errors
- Verify network connectivity to both clusters
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
qumulosync_20241127_143500.log
```

---

## Support

- **Issues**: [GitHub Issues](https://github.com/Qumulo/QumuloSync/issues)
- **Slack**: Qumulo Care Slack channel
- **Docs**: [Qumulo Documentation](https://docs.qumulo.com)

---

## License

MIT License - Copyright (c) 2022 Qumulo, Inc.
