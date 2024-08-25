# Configure S3 storage

This is a guide on how to configure S3 storage for a Charmed MongoDB replica set or sharded cluster.

## Prerequisites
* Access to [Amazon S3](https://aws.amazon.com/s3/) storage.
* A replica set with at least three nodes deployed **OR** a sharded cluster with at least one shard deployed
  * See [How To > Scale replicas and shards](/t/8637). 

## Summary
* [Deploy `s3-integrator`](#heading--deploy-s3)
* [Configure `s3-integrator`](#heading--configure-s3)
* [Pass S3 configuration to MongoDB](#heading--integrate-with-mongodb)

---

<a href="#heading--deploy-s3"><h2 id="heading--deploy-s3">Deploy <code>s3-integrator</code></h2></a>
S3 configurations are managed with the [s3-integrator charm](https://charmhub.io/s3-integrator).

First, deploy the `s3-integrator` charm:
```shell
juju deploy s3-integrator
```
<a href="#heading--configure-s3"><h2 id="heading--configure-s3">Configure <code>s3-integrator</code></h2></a>

To sync your `s3-integrator` credentials, run:
```shell
juju run s3-integrator/leader sync-s3-credentials access-key=<access-key-here> secret-key=<secret-key-here>
```
To configure `s3-integrator`, run `juju config` with the [relevant parameters](https://charmhub.io/s3-integrator/configure) to your S3 storage. For example:
```shell
juju config s3-integrator path="my/path" region="us-west-2" bucket="pbm-test-bucket-1"
```
<a href="#heading--integrate-with-mongodb"><h2 id="heading--integrate-with-mongodb">Pass S3 configuration to MongoDB</h2></a>

### Determine the application to run backup actions on
The application to pass to juju when running backup and restore actions depends on whether your deployment is a replica set or sharded cluster:

**When running Charmed MongoDB as a replica-set**, this will be the name of your Charmed MongoDB Application.

 **When running Charmed MongoDB as a sharded cluster**, this will be the name of your Charmed MongoDB Application running with the role “config-server” - **never the shard applications.**

### Integrate `s3-integrator` with MongoDB
To integrate your deployment with `s3-integrator`, run:

```shell
juju integrate s3-integrator <replica-set name | config-server name>
```

[note]
**Note:** You can also update your configuration options after integrating. For example:
```shell
juju config s3-integrator endpoint=<endpoint>
```
[/note]