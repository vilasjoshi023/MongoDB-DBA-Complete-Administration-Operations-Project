# MongoDB DBA – Complete Administration & Operations Project

A hands-on MongoDB DBA portfolio covering administration, engineering, reliability, security, performance, automation, and production operations.

> **Lab baseline:** MongoDB Server 7.0.48, mongosh 2.14.3, CentOS Stream 10, three-node lab.
> **Documentation approach:** practical commands, validated outputs, troubleshooting evidence, operational runbooks, and clear completion status.

## Project objectives

- Build end-to-end MongoDB DBA capabilities from fundamentals through production operations.
- Maintain reproducible hands-on labs and safe operational procedures.
- Document errors, root causes, remediation, validation, and lessons learned.
- Develop reusable Bash/Python tooling and operational checklists.
- Prepare for MongoDB DBA / Database Engineer / Platform Reliability interviews.

## DBA responsibility domains

See [DBA Role and Responsibilities](docs/DBA-ROLE-AND-RESPONSIBILITIES.md) for the complete responsibility matrix and scope.

1. Installation, upgrades and package lifecycle
2. MongoDB architecture and storage engine
3. Configuration and service management
4. Database, collection and user administration
5. Security, authentication, authorization and auditing
6. Index lifecycle and index design
7. Query and system performance tuning
8. Backup, restore and point-in-time recovery concepts
9. Replica set administration and recovery
10. Sharding and distributed cluster operations
11. Monitoring, alerting and observability
12. Troubleshooting and incident response
13. Linux and operating-system administration
14. Bash/Python automation and scheduling
15. High availability and disaster recovery
16. MongoDB Atlas and cloud operations
17. Production change, patching and capacity management
18. SOPs, runbooks and operational documentation
19. Scenario-based interview preparation
20. End-to-end implementation projects

## Today's lab: 24 November 2026

See [Daily Lab Log](daily-work-logs/2026-11-24-mongodb-installation-and-replica-set-preparation.md).

### Environment

| Node | Hostname | IP address | OS |
|---|---|---|---|
| VM1 | VILASLAB1 | 192.168.62.128 | CentOS Stream 10 |
| VM2 | VILASLAB2 | 192.168.62.131 | CentOS Stream 10 |
| VM3 | VILASLAB3 | 192.168.62.132 | CentOS Stream 10 |

### Completed and verified in today's discussion

- MongoDB Server 7.0.48 installed across the three lab VMs.
- mongosh 2.14.3 available.
- MongoDB service running and local ping checks returned `{ ok: 1 }` on the nodes where shown.
- Repository/GPG and crypto-policy installation issue investigated and worked around for package installation; system policy was returned to DEFAULT afterward, according to the lab activity discussed.
- MongoDB Database Tools dependency was investigated during installation.
- VM1's incorrect bind IP (198.168.62.128) was corrected to 192.168.62.128 after a listener startup failure.
- VM1 configured with replica set name `rs0`.
- VM1 firewall rule opened TCP 27017 from `192.168.62.0/24`.
- VM1 successfully tested TCP connectivity to VM2 and VM3 on port 27017 using `nc -zv`.

### Replica-set work status

Replica-set configuration and network readiness were being prepared. Hostname resolution and all three nodes' final `mongod.conf` settings still need confirmation before initialization. The latest confirmed state does **not** establish that `rs.initiate()` has been run or that PRIMARY/SECONDARY states have been achieved. See the daily log for the safe next steps.

## Repository layout

```text
.
├── README.md
├── docs/
│   └── DBA-ROLE-AND-RESPONSIBILITIES.md
├── daily-work-logs/
│   └── 2026-11-24-mongodb-installation-and-replica-set-preparation.md
├── practical-labs/
├── scripts/
├── configuration-examples/
├── troubleshooting/
├── backup-recovery/
├── monitoring/
├── security/
├── interview-preparation/
└── checklists/
```

Folders for forthcoming lab modules will gain detailed procedures, scripts, test evidence, and runbooks as each exercise is completed.

## Safety and documentation conventions

- Never commit passwords, private keys, keyfiles, certificates, connection strings with credentials, or production data.
- Use placeholders for secrets and lab-specific values in examples.
- Label every procedure as **planned**, **in progress**, **completed**, or **validated**.
- Do not claim success without command output or observable evidence.
- Test destructive operations only on disposable lab data and explain rollback/recovery steps.

## Lab scope disclaimer

This is a learning and portfolio repository. The current three-node lab is not a production deployment. Production runbooks must include security hardening, backups, capacity planning, monitoring, access controls, change approval, and recovery testing before operational use.
