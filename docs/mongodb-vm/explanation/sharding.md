# Sharding in Charmed MongoDB
This page goes over the logic and implementation behind sharding in Charmed MongoDB. For information about how sharding works in general, we recommend checking out this page from MongoDB's official documentation: [MongoDB | Sharding in MongoDB](https://www.mongodb.com/basics/sharding#:~:text=A%20shard%20is%20a%20replica,into%20a%20consistent%20client%20response.).

In Charmed MongoDB, sharding is designed around the idea of multiple juju applications making up the cluster, unlike replication, where multiple units make up the replica set. These different applications act as cluster components such as shards, config-servers, and mongos-routers. In this page, we will describe these components and the charms they are associated with.

## Summary
* [Sharded cluster components](#heading--components)
  * `config-server`
  * `shard`
  * `mongos`
* [Sharded cluster architecture](#heading--example)
* [Replication-only vs. sharded cluster deployments](#heading--comparison)

---

<a href="#heading--components"><h2 id="heading--components"> Sharded cluster components </h2></a>

### `config-server`
config-servers exist as a configuration of the already existing Charmed MongoDB charm. The config server manages the related sharding applications and is responsible for adding/removing shards, creating users, managing passwords, creating backups, and enabling TLS.

### `shard` 
Shards exist as a configuration of the already existing Charmed MongoDB charm. Once deployed as a shard, shards are unable to create users, manage passwords, and create backups.

### `mongos`
The mongos router is present in two forms in Charmed MongoDB:

1. **Cluster admin** - In this case, `mongos` runs as an administrative daemon within the config-server, managing shards, users, passwords, etc. It is not advisable to access this `mongos` daemon directly in a production environment.
2. **Router for client connections** -  In this case, `mongos` runs as a subordinate charm, [Charmed Mongos](https://charmhub.io/mongos), and acts as an independent router for client applications.

#### Designed for reduced latency
We designed our deployments around creating the most efficient sharded cluster possible. For this reason, the administrative `mongos` router runs within the same virtual machine as the config-server. This means that administrative cluster actions do not need to make additional network calls to configure the cluster. 

On the subordinate [Mongos Charm](https://charmhub.io/mongos), a similar approach is taken for the non-administrative `mongos` router (i.e. the `mongos` client router). In this case, `mongos` is running using the Unix Domain Socket so that client requests to the sharded cluster do not have to make a network call when interfacing with their associated router.   


> :construction:  More information about [Charmed Mongos](https://charmhub.io/mongos) is coming soon.

<a href="#heading--example"><h2 id="heading--example"> Sharded cluster architecture </h2></a>

The image below depicts an example deployment of a sharded cluster in Charmed MongoDB and the relationships between all components:

<img src="https://assets.ubuntu.com/v1/6463e928-updated_sharding_deployment.jpg" alt="A flowchart describing a sharded cluster deployment with two external applications, two shards, and one config-server" title="Sample sharded cluster"/>

For more information about Charmed MongoDB sharded cluster deployments, check the following pages in this documentation:
* [Tutorial > Deploy a sharded cluster > 2. Deploy MongoDB](/t/13291) for beginner-friendly, guided steps
* [How-To > Deploy MongoDB](/t/13457#heading--sharded-cluster) for concise instructions


<a href="#heading--comparison"><h2 id="heading--comparison"> Replication-only vs. sharded cluster deployments </h2></a>

Some key differences between a bare replica set deployment and a sharded cluster deployment include:

| Replication-only                    | Sharded cluster                     |
|-------------------------------------------------|------------------------------------------------|
| Requires one MongoDB application                | Requires multiple MongoDB applications         |
| Can create users via `mongodb_client` interface | Can create users via `config-server` interface |
| Can be integrated with legacy interface         | Cannot be integrated with legacy interface     |