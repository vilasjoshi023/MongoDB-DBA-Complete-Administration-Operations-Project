# MongoDB DBA – Roles and Responsibilities

This document defines the capability map for a MongoDB Database Administrator / Database Engineer. Each domain is intended to become a practical module with procedures, command examples, validation evidence, troubleshooting cases, and operational checklists.

## 01. Installation and lifecycle management

**Responsibilities**
- Plan supported MongoDB versions and platform prerequisites.
- Configure official repositories and package signing verification.
- Install server, shell and Database Tools components.
- Manage package dependencies, upgrades, patching and rollback planning.
- Start, stop, enable and troubleshoot the `mongod` service.
- Validate version, service health, ports, logs and local connectivity.

**Practical evidence**
- Installation record, package inventory, version check, service status, upgrade/rollback runbook.

## 02. MongoDB architecture

**Responsibilities**
- Explain `mongod`, `mongos`, replica-set members and config servers.
- Understand databases, collections, documents, BSON and indexes.
- Understand WiredTiger, cache, journaling, checkpoints and storage behavior.
- Explain client request flow, read/write paths and replication concepts.
- Understand the roles of oplog, elections and cluster metadata.

**Practical evidence**
- Architecture diagrams, component mapping, data-flow notes and troubleshooting exercises.

## 03. Configuration and service management

**Responsibilities**
- Maintain `mongod.conf` and configuration baselines.
- Manage port, bind IP, storage paths, logging, process management and replication settings.
- Apply configuration changes safely and validate service restart/reload behavior.
- Maintain environment-specific configuration and change records.
- Review startup logs and configuration drift.

**Practical evidence**
- Sanitized configuration examples, parameter guide, service-management SOP.

## 04. Database and collection administration

**Responsibilities**
- Create and manage databases and collections.
- Perform CRUD operations and data validation.
- Manage collection options, validators, capped collections and TTL behavior where appropriate.
- Manage users, roles and least-privilege access.
- Monitor database and collection statistics and storage growth.
- Plan data lifecycle and retention.

**Practical evidence**
- Administration scripts, user/role matrix, CRUD exercises and collection maintenance runbooks.

## 05. Security and compliance

**Responsibilities**
- Enable and validate authentication.
- Apply role-based access control (RBAC) and least privilege.
- Manage users, built-in roles and custom roles.
- Configure transport encryption (TLS) and internal member authentication (keyfile or certificate-based mechanisms).
- Protect secrets, certificates, keyfiles and backups.
- Configure and review auditing where supported by the deployment/licensing model.
- Review access, rotate credentials and investigate security events.

**Practical evidence**
- Hardened lab configuration, access-review checklist and security incident scenarios.

## 06. Index management

**Responsibilities**
- Design single-field, compound, multikey, text, geospatial, TTL and wildcard indexes as workload requires.
- Review index usage, selectivity, cardinality and write overhead.
- Analyze query plans and index bounds.
- Build, hide, modify and remove indexes safely.
- Detect redundant or unused indexes.
- Balance query performance against storage and write costs.

**Practical evidence**
- Index design exercises, explain plans, index usage review and change/rollback procedure.

## 07. Performance and query tuning

**Responsibilities**
- Diagnose slow operations and query latency.
- Use `explain()` and profiler/diagnostic information appropriately.
- Review CPU, memory, WiredTiger cache, disk I/O, locks, queues and connection behavior.
- Tune query shape, indexes, schema and application access patterns.
- Identify hot documents, expensive aggregations and inefficient scans.
- Establish baselines and measure before/after changes.

**Practical evidence**
- Slow-query investigations, before/after plans, tuning reports and performance baselines.

## 08. Backup and recovery

**Responsibilities**
- Define RPO/RTO with service owners.
- Plan logical and physical backup approaches appropriate to deployment topology.
- Use `mongodump` and `mongorestore` correctly for supported use cases.
- Protect, encrypt, retain and monitor backups.
- Understand filesystem/storage snapshots and consistency requirements.
- Design point-in-time recovery using supported backup and oplog strategies.
- Perform restore drills and verify recovered data.

**Practical evidence**
- Backup schedule, restore validation, recovery checklist and recovery test report.

## 09. Replication and replica-set administration

**Responsibilities**
- Deploy and maintain replica sets.
- Understand PRIMARY/SECONDARY/ARBITER roles and voting configuration.
- Monitor replication health, lag, oplog window and member state.
- Understand elections, heartbeats, write concern, read preference and rollback behavior.
- Perform planned maintenance, member replacement and recovery.
- Test failover and document application impact.

**Practical evidence**
- Three-node lab, replica-set status reports, failover exercise and recovery runbook.

## 10. Sharding and distributed cluster operations

**Responsibilities**
- Understand shard clusters, shards, config-server replica sets and `mongos`.
- Design shard keys and evaluate cardinality, distribution and query targeting.
- Monitor balancing, chunk distribution and migration impact.
- Diagnose scatter-gather queries and uneven data distribution.
- Plan shard expansion and maintenance.

**Practical evidence**
- Sharded-cluster lab, shard-key evaluation and balancing/diagnostic exercises.

## 11. Monitoring, metrics and alerting

**Responsibilities**
- Monitor availability, connections, operations, latency and resource utilization.
- Use `mongostat`, `mongotop`, logs and database diagnostic commands.
- Collect host metrics and database metrics.
- Build dashboards and actionable alerts.
- Monitor replication lag, storage growth, cache pressure, backup success and certificate expiry.
- Establish operational thresholds and escalation paths.

**Practical evidence**
- Monitoring checklist, dashboard specification, alert runbooks and sample health checks.

## 12. Troubleshooting and incident response

**Responsibilities**
- Resolve service startup and connection failures.
- Diagnose authentication, authorization, DNS, firewall and TLS issues.
- Investigate slow queries, high CPU, memory pressure and disk-full conditions.
- Diagnose replication lag, elections, member health and oplog-window problems.
- Use logs, metrics and evidence to isolate root cause.
- Document impact, timeline, mitigation, root cause and preventive actions.

**Practical evidence**
- Troubleshooting playbooks, incident reports and fault-injection labs.

## 13. Linux and operating-system administration

**Responsibilities**
- Manage systemd services, processes, users, groups and file permissions.
- Monitor CPU, memory, swap, filesystem capacity and I/O.
- Configure DNS/host resolution, routing, firewall rules and port access.
- Understand file descriptors, limits, time synchronization and kernel/system logs.
- Apply OS patching and operational hardening.
- Coordinate OS and database maintenance windows.

**Practical evidence**
- Linux DBA command reference, host baseline and OS troubleshooting scenarios.

## 14. Automation and scripting

**Responsibilities**
- Automate health checks, log review, backups and validation.
- Use Bash and Python with safe error handling and logging.
- Schedule jobs using cron or systemd timers.
- Automate repeatable deployment/configuration tasks where appropriate.
- Store scripts in version control and validate them before operational use.
- Avoid exposing credentials in scripts or logs.

**Practical evidence**
- Health-check scripts, backup wrappers, inventory and report generators.

## 15. High availability and disaster recovery

**Responsibilities**
- Design service availability and failure-domain strategy.
- Plan replica-set topology, quorum and maintenance behavior.
- Define RPO/RTO and recovery dependencies.
- Run failover, restore and disaster-recovery exercises.
- Document regional/site outage and node replacement procedures.
- Validate application reconnect and retry behavior.

**Practical evidence**
- HA design, failure scenarios, DR plan and periodic test report.

## 16. Cloud operations and MongoDB Atlas

**Responsibilities**
- Understand Atlas cluster tiers, networking, users, roles and project organization.
- Configure private connectivity and IP access controls.
- Monitor metrics, alerts, backups and maintenance.
- Manage scaling, upgrades and cloud cost drivers.
- Understand cloud-provider networking and identity integrations.
- Apply secure operational practices to managed database services.

**Practical evidence**
- Atlas sandbox exercises, deployment checklist and monitoring review.

## 17. Production operations and capacity management

**Responsibilities**
- Perform capacity forecasting for storage, memory, CPU and IOPS.
- Plan and execute patches, upgrades and maintenance.
- Follow change approval, deployment validation and rollback procedures.
- Manage incidents, problem records and operational handovers.
- Coordinate with application, infrastructure, security and vendor teams.
- Review SLAs, SLOs, availability and recurring operational risks.

**Practical evidence**
- Capacity report, change plan, maintenance checklist and incident timeline.

## 18. DBA documentation and operational governance

**Responsibilities**
- Maintain SOPs, runbooks, architecture diagrams and configuration baselines.
- Document backup/restore, failover, patching and access procedures.
- Maintain inventory, ownership, dependencies and escalation contacts.
- Record incidents, known errors, risks and corrective actions.
- Keep documentation version-controlled and reviewed.

**Practical evidence**
- SOP library, runbook index, operational checklist and change templates.

## 19. Interview and technical readiness

**Responsibilities**
- Explain MongoDB architecture and operational decisions clearly.
- Demonstrate hands-on administration and troubleshooting.
- Solve scenario-based performance, backup, security and HA problems.
- Explain trade-offs and failure modes rather than memorizing definitions.
- Present project evidence and lessons learned.

**Practical evidence**
- Topic-wise questions, scenario answers, mock interviews and lab demonstrations.

## 20. End-to-end practical projects

**Responsibilities**
- Deliver practical projects spanning design, implementation, validation and operations.
- Maintain reproducible labs and sanitized configuration.
- Document acceptance criteria, test cases and outcomes.
- Demonstrate operational readiness, recovery and troubleshooting.

**Suggested projects**
1. Single-node MongoDB installation and administration.
2. Three-node replica set with failover and recovery testing.
3. Authentication, RBAC and TLS hardening lab.
4. Backup/restore and recovery validation.
5. Index and query performance tuning.
6. Monitoring and alerting lab.
7. Sharded-cluster deployment and shard-key evaluation.
8. Bash/Python DBA automation toolkit.
9. MongoDB Atlas deployment and operational review.
10. Production-style incident and disaster-recovery simulation.

---

## Operational competency standard

A topic is not considered complete merely because a command was executed. Each module should capture:

1. Objective and architecture/context.
2. Prerequisites and risk assessment.
3. Commands/configuration used.
4. Expected and actual output.
5. Validation and monitoring evidence.
6. Failure scenarios and troubleshooting.
7. Recovery/rollback procedure.
8. Lessons learned and interview notes.

## Credential and data protection

Never commit production data, passwords, private keys, keyfiles, certificates, access tokens, or unredacted connection strings. Use dummy data and sanitized lab examples only.
