# Daily Work Log – MongoDB Installation and Replica-Set Preparation

**Date:** 24 November 2026  
**Platform:** CentOS Stream 10 (Coughlan), x86_64  
**MongoDB Server:** 7.0.48  
**mongosh:** 2.14.3  
**Replica set target name:** `rs0`  
**Port:** TCP 27017

## 1. Objective

Build a three-VM MongoDB lab and prepare the nodes for replica-set initialization, while recording installation issues, configuration corrections, service validation and network tests.

## 2. Lab inventory

| Node | Hostname used in lab | IP address | Planned role |
|---|---|---|---|
| VM1 | VILASLAB1 | 192.168.62.128 | Intended preferred PRIMARY |
| VM2 | VILASLAB2 | 192.168.62.131 | Intended SECONDARY |
| VM3 | VILASLAB3 | 192.168.62.132 | Intended SECONDARY |

All three VMs use CentOS Stream 10. The intended topology is a three-member replica set named `rs0`. These are lab intentions, not proof of an initialized replica set.

## 3. Installation and package-management work

### Work recorded

- MongoDB Server 7.0.48 was installed on the three VMs during the lab activity.
- mongosh 2.14.3 was available.
- The MongoDB 7.0 repository was configured for the RHEL 10 / x86_64 package path.
- The MongoDB repository signing-key import/package verification encountered a crypto-policy compatibility issue involving SHA-1 certification.
- A temporary legacy crypto-policy workaround was used during installation, and the system policy was returned to DEFAULT afterward, as recorded in the troubleshooting conversation.
- A separate MongoDB Database Tools package dependency was encountered while installing MongoDB tools. A manual package approach was discussed; the exact final tools installation result was not independently confirmed in the latest evidence.

### Useful validation commands

```bash
mongod --version
mongosh --version
rpm -qa | grep -i mongodb | sort
systemctl status mongod --no-pager
```

### Local database connectivity

The following local ping test was confirmed on nodes where output was shared:

```bash
mongosh --eval 'db.adminCommand({ping:1})'
```

Observed result:

```javascript
{ ok: 1 }
```

## 4. Configuration work

The intended configuration for each VM is:

### VM1 – VILASLAB1

```yaml
net:
  port: 27017
  bindIp: 127.0.0.1,192.168.62.128

replication:
  replSetName: rs0
```

### VM2 – VILASLAB2

```yaml
net:
  port: 27017
  bindIp: 127.0.0.1,192.168.62.131

replication:
  replSetName: rs0
```

### VM3 – VILASLAB3

```yaml
net:
  port: 27017
  bindIp: 127.0.0.1,192.168.62.132

replication:
  replSetName: rs0
```

**Important:** VM1's final corrected listener was confirmed. VM2 and VM3 final configuration contents must still be checked directly on those hosts before proceeding. Do not assume the example above proves their active configuration.

## 5. Configuration error and resolution

### Error

VM1 initially had an incorrect bind address:

```yaml
bindIp: 127.0.0.1,198.168.62.128
```

MongoDB could not bind to the non-existent local address and exited with listener setup failure / exit code 48. The log reported:

```text
Error setting up listener ... Cannot assign requested address
```

### Resolution

The address was corrected to the actual VM1 IP:

```yaml
bindIp: 127.0.0.1,192.168.62.128
```

The service subsequently ran successfully on VM1, listening on loopback and the VM1 interface address.

### Validation captured

```bash
systemctl status mongod --no-pager
ss -lntp | grep 27017
mongosh --eval 'db.adminCommand({ping:1})'
```

Observed VM1 service state: active (running). Listener addresses shown were `127.0.0.1:27017` and `192.168.62.128:27017`. Local ping returned `{ ok: 1 }`.

## 6. Firewall and network validation

A permanent VM1 firewalld rich rule was added to permit TCP 27017 from the lab subnet:

```bash
firewall-cmd --permanent \
  --add-rich-rule='rule family="ipv4" source address="192.168.62.0/24" port port="27017" protocol="tcp" accept'

firewall-cmd --reload
```

VM1 successfully pinged VM2 and VM3 with no packet loss in the tests shared.

TCP checks from VM1:

```bash
nc -zv 192.168.62.131 27017
nc -zv 192.168.62.132 27017
```

Both checks returned a successful TCP connection. This confirms reachability from VM1 to those node addresses on port 27017 at the time of testing; it does not by itself validate replica-set health or bidirectional connectivity.

## 7. Replica-set preparation status

### Confirmed

- The three-node lab inventory and target replica-set name `rs0` were established.
- VM1's corrected bind address and replica-set parameter were part of the confirmed configuration discussion.
- VM1 could reach VM2 and VM3 on TCP 27017.
- MongoDB services were running and local ping checks succeeded on the nodes where evidence was shared.

### Pre-initialization checks – user confirmed completed

The user confirmed that hostname correction and the preceding three-node configuration checks were completed. The earlier checks included hostname resolution, MongoDB configuration review, and port connectivity validation.

### Replica-set initialization – confirmed completed

The user subsequently shared the following command output from the MongoDB shell on VM1:

```javascript
rs.status().members.forEach(m => {
  print(m.name + " : " + m.stateStr)
})
```

Observed output:

```text
VILASLAB1:27017 : PRIMARY
VILASLAB2:27017 : SECONDARY
VILASLAB3:27017 : SECONDARY
```

This verifies that all three configured members reached the intended PRIMARY/SECONDARY states at the time of the check. The shell prompt was `rs0 [direct: primary]`, also consistent with VM1 being the primary.

The user has not yet shared a full `rs.status()`, `rs.conf()`, or `db.hello()` output. Those additional checks remain useful, but are not required to record the member-state result shown above.

### Next validation – data replication

The next practical exercise is to write a test document on the primary and read it from each secondary using a secondary read preference. Do not use production data for this lab. Continue with the commands in the next practical lesson.

### Production security follow-up

Before production-style use, plan internal authentication (keyfile or x.509), user authentication, TLS, and secrets management.

After all preflight checks, initialization was performed from VM1. The member-state output above is the evidence for the resulting cluster state.

```javascript
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "VILASLAB1:27017", priority: 2 },
    { _id: 1, host: "VILASLAB2:27017", priority: 1 },
    { _id: 2, host: "VILASLAB3:27017", priority: 1 }
  ]
})
```

Then validate:

```javascript
rs.status()
rs.status().members.forEach(m => print(m.name + " : " + m.stateStr))
rs.conf()
```

Expected steady-state design: VM1 PRIMARY and VM2/VM3 SECONDARY. Actual election outcomes must be checked, not assumed. Do not re-run `rs.initiate()` if the set has already been initialized.

## 8. Troubleshooting register

| Issue | Impact | Resolution / status |
|---|---|---|
| MongoDB signing-key SHA-1 certification rejected by crypto policy | Package install/verification blocked | Temporary legacy policy used for installation; DEFAULT restored afterward |
| MongoDB Database Tools dependency | Tools package dependency issue | Separate package approach discussed; final installed state needs verification |
| Incorrect VM1 bind IP: 198.168.62.128 | mongod listener startup failed, exit code 48 | Corrected to 192.168.62.128; service subsequently active |
| Initial localhost-only binding | Remote nodes could not connect | Network bind address configured; VM1 listener validated |
| TCP access to remote VM nodes | Required for replica-set communication | VM1-to-VM2 and VM1-to-VM3 port 27017 tests succeeded |

## 9. Next session plan

- Validate hostnames and name resolution on all nodes.
- Validate each node's effective MongoDB config.
- Validate TCP paths among all members.
- Confirm no replica set has already been initiated.
- Initialize `rs0` once all prerequisites are satisfied.
- Verify member states, election behavior and replication.
- Add internal authentication and user authentication before treating the lab as a secured deployment.
- Capture exact command outputs and add them to this repository.

## 10. Replica-set validation evidence

**Verified member states shared by the user:**

```text
VILASLAB1:27017 : PRIMARY
VILASLAB2:27017 : SECONDARY
VILASLAB3:27017 : SECONDARY
```

**Outcome:** The three-node replica set `rs0` is initialized and all three members are reported in the expected steady states. This establishes successful replica-set formation and election at the time of the captured output. A separate data replication read/write test is the next validation step.

## 11. Evidence and accuracy note

This log was assembled from the command outputs and troubleshooting details shared in the working conversation on 24 November 2026. Where a complete output or final configuration was not provided, the item is explicitly marked as pending verification rather than recorded as successful.
