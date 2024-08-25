# How to restore a backup

This is a guide on how to perform a basic restore of a local backup of your Charmed MongoDB replica set or sharded cluster.

## Prerequisites
* A replica set with at least three nodes deployed **OR** a sharded cluster with at least one shard deployed
  * See [How To > Scale replicas and shards](/t/8637)
* Configured settings for S3 storage
  * See [How To > Configure S3](/t/8834)
* An existing backup in your S3 storage
  * See [How To > Create a backup](/t/8788)

## Summary
* [List all backups](#heading--list-backups)
* [Restore backup](#heading--restore-backup)

---

<a href="#heading--determine-app"><h2 id="heading--determine-app"> Determine the application to run backup actions on </h2></a>

The application to pass to `juju` when running backup and restore actions depends on whether your deployment is a replica set or sharded cluster:

**When running Charmed MongoDB as a replica-set**, this will be the name of your MongoDB application.

**When running Charmed MongoDB as a sharded cluster**, this will be the name of your MongoDB application running with the “`config-server`” role - **never the shard applications.**


<a href="#heading--list-backups"><h2 id="heading--list-backups"> List all backups </h2></a>
To list all backups available, run:
```shell
juju run <replica-set name | config-server name>/leader list-backups
```

This will display available backups similar to the output below:
```shell
    backup-id             | backup-type  | backup-status
    ----------------------------------------------------
    YYYY-MM-DDTHH:MM:SSZ  | logical      | finished 
```
<a href="#heading--restore-backup"><h2 id="heading--restore-backup"> Restore a backup </h2></a>
> Before restoring a backup, check that your Charmed MongoDB deployment is `active` and `idle` with `juju status`.  

To restore a backup from the list, run the `restore` command using the `backup-id`:
```shell
juju run <replica-set name | config-server name>/leader restore backup-id=YYYY-MM-DDTHH:MM:SSZ
```

The restore will now be in progress.