# How to scale replicas and shards

In this guide you will find instructions on how to scale your [replica set](#heading--replica-set) and [sharded cluster](#heading--sharded-cluster).


## Summary
* [**Scale a replica set**](#heading--replica-set)
  * [Add replicas](#heading--add-replicas) to a replica set
  * [Remove replicas](#heading--remove-replicas) from a replica set
  * [Retrieve primary replica](#heading--retrieve-primary-replica)
* [**Scale a sharded cluster**](#heading--sharded-cluster)
  * [Add shards](#heading--add-shards-cluster) to a cluster
  * [Remove shards](#heading--remove-shards-cluster) from a cluster

---

<a href="#heading--replica-set"><h2 id="heading--replica-set">Scale a replica set</h2></a>

To scale a replica set, simply use `juju`'s  [`add-unit`](https://juju.is/docs/juju/juju-add-unit) and [`remove-unit`](https://juju.is/docs/juju/juju-remove-unit) commands. 

<a href="#heading--add-replicas"><h3 id="heading--add-replicas">Add replicas</h3></a>

To add more replicas, run:
```shell
juju add-unit <application_name> -n <num_of_replicas_to_add>
```
Where an `application` can be either a bare replica set, shard, or config-server.

<a href="#heading--remove-replicas"><h3 id="heading--remove-replicas">Remove replicas</h3></a>

`remove-unit` allows removing more than one replica so long as they **do not constitute the majority of the replicas**. 

To remove replicas, run:
```shell
juju remove-unit <application_name>/<unit_number> <application_name>/<unit_number>
```
Where an `application` can be either a bare replica set, shard, or config-server.

<a href="#heading--retrieve-primary-replica"><h3 id="heading--retrieve-primary-replica">Retrieve primary replica</h3></a>

To retrieve the primary replica, use the `juju` action `get-primary`:
```shell
juju run <application_name>/<unit_number> get-primary
```
Where an `application` can be either a bare replica set, shard, or config-server.

<a href="#heading--sharded-cluster"><h2 id="heading--sharded-cluster">Scale a sharded cluster</h2></a>

<a href="#heading--add-shards-cluster"><h3 id="heading--add-shards-cluster">Add shards to a cluster</h3></a>
To add a shard to a cluster, first deploy the new shard.

To deploy a shard named `new-shard`, run:
```shell
juju deploy mongodb --config role="shard" new-shard -n <number of replicas for shard>
```

Wait for the shard to show `blocked` and `idle` with `juju status --watch 1s`.

Next, add it to your config-server:
```shell
juju integrate <config-server-name>:config-server new-shard:sharding
```

<a href="#heading--remove-shards-cluster"><h3 id="heading--remove-shards-cluster">Remove shards from a cluster</h3></a>
Like official MongoDB, Charmed MongoDB does not support removing the last shard. 

To remove a shard named `new-shard` that is not the last, run:
```shell
juju remove-relation <config-server-name>:config-server new-shard:sharding
```

You can watch your sharded cluster scale down with  `juju status --watch 1s`. 

Once the shard is drained, you can fully remove it with:
```shell
juju remove-application new-shard
```