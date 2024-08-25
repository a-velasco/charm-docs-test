> [Charmed MongoDB Tutorials](/t/8061) > [**Deploy a sharded cluster**](/t/13290) > 6. Integrate with other applications

# Integrate MongoDB with other applications
While `mongos` exists inside our application `config-server`, its only purpose is to perform database admin operations. We have used this process throughout the tutorial so far to simplify our environment while learning about MongoDB operations, but doing so **is not suitable for a production deployment**.

In order to access the cluster safely, we will use Juju [integrations](https://juju.is/docs/juju/integration) to connect to [Charmed Mongos](https://charmhub.io/mongos) - a charm that acts as a router to connect client applications to sharded clusters. 

Integrations are the easiest way to create a user for MongoDB in Charmed MongoDB. An integration automatically creates a username, password, and database for the desired user/application. In this section of the tutorial, we will learn how to correctly access our MongoDB cluster by integrating with Charmed MongoDB. We will also see how user management works via integrations.

## Summary
* [Deploy `mongos` and `data-integrator` charms](#heading--deploy-mongos-and-data-integrator)
* [Integrate with MongoDB](#heading--integrate-with-mongodb)
* [Access the integrated database](#heading--access-integrated-database)
* [Remove a user](#heading--remove-user)
* [Recreate a user](#heading--recreate-user)

---
<a href="#heading--deploy-mongos-and-data-integrator"><h2 id="heading--deploy-mongos-and-data-integratorr">Deploy the `mongos` and `data-integrator` charms</h2></a>
Since Charmed Mongos is a [subordinate charm](https://juju.is/docs/sdk/charm-taxonomy#heading--subordinate-charms), it requires a [principal charm](https://juju.is/docs/sdk/charm-taxonomy#heading--principal-charms) to host it. For this tutorial, we will use the [`data-integrator`](https://charmhub.io/data-integrator) charm to host the `mongos` charm. 

Deploy `mongos` and `data-integrator`

```shell
juju deploy data-integrator --channel edge --config database-name=test-database
juju deploy mongos --channel 6/edge
```

<details>
<summary> Watch <code>juju status --watch 1s</code> until the <code>data-integrator</code> status is <code>blocked</code> and the <code>mongos</code> status is <code>unknown</code>  </summary>

```shell
Model     Controller  Cloud/Region         Version  SLA          Timestamp
tutorial  overlord    localhost/localhost  3.1.7    unsupported  14:30:39+01:00

App              Version  Status   Scale  Charm            Channel  Rev  Exposed  Message
config-server             active       1  mongodb          6/beta   149  no       Primary
data-integrator           blocked      1  data-integrator  stable    19  no       Please specify either topic, index, or database name
mongos                    unknown      0  mongos           6/edge     5  no
shard0                    active       1  mongodb          6/beta   149  no       Primary
shard1                    active       1  mongodb          6/beta   149  no       Primary

Unit                Workload  Agent  Machine  Public address  Ports            Message
config-server/0*    active    idle   0        10.67.56.193    27017-27018/tcp  Primary
data-integrator/0*  blocked   idle   4        10.67.56.157                     Please specify either topic, index, or database name
shard0/0*           active    idle   1        10.67.56.197    27017/tcp        Primary
shard1/0*           active    idle   2        10.67.56.194    27017/tcp        Primary

Machine  State    Address       Inst id        Base          AZ  Message
0        started  10.67.56.193  juju-5646f0-0  ubuntu@22.04      Running
1        started  10.67.56.197  juju-5646f0-1  ubuntu@22.04      Running
2        started  10.67.56.194  juju-5646f0-2  ubuntu@22.04      Running
4        started  10.67.56.157  juju-5646f0-4  ubuntu@22.04      Running

Integration provider                   Requirer                               Interface              Type     Message
config-server:config-server            shard0:sharding                        shards                 regular
config-server:config-server            shard1:sharding                        shards                 regular
config-server:database-peers           config-server:database-peers           mongodb-peers          peer
data-integrator:data-integrator-peers  data-integrator:data-integrator-peers  data-integrator-peers  peer
mongos:router-peers                    mongos:router-peers                    mongos-peers           peer     joining
shard0:database-peers                  shard0:database-peers                  mongodb-peers          peer
shard1:database-peers                  shard1:database-peers                  mongodb-peers          peer
```
</details>

<a href="#heading--integrate-with-mongodb"><h2 id="heading--integrate-with-mongodb">Integrate with MongoDB</h2></a>
Now we will connect our client application, `data-integrator`, to the cluster via our `mongos` router:

```shell
juju integrate mongos data-integrator
juju integrate mongos config-server
```

<details>
<summary> Watch <code>juju status --watch 1s --relations  </code> until the integrations are ready.  </summary>

```shell
Model     Controller  Cloud/Region         Version  SLA          Timestamp
tutorial  overlord    localhost/localhost  3.1.7    unsupported  18:40:10+01:00

App              Version  Status  Scale  Charm            Channel  Rev  Exposed  Message     
config-server             active      1  mongodb          6/beta   149  no       Primary     
data-integrator           active      1  data-integrator  edge      19  no                                                                      
mongos                    active      1  mongos           6/edge     5  no                                                  
shard0                    active      1  mongodb          6/beta   149  no       Primary     
shard1                    active      1  mongodb          6/beta   149  no       Primary     
                                                                                          
Unit                Workload  Agent  Machine  Public address  Ports            Message       
config-server/0*    active    idle   0        10.67.56.193    27017-27018/tcp  Primary                
data-integrator/0*  active    idle   6        10.67.56.187                                                                                                    
  mongos/0*         active    idle            10.67.56.187    27018/tcp                                                                        
shard0/0*           active    idle   1        10.67.56.197    27017/tcp        Primary       
shard1/0*           active    idle   2        10.67.56.194    27017/tcp        Primary       
                                                                        
Machine  State    Address       Inst id        Base          AZ  Message
0        started  10.67.56.193  juju-5646f0-0  ubuntu@22.04      Running                   
1        started  10.67.56.197  juju-5646f0-1  ubuntu@22.04      Running                   
2        started  10.67.56.194  juju-5646f0-2  ubuntu@22.04      Running          
6        started  10.67.56.187  juju-5646f0-6  ubuntu@22.04      Running                                             
                                                                                                                     
Integration provider                   Requirer                               Interface              Type         Message
config-server:cluster                  mongos:cluster                         config-server          regular
config-server:config-server            shard0:sharding                        shards                 regular         
config-server:config-server            shard1:sharding                        shards                 regular         
config-server:database-peers           config-server:database-peers           mongodb-peers          peer            
data-integrator:data-integrator-peers  data-integrator:data-integrator-peers  data-integrator-peers  peer       
data-integrator:mongos                 mongos:mongos_proxy                    mongos_client          subordinate         
mongos:router-peers                    mongos:router-peers                    mongos-peers           peer
shard0:database-peers                  shard0:database-peers                  mongodb-peers          peer
shard1:database-peers                  shard1:database-peers                  mongodb-peers          peer
```
</details>

Retrieve the `mongos` URI

```shell
juju run data-integrator/leader get-credentials
```

This should output something like:

```shell
Running operation 19 with 1 task
  - task 20 on unit-data-integrator-2

Waiting for task 20...
mongos:
  data: '{"database": "test-database", "external-node-connectivity": "true", "requested-secrets":
    "[\"username\", \"password\", \"tls\", \"tls-ca\", \"uris\"]"}'
  database: test-database
  password: gRBRV3SAVV9hKWHxocI7KyZgkQgP184W
  uris: mongodb://relation-16:gRBRV3SAVV9hKWHxocI7KyZgkQgP184W@10.67.56.187:27018/test-database?&authSource=admin
  username: relation-16
ok: "True"

```

Save the value listed under **`uris`**. We will use this later to access the database through `mongos`.

<a href="#heading--access-integrated-database"><h2 id="heading--access-integrated-database">Access the integrated database</h2></a>
Instead of connecting to `mongos` via the `config-server` application like we did in [3. Access a sharded cluster](), we will access `mongos` directly via the host application. This is because each application related to the sharded mongodb cluster will have their own `mongos` router. 

Additionally, the sharded cluster from Charmed MongoDB will automatically create a special username and password for the `data-integrator` charm. This is the best way to access the sharded Charmed MongoDB cluster.

Access the unit hosting `mongos`:
```shell
juju ssh mongos/0
```
Then, access the MongoDB shell with the URI you saved in the [previous step](#heading--integrate-with-mongodb). Make sure to wrap the URI in `"` with no trailing whitespace:

```shell
charmed-mongodb.mongosh "<URI>"
```

You are now in the MongoDB shell as a new user created specifically for the `data-integrator` charm. 

When you relate two applications, Charmed MongoDB automatically sets up a user and database for you. Enter `db.getName() ` into the MongoDB shell, this will output:

```
test-database
```

This is the name of the database we specified when we first deployed the `data-integrator` charm. 

To create a collection in `test-database` and then show the collection, enter:

```shell
db.createCollection("test-collection")
show collections
```

Now insert a document into this database:

```json
db.test_collection.insertOne(
{
First_Name: "Jammy",
Last_Name: "Jellyfish",
})
```

You can verify this document was inserted by running:
```
db.test_collection.find()
```

Leave the MongoDB shell by typing `exit`. 

You will be back in the host of Charmed MongoDB (`mongodb/0`). Exit this host by typing `exit` again. 

You should now be at the original shell where you can interact with Juju and LXD.

<a href="#heading--remove-user"><h2 id="heading--remove-user">Remove the user</h2></a>
Removing the integration automatically removes the user that was created with it, and stops the `mongos` router. 

To remove the relation, run:
```
juju remove-relation mongos config-server
```

Now try again to connect to the same URI you just used to [access the integrated database](#heading--access-integrated-database):

```
juju ssh mongos/0
charmed-mongodb.mongosh "<URI>"
```

You will see an error like the one shown below, since the user no longer exists.

```
MongoNetworkError: connect ECONNREFUSED 10.67.56.187:27018
```

Now run `exit` once to leave the unit.

You should now be in the shell you started in where you can interact with Juju and LXD.

<a href="#heading--recreate-user"><h2 id="heading--recreate-user">Recreate the user</h2></a>

If you wanted to recreate this user all you would need to do is relate the the two applications again:

```shell
juju integrate config-server mongos
```

Re-relating generates a new password for this user, and therefore a new URI you can see the new URI with:

```shell
juju run data-integrator/leader get-credentials
```

**Next step**: [7. Enable security ](/t/13349)