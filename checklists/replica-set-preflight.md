# Replica Set Preflight Checklist

- [ ] Confirm actual hostname on every node.
- [ ] Confirm each node resolves every member hostname to the expected private IP.
- [ ] Confirm MongoDB Server versions are compatible and consistent.
- [ ] Confirm each mongod service is active.
- [ ] Confirm each node binds to loopback and its own correct private IP.
- [ ] Confirm every node uses the same replica set name.
- [ ] Confirm TCP 27017 connectivity between all members.
- [ ] Confirm time synchronization.
- [ ] Confirm no previous replica-set initialization exists.
- [ ] Plan keyfile or x.509 internal authentication and access controls.
- [ ] Initialize only once, from one member, after all checks pass.
- [ ] Validate rs.status(), rs.conf(), member states and replication behavior.
