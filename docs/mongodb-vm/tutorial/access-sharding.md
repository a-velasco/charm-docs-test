> [Charmed MongoDB Tutorials](/t/8061) > [**Deploy a sharded cluster**](/t/13290) > 3. Access a sharded cluster

# Access a sharded cluster
In sharded MongoDB clusters, the database is accessed via [`mongos`](https://www.mongodb.com/docs/manual/core/sharded-cluster-query-router/). In Charmed MongoDB clusters, we have an operational `mongos` daemon running as the admin user within the `config-server`. 

Connecting to the database requires a Uniform Resource Identifier (URI). MongoDB expects a [MongoDB-specific URI](https://www.mongodb.com/docs/manual/reference/connection-string/) that contains information used to authenticate us to the database. 

In this part of the tutorial, we will generate a URI containing database credentials 
that will allow us to connect to the database via the `mongos` router.

## Summary
* [Retrieve URI Data](#heading--retrieve-uri-data)
  * Username | `$DB_USERNAME`
  * Password | `$DB_PASSWORD`
* [Generate a URI](#heading--generate-mongodb-uri)
* [Connect to the `mongo` shell](#heading--connect-via-uri) via URI

[note type="caution"]
**Disclaimer:** This part of the tutorial accesses the sharded MongoDB cluster via the `operator` user. This is the admin user in Charmed MongoDB.

**Do not directly interface with the `operator` user in a production environment.** Always connect to MongoDB via a separate user. We will cover this later in this tutorial when we learn how to [integrate with other applications](/t/13348).
[/note]

--- 
<a href="#heading--retrieve-uri-data"><h2 id="heading--retrieve-uri-data">Retrieve URI Data</h2></a>
To access MongoDB via `mongos`, we need to build a URI of the format
```
mongodb://$DB_USERNAME:$DB_PASSWORD@localhost:27018/admin
```
Let's start by retrieving the necessary information for these environment variables.

### Username | `$DB_USERNAME`
In this case, we are using the `admin` user to connect to MongoDB. Use `operator` as the username:
```
export DB_USERNAME="operator"
```
### Password | `$DB_PASSWORD`
The password can be retrieved by running the `get-password` action on the Charmed MongoDB application:
```shell
juju run config-server/leader get-password
```

Running the command above will output

```shell
Running operation 5 with 1 task
  - task 6 on unit-config-server-0

Waiting for task 6...
password: 9JLjd0tuGngW5xFKWWbo0Blxyef0oGec
```
You can manually copy the password displayed above into the `DB_PASSWORD` environment variable, or use the command below to retrieve and assign it in one go:

```shell
export DB_PASSWORD=$(juju run config-server/leader get-password  | grep password|  awk '{print $2}')
```

<a href="#heading--generate-mongodb-uri"><h2 id="heading--generate-mongodb-uri">Generate a MongoDB URI</h2></a>
Now that you have the necessary fields to connect to the URI, you can connect to MongoDB with `charmed-mongodb.mongosh` via the URI. 

You can create the URI with the environment variables you exported earlier:

```shell
export URI=mongodb://$DB_USERNAME:$DB_PASSWORD@localhost:27018/admin
```
Now echo the URI and save it, as you won’t be able to rely on this environment variable when you `ssh` into a unit in the next step:
```shell
echo $URI
```

<a href="#heading--connect-via-uri"><h2 id="heading--connect-via-uri">Connect via MongoDB URI</h2></a>
The `mongo` shell is already installed in Charmed MongoDB as `charmed-mongodb.mongosh`. 
To access the unit hosting Charmed MongoDB, first `ssh` into the config-server unit: 
```shell
juju ssh config-server/0
```
Then use your URI to to access the `mongo` shell:
```shell
charmed-mongodb.mongosh <saved URI>
```

Below is an example output of running this command inside the leader unit:
```shell
ubuntu@juju-573669-0:~$ charmed-mongodb.mongosh mongodb://operator:e4HufmNaNbIdr0ArNgRTWSe7LXmNID8c@10.56.148.138/admin

Current Mongosh Log ID:	65c68b16f117c8adb6cab710
Connecting to:		mongodb://<credentials>@localhost/admin?directConnection=true&appName=mongosh+2.0.1
Using MongoDB:		6.0.6-5
Using Mongosh:		2.0.1
mongosh 2.1.4 is available for download: https://www.mongodb.com/try/download/shell

For mongosh info see: https://docs.mongodb.com/mongodb-shell/

config-server [direct: primary] admin>

```

You can verify which shards are in the sharded cluster by running `sh.status()`

The first few lines will output:
```yaml
shardingVersion
{ _id: 1, clusterId: ObjectId("65c67095d6c910a966c84b68") }
---
shards
[
  {
    _id: 'shard0',
    host: 'shard0/10.56.148.149:27017',
    state: 1,
    topologyTime: Timestamp({ t: 1707504367, i: 4 })
  },
  {
    _id: 'shard1',
    host: 'shard1/10.56.148.60:27017',
    state: 1,
    topologyTime: Timestamp({ t: 1707504376, i: 2 })
  }
]

...
```

Notice how `shard0` and `shard1` are both present under **`shards`**

When you’re ready to leave the MongoDB shell, just type `exit`. You will be back in the host of Charmed MongoDB (`mongodb/0`).

Exit this host by once again typing `exit`. Now you will be in your original shell where you can interact with Juju and LXD.

**Next Step:** [4. Add and remove shards](/t/13302)