# How to deploy Charmed MongoDB

This is a guide on how to deploy Charmed MongoDB as a replica set or a sharded cluster.
 
## Summary
* [**Create a replica set**](#heading--replica-set)
  * Deploy single replica
  * Deploy multiple replicas
* [**Create a sharded cluster**](#heading--sharded-cluster)
  * Deploy single replica
  * Deploy multiple replicas
  * Integrate cluster components
---
<a href="#heading--replica-set"><h2 id="heading--replica-set">Create a MongoDB replica set</h2></a>

### Deploy a single replica

**To deploy a single unit of MongoDB as a replica set**, run:
```shell
juju deploy mongodb
```
>This will deploy the latest stable release <!--TODO: link to release pg-->. To specify a different version, use the [`--channel` flag](https://juju.is/docs/sdk/channel), e.g. `--channel=6/beta`.

### Deploy multiple replicas

**To deploy MongoDB with multiple replicas**, specify the number of desired replicas with the `-n` option:
```shell
juju deploy mongodb -n <number_of_replicas>
```

<a href="#heading--sharded-cluster"><h2 id="heading--sharded-cluster">Create a MongoDB sharded cluster</h2></a>

To create a sharded cluster, we must first deploy each cluster component separately with a manually defined role, then integrate them.

### Deploy cluster components with a single replica
Deploying a shard application will assign it one replica by default.

**To deploy a sharded cluster with two shards**, run:
```shell
juju deploy mongodb --config role="config-server" <config_server_name>
juju deploy mongodb --config role="shard" <shard_name_0>
juju deploy mongodb --config role="shard" <shard_name_1>
```
>This will deploy the latest stable release <!--TODO: link to release pg-->. To specify a different version, use the [`--channel` flag](https://juju.is/docs/sdk/channel), e.g. `--channel=6/beta`.

### Deploy cluster components with multiple replicas
To configure the amount of replicas for a shard or config-server during deployment, just use the `-n` flag.

**To deploy a shard and config-server with 3 replicas each**, run:

```shell
juju deploy mongodb --config role="shard" <shard_name> -n 3
juju deploy mongodb --config role="config-server" <config_server_name> -n 3
```
>This will deploy the latest stable release <!--TODO: link to release pg-->. To specify a different version, use the [`--channel` flag](https://juju.is/docs/sdk/channel), e.g. `--channel=6/beta`.

Note that you can also change the number of replicas for a shard or config server after deployment. See [Manage units > How to add shards to a cluster](https://charmhub.io/mongodb/docs/h-manage-units#heading--add-shards-cluster).

### Integrate cluster components
To create a cluster, integrate the shard applications with the config-server application. 

For the case of two shards and one config server, you would run:
```shell
juju integrate config-server:config-server <shard_name_0>:sharding
juju integrate config-server:config-server <shard_name_1>:sharding
```