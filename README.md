# TroubleShooting
After a fresh installation of Nutanix Community Edition (CE), the cluster entered a "service lockout" state. Despite using default factory credentials, the Prism Web Console remained inaccessible due to a synchronization gap between the backend security services and the frontend gateway. This project documents the systematic troubleshooting and resolution of this critical infrastructure block.

## Resolution Path: Manual Cluster Initialization & Lockout Recovery
After verifying that the Genesis and Foundation services were running but the cluster was unconfigured, I manually bootstrapped the environment and cleared the persistent security lockouts.

### 1. Manual Cluster Creation
Since the automated installation did not initialize the cluster, I manually formed the single node cluster to establish the storage metadata layer:

```bash
#Initializing the cluster with the source IP and redundancy factor
cluster -s 192.168.1.171 --redundancy_factor=1 create
```

### 2. Identifying the Service Lockout
After verifying the cluster was healthy via cluster status command (confirming all services had active PIDs), the Prism Element Web Console remained inaccessible, returning an "Account Locked" error. This was identified as a PAM (Pluggable Authentication Module) lockout on the Controller VM (CVM).

### 3. Credential Synchronization & Service Flush
I executed the following sequence to reset the security tally, sync the admin password, and restart the web gateway:

Reset login tally across the cluster:

```bash
#Reset login tally
allssh "sudo faillock --user admin --reset"

#Sync password to NCLI / Force update the administrative password in NCLI:
ncli user reset-password user-name=admin password=<REDACTED>
```

Final security clearance:
```bash
#Reset login tally
allssh "sudo faillock --user admin --reset"

#Restart Prism service to apply changes:
genesis stop prism && cluster start
```
## Technical Takeaways
Bootstrapping: Demonstrated the ability to manually initialize a Nutanix cluster via CLI when the automated discovery process fails.

Security Administration: Resolved Linux-level PAM lockouts (faillock) affecting application-level access.

Service Management: Managed distributed services across a Nutanix Controller VM using genesis and allssh utilities.
