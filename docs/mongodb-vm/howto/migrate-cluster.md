# How to migrate to a new a cluster via restore
This page contains the steps needed to prepare for and perform a restore of a backup from a different cluster, i.e. cluster migration via restore.

>We will use the following terms:
>
>* **new cluster** - the backed-up cluster that we will migrate data from.
>* **old cluster** - the currently active cluster that will be restored to the state of the new cluster.

## Prerequisites
* For **bare replica sets:**
  * The new cluster has a number of replicas equal to or greater than the old cluster.
    * *E.g. Your old cluster had 2 replicas, so your new cluster must have 2 or more replicas*
  * See [How To > Scale replicas and shards](https://charmhub.io/mongodb/docs/h-manage-units)
* For **sharded clusters:**
  * The new cluster has a number of shards and replicas equal to or greater than those in the old cluster.
    * *E.g. Your old cluster had 2 shards, so your new cluster must have 2 or more shards.*
*Your old shard had 4 replicas, so your new shard must have 4 or more replicas.*
  * See [How To > Scale replicas and shards](https://charmhub.io/mongodb/docs/h-manage-units)
* Configured settings for S3 storage
  * See [How To > Configure S3](https://charmhub.io/mongodb/docs/h-configure-s3)
* Backups from the new cluster in your S3 storage
* Password from your new cluster

## Summary
* [Change password of old cluster](#heading--change-password)
* [List all backups](#heading--list-backups)
* [Define remap pattern (sharded cluster only)](#heading--remap-pattern)
* [Restore backup](#heading--restore-backup)
---
<a href="#heading--change-password"><h2 id="heading--change-password"> Change password of old cluster </h2></a>

The migration process will restore the password from the new cluster to your old cluster. Set the password of your old cluster to the new cluster’s password:
```shell
juju run <replica-set name | config-server name>/leader set-password password=<new cluster password>
```

<a href="#heading--list-backups"><h2 id="heading--list-backups"> List all backups </h2></a>

To view the available backups, enter the command `list-backups`:
```shell
juju run <replica-set name | config-server name>/leader list-backups
```

This shows a list of the available backups similar to the output below. Identify which backup-id corresponds to the new cluster.
```shell
    backup-id             | backup-type  | backup-status
    ----------------------------------------------------
    YYYY-MM-DDTHH:MM:SSZ  | logical      | finished 
```

<a href="#heading--remap-pattern"><h2 id="heading--remap-pattern"> Define remap pattern (sharded cluster only) </h2></a>
If you are running Charmed MongoDB as a bare replica set, you can skip this step.

A remap pattern is a pattern that specifies which old components should be migrated to which new components. They are used when we are migrating to a cluster with new names. 

**If you are running Charmed MongoDB as a sharded cluster and you wish to migrate to a cluster with new names, you must generate a remap pattern.** 

The following example shows how to create a remap pattern for a sample cluster migration:

Sample **old cluster**:
```shell
Model    Controller  Cloud/Region         Version  SLA          Timestamp
old-mod  overlord    localhost/localhost  3.1.6    unsupported  17:27:22Z

App            Version  Status   Scale  Charm       Channel  Rev  Exposed  Message
application             active       1  application            0  no
config-server           active       1  mongodb                0  no
shard-one               active       1  mongodb                1  no
shard-two               active       1  mongodb                2  no

Unit                         Workload  Agent  Machine  Public address  Ports            Message
application/0*               active    idle   4        10.130.84.77
config-server/0*             active    idle   0        10.130.84.120   27017-27018/tcp
self-signed-certificates/0*  active    idle   3        10.130.84.32
shard-one/0*                 active    idle   1        10.130.84.78    27017/tcp
shard-two/0*                 active    idle   2        10.130.84.68    27017/tcp

```

Sample **new cluster**:
```shell
Model    Controller  Cloud/Region         Version  SLA          Timestamp
new-mod  overlord    localhost/localhost  3.1.6    unsupported  17:27:22Z

App               Version  Status   Scale  Charm        Channel  Rev  Exposed  Message
application                active       1  application             0  no
config-server-new          active       1  mongodb                 0  no
shard-one-new              active       1  mongodb                 1  no
shard-two-new              active       1  mongodb                 2  no

Unit                         Workload  Agent  Machine  Public address  Ports            Message
application/0*               active    idle   4        10.130.84.77
config-server-new/0*         active    idle   0        10.130.84.120   27017-27018/tcp
self-signed-certificates/0*  active    idle   3        10.130.84.32
shard-one-new/0*             active    idle   1        10.130.84.78    27017/tcp
shard-two-new/0*             active    idle   2        10.130.84.68    27017/tcp
```

The following would be a valid remap pattern: 
```shell
config-server-new=config-server,shard-one-new=shard-one,shard-two-new=shard-two
```

<a href="#heading--restore-backup"><h2 id="heading--restore-backup"> Restore backup </h2></a>

To restore your current cluster to the state of the previous cluster, run the restore command and pass the correct `backup-id` to the command:
 ```shell
juju run <replica-set name | config-server name>/leader restore backup-id=YYYY-MM-DDTHH:MM:SSZ [remap-pattern=<remap-pattern>]
```

Your restore will then be in progress. Once it is complete, your old cluster will represent the state of the new cluster.