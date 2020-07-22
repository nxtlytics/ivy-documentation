# etcd resources

## KubeCon 2018 deep dive

- Metrics to watch
  - `etcd_disk_wal_fsync_duration_seconds_bucket`
  - `etcd_disk_backend_commit_duration_seconds_bucket`
  - `etcd_server_leader_changes_seen_total`
  - `etcd_mvcc_db_total_size_in_bytes` # Never let this get to 2.1 GB
- Write an etcd recovery handbook
- https://youtu.be/GJqO1TYzVDE
- They mention how to safely upgrade
  - [Upgrades documentation](https://github.com/etcd-io/etcd/tree/master/Documentation/upgrades)
  - [Downgrade issue to keep in mind](https://github.com/etcd-io/etcd/issues/9306)
- [Tuning etcd](https://github.com/etcd-io/etcd/blob/master/Documentation/tuning.md)
- [Maintenance](https://github.com/etcd-io/etcd/blob/master/Documentation/op-guide/maintenance.md)
- [Nick Young etcd presentation](https://static.sched.com/hosted_files/k8sforumsydney19/e4/etcd%20presentation%20takeaway.pdf)
