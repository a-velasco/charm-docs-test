# How to manage client connections
[Integrations](https://juju.is/docs/juju/relation) (formerly "relations") are connections between two applications with compatible endpoints. These connections simplify the creation and management of users, passwords, and other shared data.

## Summary
* [Create client connection to **bare replica set**](#heading--replica-set)
* [Create client connection to **sharded cluster**](#heading--sharded-cluster)
* [Rotate client password](#heading--rotate-password)
---
<a href="#heading--replica-set"><h2 id="heading--replica-set">Create a client connection to a **bare replica set**</h2></a>

To create a client connection to a replica set, simply integrate your client application with a Charmed MongoDB application running in replication mode. 

[note]
The legacy `mongodb` interface is being deprecated soon. **Integrations for client connections are now only supported via the [`mongodb_client` interface](https://charmhub.io/mongodb/integrations#database).**

If you do not have a client application that implements this interface, you can use the [`data-integrator` charm](https://github.com/canonical/data-integrator).
[/note]

To integrate `mongodb` with a client application, run:
```shell
juju integrate mongodb <application>
```
### Remove client connection
To remove a client connection and the user associated with it:
```shell
juju remove-relation mongodb <application>
```

<a href="#heading--sharded-cluster"><h2 id="heading--sharded-cluster">Create a client connection to a **sharded cluster**</h2></a>

To create a client connection to a sharded cluster, you must use the [Charmed Mongos](https://charmhub.io/mongos) router.
[note]
`mongos` is a [subordinate charm](https://juju.is/docs/sdk/charm-taxonomy#heading--subordinate-charms) and **must have a host charm.**  If you do not have a suitable host application, you may use the [`data-integrator` charm](https://github.com/canonical/data-integrator).
[/note]

To deploy `mongos` and `data-integrator`, run:
```shell
juju deploy mongos
juju deploy data-integrator [necessary options]
```

Wait for `mongos`'s status to be `idle`, then integrate it with `data-integrator`:
```shell
juju integrate mongos data-integrator
```

Next, integrate the `mongos` charm to a Charmed MongoDB application running as a config-server:

```shell
juju integrate config-server mongos
```
### Remove client connection
To remove a client connection to a sharded cluster:

```shell
juju remove-relation config-server mongos
```

<a href="#heading--rotate-password"><h2 id="heading--rotate-password">Rotate the client password</h2></a>

To rotate client passwords, the integration should be removed and integrated again:

```shell
juju remove-relation <application> mongodb
juju integrate <application> mongodb
```
This process will generate a new user and password for the application. It works for both replica sets and sharded cluster client connections.

### Internal operator user

The `operator` (i.e. admin) user is used internally by the Charmed MongoDB Operator. To rotate its password, use the [`set-password` action](https://charmhub.io/mongodb/actions#set-password).

To set a specific password for the `operator` user, run:

```shell
juju run mongodb/leader set-password password=<password>
```
To randomly generate a password for the `operator` user, run:

```shell
juju run mongodb/leader set-password
```