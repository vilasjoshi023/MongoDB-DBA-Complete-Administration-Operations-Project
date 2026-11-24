# MongoDB Patching and Upgrade Plan – Three-Node Replica Set

**Prepared:** 24 November 2026  
**Current lab version:** MongoDB Community Server 7.0.48  
**Shell:** mongosh 2.14.3  
**Operating system:** CentOS Stream 10  
**Replica set:** rs0  
**Members:** VILASLAB1 (192.168.62.128), VILASLAB2 (192.168.62.131), VILASLAB3 (192.168.62.132)

> **Status:** Planning and runbook only. No patch or major-version upgrade has been performed.

## 1. Current-state assessment

The lab currently has three healthy voting members. VM1 has priority 2; VM2 and VM3 have priority 1. The documented failover drill showed VM1 stepping down, VM2 becoming PRIMARY, and VM1 later regaining PRIMARY after the 60-second step-down exclusion ended.

The user-confirmed replica-set tests include:
- All three members healthy, with VM1 PRIMARY and VM2/VM3 SECONDARY.
- A majority-acknowledged test insert on the primary.
- The inserted document read successfully from both secondaries.
- Secondary replication lag reported as 0 seconds at the time of the check.
- Successful controlled `rs.stepDown(60)` and election of VM2.
- VM1 later regained PRIMARY, consistent with its higher priority.

### Release position as at 24 November 2026

- MongoDB 7.0.48 was announced on 11 November 2026 and is the latest publicly announced self-managed 7.0 patch release verified for this planning document.
- The MongoDB 7.0 release notes list 7.0.49 dated 18 November 2026 as **Atlas Only**, not a self-managed package release.
- Therefore, do not try to install 7.0.49 on this self-managed lab. Stay on 7.0.48 unless MongoDB publishes a subsequent self-managed 7.0 patch.
- For a future major upgrade, use the MongoDB 8.0 release series as the next step from 7.0. MongoDB 8.0.38 was announced on 11 November 2026; re-check the current patch at the time of the change.

Sources:
- MongoDB 7.0 release notes: https://www.mongodb.com/docs/v7.0/release-notes/7.0/
- MongoDB 7.0.48 release announcement: https://www.mongodb.com/community/forums/t/mongodb-7-0-48-is-released/344844
- MongoDB server release announcements: https://www.mongodb.com/community/forums/c/announcements/server-releases/109

## 2. Maintenance principles

1. Use a maintenance window, approved change record and documented rollback plan.
2. Verify recoverable backups before touching packages.
3. Upgrade one replica-set member at a time; never stop both secondaries at once.
4. Keep a voting majority available. In this three-voter set, at least two voting members must remain available for majority decisions.
5. Allow each restarted member to rejoin and reach SECONDARY/healthy state before moving to the next member.
6. Upgrade the current PRIMARY last. Step it down only after the upgraded secondaries are healthy.
7. Do not change FCV for a patch update within the same major release.
8. For major upgrades, keep the old FCV during the binary rollout and only advance FCV after a successful burn-in and approval.
9. Never assume that a package downgrade is a safe rollback. Validate the supported downgrade path and preserve tested backups.

## 3. Track A – Patch update within MongoDB 7.0.x

### A1. Pre-change inventory (run on all three VMs)

```bash
hostname
cat /etc/os-release
mongod --version | head -n 1
mongosh --version
rpm -qa | grep -E '^(mongodb|mongosh)' | sort
sudo dnf repolist
```

From mongosh, capture:

```javascript
db.hello()
rs.status()
rs.conf()
db.adminCommand({ getParameter: 1, featureCompatibilityVersion: 1 })
```

Record the current package versions, FCV, member roles, uptime, replication lag, available disk space, and service state.

### A2. Check available packages before deciding to patch

On each node, inspect the enabled MongoDB repository and available package versions:

```bash
sudo dnf makecache --refresh
sudo dnf --showduplicates list mongodb-org-server
sudo dnf repoquery --installed 'mongodb*' | sort
```

Preview a proposed package transaction without applying it:

```bash
sudo dnf upgrade --assumeno mongodb-org
```

Confirm that the proposed version is the intended self-managed release and that all MongoDB component packages are aligned. Do not select an Atlas-only build.

### A3. Back up and establish a recovery point

Before the maintenance window:
- Take a supported backup for the deployment and protect it from accidental deletion.
- For a logical replica-set backup, assess `mongodump --oplog` and the matching `mongorestore --oplogReplay` workflow for the installed Database Tools version.
- For production, select a backup mechanism appropriate to data size, RPO/RTO, encryption, and recovery architecture.
- Perform a restore test into an isolated location. A backup is not considered proven until restoration is validated.
- Preserve sanitized `/etc/mongod.conf`, repository definitions, package inventory, service settings and relevant OS configuration.
- Confirm adequate free space for backups, package downloads, logs and rollback artifacts.

### A4. Rolling patch order

Use this order if VM1 is PRIMARY when the change begins:

**1. Patch VM2 (SECONDARY)**
- Reconfirm that VM1 is PRIMARY and VM2/VM3 are healthy.
- Stop only VM2's mongod service.
- Apply the approved 7.0.x package update using the tested DNF transaction.
- Start mongod and verify service health and exact version.
- Wait until VM2 rejoins as SECONDARY with health 1 and catches up.

**2. Patch VM3 (SECONDARY)**
- Repeat the same process for VM3 only after VM2 is healthy and synchronized.
- Wait until VM3 is SECONDARY with health 1 and replication lag is acceptable/settled.

**3. Patch VM1 (current PRIMARY) last**
- Recheck that both secondaries are healthy and synchronized.
- Connect to the current PRIMARY and perform a planned step-down.
- Confirm another member has become PRIMARY.
- Stop VM1's mongod service, apply the approved package update and restart it.
- Wait for VM1 to rejoin as SECONDARY and synchronize.

For every node, use the OS shell for service/package operations:

```bash
sudo systemctl stop mongod
# Apply the approved and reviewed package transaction here.
sudo systemctl start mongod
sudo systemctl status mongod --no-pager
mongod --version | head -n 1
```

Do not run the stop/update/start sequence against multiple members in parallel. Do not run the package transaction until the selected patch build has been confirmed from the repository and the backup/restore gate is passed.

### A5. Patch post-validation

On the replica set:

```javascript
rs.status().members.forEach(m => print(m.name + " | " + m.stateStr + " | health=" + m.health))
rs.printSecondaryReplicationInfo()
rs.conf()
db.hello()
```

On each VM:

```bash
systemctl is-active mongod
mongod --version | head -n 1
rpm -qa | grep -E '^(mongodb|mongosh)' | sort
```

Acceptance criteria:
- All three nodes run the intended 7.0.x server patch.
- One PRIMARY and two SECONDARY members.
- All members health 1.
- No member stuck in STARTUP, STARTUP2, RECOVERING, ROLLBACK or UNKNOWN.
- Replication catches up; no unexpected lag trend.
- Test write with majority write concern succeeds.
- Test document can be read through the replica-set connection string.
- Logs show no new startup, storage, replication or authentication failures.
- Monitoring and application checks are green.

## 4. Track B – Major upgrade from MongoDB 7.0 to 8.0

This is a separate change with stronger compatibility, backup and rollback gates. Do not combine it with an ordinary patch window.

### B1. Pre-upgrade gates

- Upgrade path: all replica-set members must be running the 7.0 series before moving to 8.0.
- Review MongoDB 8.0 compatibility changes and application behavior.
- Verify each application driver version supports MongoDB 8.0.
- Validate the exact OS/platform support and repository URL for CentOS Stream 10 at execution time. Do not assume a repository path is supported merely because a package installs.
- Verify free disk, RAM, CPU, filesystem and backup capacity for the target binaries and workload.
- Complete a full backup and a successful isolated restore test.
- Verify all members are healthy and no member is in ROLLBACK or RECOVERING.
- Verify FCV is 7.0 on all members.
- Check deployment-specific prerequisites, including any special collection types or features called out in the exact 8.0 patch's upgrade guide.
- Test the upgrade in a disposable/staging environment before production.

Check FCV on each member:

```javascript
db.adminCommand({ getParameter: 1, featureCompatibilityVersion: 1 })
```

Expected pre-upgrade version field:

```javascript
featureCompatibilityVersion: { version: "7.0" }
```

### B2. Rolling 7.0-to-8.0 sequence

Follow the official MongoDB 7.0-to-8.0 replica-set upgrade guide for the exact target release. The documented high-level sequence is:

1. Shut down one SECONDARY member.
2. Replace its 7.0 server binary/packages with the approved 8.0 release and restart.
3. Wait for it to rejoin and become healthy; repeat for the other SECONDARY.
4. Step down the PRIMARY and confirm another member has assumed PRIMARY.
5. Upgrade the former PRIMARY last and restart it.
6. Verify all members run 8.0 and the set is healthy.
7. Run with the previous FCV during a defined burn-in/validation period.
8. Only after approval, advance FCV to 8.0.

Do not run the FCV change during an initial sync. Do not advance FCV until the rollback decision window is closed.

Official procedure: https://www.mongodb.com/docs/v8.0/release-notes/8.0-upgrade-replica-set/

### B3. FCV advancement – approval gate

Once every member is running the target 8.0 binaries, the deployment has passed the burn-in, and the change owner has approved the irreversible compatibility step, run on the PRIMARY:

```javascript
db.adminCommand({ setFeatureCompatibilityVersion: "8.0", confirm: true })
```

This is a separate logical change from installing binaries. Raising FCV enables new-version behavior and can complicate or prevent downgrade. Follow the target release's downgrade documentation and support policy; never treat an FCV raise as a routine reversible setting.

### B4. Major-upgrade post-validation

- All three nodes report the target 8.0 patch version.
- Exactly one PRIMARY and two SECONDARY members.
- All members health 1 and synchronized.
- FCV is at the intended value after explicit approval.
- Application/driver connectivity and CRUD/transaction tests succeed.
- Backup and restore procedures are still valid.
- Monitoring, logs, performance and resource utilization meet baseline.
- Change record contains start/end, versions, FCV, test results, issues and sign-off.

## 5. Rollback and recovery plan

### Patch release rollback
- Stop further rollout if a member fails to start or rejoin.
- Preserve package transaction output, mongod logs and configuration.
- Do not take down another member while a node is unhealthy.
- Use only the documented downgrade path for the exact package/build and data state.
- Restore from the validated backup if downgrade is unsupported or unsafe.

### Major version rollback
- Before FCV is advanced, stop and assess if the binary rollback procedure is supported by MongoDB for the exact target versions and data state.
- After FCV is advanced, do not assume a simple package downgrade is possible. Follow MongoDB's documented downgrade constraints or restore/migrate using an approved recovery plan.
- Keep the backup independent of the database nodes and validate restoration before the change.

## 6. Lab execution sequence

| Phase | Activity | Status |
|---|---|---|
| 1 | Capture current versions, package inventory, FCV and baseline health | Planned |
| 2 | Verify backup tools and perform backup/restore drill | Planned |
| 3 | Check for a newer self-managed 7.0 patch | Current 7.0.48 verified; no 7.0.49 self-managed update selected |
| 4 | Practice rolling 7.0 patch update when a newer public 7.0 package is available | Planned |
| 5 | Validate CentOS Stream 10 support/repository for 8.0 | Planned |
| 6 | Stage-test 7.0-to-8.0 rolling upgrade | Planned |
| 7 | Production-style FCV approval and post-upgrade validation | Planned |

## 7. DBA change checklist

- [ ] Approved maintenance window and change record
- [ ] Stakeholder/application owner notification
- [ ] Current versions and FCV captured on all nodes
- [ ] Replica-set health and lag baseline captured
- [ ] Backup completed and restore test passed
- [ ] Target package/repository/vendor platform support verified
- [ ] Compatibility and driver checks passed
- [ ] Free disk and resource capacity checked
- [ ] Configurations and repository files backed up
- [ ] One-member-at-a-time rollout plan reviewed
- [ ] Post-change validation and monitoring owner assigned
- [ ] Rollback/recovery procedure reviewed
- [ ] Final change evidence and sign-off recorded

## 8. Official references

- MongoDB 7.0 release notes: https://www.mongodb.com/docs/v7.0/release-notes/7.0/
- MongoDB 7.0.48 release announcement: https://www.mongodb.com/community/forums/t/mongodb-7-0-48-is-released/344844
- Upgrade replica set from 7.0 to 8.0: https://www.mongodb.com/docs/v8.0/release-notes/8.0-upgrade-replica-set/
- MongoDB versioning and FCV: https://www.mongodb.com/docs/manual/reference/command/setFeatureCompatibilityVersion/
- MongoDB 8.0 platform installation guidance: https://www.mongodb.com/docs/v8.0/tutorial/install-mongodb-on-red-hat/
