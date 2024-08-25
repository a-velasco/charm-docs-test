# Create a backup

This is a guide on how to create and list backups of a Charmed MongoDB replica set or sharded cluster using Amazon S3 storage. 

## Prerequisites
* Access to [Amazon S3](https://aws.amazon.com/s3/) storage.
* Configured settings for S3 storage
  * See [How To > Configure S3](https://discourse.charmhub.io/t/8834)
* A replica set with at least three nodes deployed **OR** a sharded cluster with at least one shard deployed
  * See [How To > Scale replicas and shards](https://discourse.charmhub.io/t/8637)

## Summary
* [Determine the application to run backup actions on](#heading--determine-app)
* [Create a backup](#heading--create-backup)
* [List backups](#heading--list-backups)
---

<a href="#heading--determine-app"><h2 id="heading--determine-app"> Determine the application to run backup actions on </h2></a>

The application to pass to `juju` when running backup and restore actions depends on whether your deployment is a replica set or sharded cluster:

**When running Charmed MongoDB as a replica-set**, this will be the name of your MongoDB application.

**When running Charmed MongoDB as a sharded cluster**, this will be the name of your MongoDB application running with the “`config-server`” role - **never the shard applications.**

<a href="#heading--create-backup"><h2 id="heading--create-backup"> Create a backup</h2></a>

> Before creating a backup, check that your Charmed MongoDB deployment is `active` and `idle` with `juju status`.  

To create a backup, use the `create-backup` command on the leader unit:
```shell
juju run <replica-set name | config-server name>/leader create-backup
```

<a href="#heading--list-backups"><h2 id="heading--list-backups"> List all backups</h2></a>

To list all available, failed, and in-progress backups, use the `list-backup` command on the leader unit:
```shell
juju run <replica-set name | config-server name>/leader list-backups
```