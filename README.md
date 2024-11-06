## QumuloSync

Sync configuration settings (NFS Exports, SMB Shares, S3 Buckets, etc) between Qumulo clusters

## Overview

Qumulo's software is enhanced, complemented, and managed by a robust, RESTful API. The browser-based application is driven 100% by our API. Because of this user-driven design, end-users can also leverage the power and flexibility of this rich API.  

All bindings and applications are built on top of the REST standardized endpoint on each Qumulo cluster's nodes. Qumulo's engineering team provides Python library bindings for accessing the API.  

We have prepared automation to run cluster sync operations for the configurations for SMB, NFS, S3 and directory Quota definitions which will be required for the secondary cluster.

## Definitions

*   **Source cluster**: The cluster that has the replication relationships source directories.
*   **Destination cluster**: The cluster that has the replication relationships target directories. Target directories are read-only during the replication relationships are connected.
*   **Top dir**: If you replicate your directories under a specific directory on the destination cluster, you can define that directory here.
*   **Skip replication**: The sync script has a control mechanism to sync only the definitions, which is part of an existing replication between the source and destination clusters. However, if you want to synchronize your definitions without this control, you can skip this control by setting this definition true.

## Installation

### Requirements:

*   Python 3.10
*   Qumulo 6.0.1+ (without Snapshot Locking)
  
### Install Path:

**Windows:**

C:\\Qumulo\\QumuloSync  

**Linux/MacOs:**

/opt/Qumulo/QumuloSync

### Installation Steps:

Before you can configure and get this report to work on your system, you will need to clone this repository on your
machine. For that, you will need to have `git` installed on your machine.

Once git is operational, then find or create C:\\Qumulo or /opt/Qumulo directory  of the `QumuloSync` repository and
clone it to your machine with the command `git clone https://github.com/Qumulo/QumuloSync.git`

You will notice that the `git clone` command will create a new directory in your current location call `QumuloSync`.

The contents of that directory should look like:

```
-rw-r--r--  1 someone  somegroup     1063 Mar 17 08:24 LICENSE
-rw-r--r--  1 someone  somegroup    12530 Mar 30 09:23 README.md
drwxr-xr-x  4 someone  somegroup      128 Mar 30 09:17 config
-rw-r--r--  1 someone  somegroup  7226342 Mar 22 08:33 QumuloSync.exe
-rw-r--r--  1 someone  somegroup  6145736 Mar 22 08:33 QumuloSync.ubuntu
-rw-r--r--  1 someone  somegroup  6145736 Mar 22 08:33 QumuloSync.ubuntu2004
-rw-r--r--  1 someone  somegroup  5394624 Mar 22 08:33 QumuloSync.macos

```

Of course, the owner will not be `someone` and the group will not be `somegroup`, but will show you as the owner and the group as whatever group you currently belong to. If in doubt, simply type `id -gn` to see your current group and `id -un` to see your current login id.

Linux and MacOS QumuloSync files execute permissions before running them. Run the below command for this purpose.

```
chmod a+x ./QumuloSync.ubuntu
```
or
```
chmod a+x ./QumuloSync.macos
```

You will need to modify the configuration file and one shell script to get this report to run in your environment. Let us
start with the configuration file.

## Configuration

If you want to run QumuloSync by using a configuration file, please follow the below details.

In the directory `config`, there is a file called `config.json` that must be modified
in order to run the report.

This application depends upon the file name `config.json` in the subdirectory `config`. Due to that restriction,
this file and the diretory it is contained in should not be moved or renamed.


**Example:**

```plaintext
{
    "force": true,
    "log": "DEBUG",
    "top_dir": "",
    "skip_replication": true,
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
    "snapshot_policy": {
		"policies": ["all"]

	},
	"quota": {
			"directories": ["all"],
			"additional_limit":0
	},
    "source": {
        "address": "10.220.150.10",
        "port": "8000",
        "username": "berat",
        "password": "PASSWORD",
        "access_token": ""
    },
    "destination": {
        "address": "10.220.151.90",
        "port": "8000",
        "username": "",
        "password": "",
        "access_token": "ACCESS_TOKEN"
    }
}
```

*   **force:** You can skip the confirmation step by setting this parameter to **true**. It is equivalent of the **\--force** parameter.
*   **top\_dir:** If you replicate your directories under a specific directory on the destination cluster, you can define that directory here.
*   **skip\_replication:** The sync script has a control mechanism to sync only the definitions, which is a part of an existing replication between the source and destination clusters. However, if you want to synchronize your definitions without this control, you can skip this control by setting this definition as **true**.
*   **shares, export, buckets and directories:** You can define specific definitions that you want to run any operations. If you want to run operations for all definitions, write **“all”**.

**Examples:**  
`"directories" :["all"]`  
or  
`"directories" :["/berat",”/ulualan”]`

*   **source and destination:** You need to set the cluster details here.
*   **address:** the cluster’s IP address or DNS name for the connection
*   **port:** the API port number of the cluster
*   You can use either the **username** & **password** or the **access\_token** of that user for authentication. The user must have the privileges below.

```plaintext
• ANALYTICS_READ
• NFS_EXPORT_READ
• NFS_EXPORT_WRITE
• NFS_SETTINGS_READ
• NFS_SETTINGS_WRITE
• QUOTA_READ
• QUOTA_WRITE
• SMB_SHARE_READ
• SMB_SHARE_WRITE
• SNAPSHOT_POLICY_READ
• SNAPSHOT_POLICY_WRITE
• S3_BUCKETS_READ
• S3_BUCKETS_WRITE
```

How to Run The QumuloSync Script

The config file and CLI parameters are fully matched. You can run your QumuloSync script without a config file using the required parameters from CLI.  

You can run the sync script with the below parameters:  

`-f`, `--force` Sync existing settings without confirmation.  
`-t _TOP_DIR_`, `--top-dir _TOP_DIR_` The top directory path for capacity check  
`-c _CONFIG_FILE_`, `--config-file _CONFIG_FILE_` The configuration file, which has the definitions of how to run this script  
`--skip-replication` Skip the replication relationship verification steps

**sub-commands:**  
{smb,nfs,quota,source,destination}  
`smb` Sync SMB shares  
`nfs` Sync NFS exports  
`s3` Sync S3 buckets
`quota` Sync directory quotas  
`snapshot_policy` Sync snapshot policies
`source` Source Qumulo cluster  
`destination` Destination Qumulo cluster  

**Source & Destination Definitions**  
`--address _ADDRESS_` Source or Destination Qumulo cluster IP address or hostname  
`--port _PORT_` Source or Destination Qumulo cluster API port number  
`--username _USERNAME_` Source or Destination Qumulo cluster username  
`--password _PASSWORD_` Source or Destination Qumulo cluster password  
`--access-token _ACCESS_TOKEN_` Source or Destination Qumulo cluster access token  

**SMB Parameters**  
`-s _SHARES [SHARES ...]_`, `--shares _SHARES [SHARES ...]_` Sync SMB shares  
`--hide` Hide SMB shares  

**NFS Parameters**  
`-n _EXPORTS [EXPORTS ...]_`, `--exports _EXPORTS [EXPORTS ...]_` Sync NFS exports  

**S3 Parameters**  
`-b _BUCKETS [BUCKETS ...]_`, `--buckets _BUCKETS [BUCKETS ...]_` Sync S3 exports  

**Quota Parameters**  
`-d _DIRECTORIES [DIRECTORIES ...]_`, `--directories _DIRECTORIES [DIRECTORIES ...]_` Sync directory quotas  
`--additional-limit _LIMIT_` Additional capacity limit for quota in percentage

**Snapshot Policy Parameters**  
`-p _POLICIES [POLICIES ...]_`, `--policies _POLICIES [POLICIES ...]_` Additional capacity limit for quota in percentage

### Examples:

**Using the config.json File**

```plaintext
.\QumuloSync.exe --config-file .\config\config.json
```

**SMB**

Run for all shares

```plaintext
.\QumuloSync.exe --force --skip-replication source --address 10.220.200.6 --port 8000 --username berat --password PASSWORD destination --address 10.220.241.52 --port 8000 --access-token ACCESS_TOKEN smb --shares all
```

Run for specific shares

```plaintext
./QumuloSync.macos --force --skip-replication source --address 10.220.200.6 --port 8000 --username berat --password PASSWORD destination --address 10.220.241.52 --port 8000 --access-token ACCESS_TOKEN smb --shares berat ulualan
```

**NFS**

Run for all exports

```plaintext
 ./QumuloSync.ubuntu --force --skip-replication source --address 10.220.200.6 --port 8000 --username berat --password PASSWORD destination --address 10.220.241.52 --port 8000 --access-token ACCESS_TOKEN nfs --exports all
```

Run for specific exports

```plaintext
.\QumuloSync.exe --force --skip-replication source --address 10.220.200.6 --port 8000 --username berat --password PASSWORD destination --address 10.220.241.52 --port 8000 --access-token ACCESS_TOKEN nfs --exports /berat /ulualan
```

**S3**

Run for all buckets

```plaintext
.\QumuloSync.exe --force --skip-replication source --address 10.220.200.6 --port 8000 --username berat --password PASSWORD destination --address 10.220.241.52 --port 8000 --access-token ACCESS_TOKEN s3 --buckets all
```

Run for specific buckets

```plaintext
./QumuloSync.macos --force --skip-replication source --address 10.220.200.6 --port 8000 --username berat --password PASSWORD destination --address 10.220.241.52 --port 8000 --access-token ACCESS_TOKEN smb --buckets berat ulualan
```

**QUOTA**

Run for all directory quotas

```plaintext
 ./QumuloSync.ubuntu2004 --force --skip-replication source --address 10.220.200.6 --port 8000 --username berat --password Admin123 destination --address 10.220.241.52 --port 8000 --access-token ACCESS_TOKEN quota --directories all --additional-limit 5
```

Run for specific directory quotas

```plaintext
.\QumuloSync.exe --force --skip-replication source --address 10.220.200.6 --port 8000 --username berat --password PASSWORD destination --address 10.220.241.52 --port 8000 --access-token ACCESS_TOKEN quota --directories /berat /ulualan --additional-limit 0
```

**SNAPSHOT POLICY**

Run for all snapshot policies

```plaintext
 ./QumuloSync.ubuntu2004 --force --skip-replication source --address 10.220.200.6 --port 8000 --username berat --password Admin123 destination --address 10.220.241.52 --port 8000 --access-token ACCESS_TOKEN snapshot_policy --policies all
```

Run for specific snapshot policies

```plaintext
.\QumuloSync.exe --force --skip-replication source --address 10.220.200.6 --port 8000 --username berat --password PASSWORD destination --address 10.220.241.52 --port 8000 --access-token ACCESS_TOKEN snapshot_policy --policies berat_hourly berat_daily 
```

## Additional Documentation

For help planning the deployment see the table of documents below.

|Documentation|Description|
|-------------|-----------|
|[Using Qumulo Core Access Tokens](https://docs.qumulo.com/administrator-guide/external-services/using-access-tokens.html) | Details on Qumulo Access Token|
|[Role-Based Access Control (RBAC) with Qumulo Core](https://care.qumulo.com/hc/en-us/articles/360036591633-Role-Based-Access-Control-RBAC-with-Qumulo-Core) | Details on Qumulo Role-Based Access Control|

## Support Statement

This tool provides Qumulo customers more flexibility using Qumulo API's capabilities. Please use GitHub Issues or your Qumulo Care Slack channel for feature requests and software deficiencies. We will address these as soon as possible, but there are no specific SLAs on response times for inquiries regarding functionality, performance, or perceived software deficiencies.

## Help

Use the [Issues](https://github.com/Qumulo/QumuloSync/issues) section of this GitHub repo to post feedback, submit feature ideas, or report bugs.

## Copyright

Copyright © 2022 [Qumulo, Inc.](https://qumulo.com)

## License

[![License](https://img.shields.io/badge/license-MIT-green)](https://opensource.org/licenses/MIT)

See [LICENSE](LICENSE) for full details

```plaintext
MIT License

Copyright (c) 2022 Qumulo, Inc.

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

## Trademarks

All other trademarks referenced herein are the property of their respective owners.

## Contributors

*   [Berat Ulualan](https://github.com/beratulualan)
*   [Michael Kade](https://github.com/mikekade)
