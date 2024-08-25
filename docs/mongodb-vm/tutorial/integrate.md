> [Charmed MongoDB Tutorials](/t/8061) > [**Deploy a replica set**](/t/8622) > 6. Integrate with other applications
# Integrate MongoDB with other applications

Juju [integrations](https://juju.is/docs/sdk/integration), previously known as "relations", are the easiest way to create a user for MongoDB in Charmed MongoDB. Relations automatically create a username, password, and database for the desired user/application. As mentioned in [Access MongoDB](/t/13237), the best practice is to connect to MongoDB via a specific user rather than the admin user.

In this part of the tutorial, you will learn how to integrate MongoDB with another charm, access an integrated database, and manage users via integrations.
## Summary
* [Deploy the `data-integrator` charm](#heading--deploy-data-integrator)
* [Integrate with MongoDB](#heading--integrate-with-mongodb)
* [Access the integrated database](#heading--access-integrated-database)
* [Remove a user](#heading--remove-user)
* [Recreate a user](#heading--recreate-user)
---

<a href="#heading--deploy-data-integrator"><h2 id="heading--deploy-data-integrator">Deploy the `data-integrator` charm</h2></a>
Before relating to a charmed application, we must first deploy our charmed application. In this tutorial we will relate to the [Data Integrator charm](https://charmhub.io/data-integrator). This is a bare-bones charm that allows for central management of database users, providing support for different kinds of data platforms (e.g. MongoDB, MySQL, PostgreSQL, Kafka, etc) with a consistent, opinionated and robust user experience. In order to deploy the Data Integrator Charm we can use the command `juju deploy` we have learned above:

```shell
juju deploy data-integrator --channel edge --config database-name=test-database
```
The expected output:
```
Located charm "data-integrator" in charm-hub...
Deploying "data-integrator" from charm-hub charm "data-integrator"...
```
<a href="#heading--integrate-with-mongodb"><h2 id="heading--integrate-with-mongodb">Integrate with MongoDB</h2></a>
Now that the Database Integrator Charm has been set up, we can relate it to MongoDB. This will automatically create a username, password, and database for the Database Integrator Charm. Relate the two applications with:
```shell
juju integrate data-integrator mongodb
```
Wait for `juju status --watch 1s` to show:
```
ubuntu@ip-172-31-11-104:~/data-integrator$ juju status
Model     Controller  Cloud/Region         Version  SLA          Timestamp
tutorial  overlord    localhost/localhost  3.1.6   unsupported  10:32:09Z

App                  Version  Status  Scale  Charm                Channel   Rev  Exposed  Message
data-integrator               active      1  data-integrator      edge       3   no
mongodb                       active      2  mongodb              5/edge   96  no

Unit                    Workload  Agent  Machine  Public address  Ports      Message
data-integrator/0*  active    idle   5        10.23.62.216               received mongodb credentials
mongodb/0*              active    idle   0        10.23.62.156    27017/tcp
mongodb/1               active    idle   1        10.23.62.55     27017/tcp  Replica set primary

Machine  State    Address       Inst id        Series  AZ  Message
0        started  10.23.62.156  juju-d35d30-0  jammy       Running
1        started  10.23.62.55   juju-d35d30-1  jammy       Running
5        started  10.23.62.216  juju-d35d30-5  jammy       Running
```
To retrieve information such as the username, password, and database. Enter:
```shell
juju run data-integrator/leader get-credentials
```
This should output something like:
```yaml
​​Running operation 21 with 1 task
  - task 22 on unit-data-integrator-0

Waiting for task 22...
mongodb:
  database: test-database
  endpoints: 10.63.102.106,10.63.102.223
  password: kXOOyTbccn2cXs3kokOqPJz3Mu9g3a5g
  replset: mongodb
  uris: mongodb://relation-2:kXOOyTbccn2cXs3kokOqPJz3Mu9g3a5g@10.63.102.106,10.63.102.223/test-database?replicaSet=mongodb&authSource=admin
  username: relation-2
ok: "True"
```
Save the value listed under `uris:` *(Note: your hostnames, usernames, and passwords will likely be different.)*

<a href="#heading--access-integrated-database"><h2 id="heading--access-integrated-database">Access the integrated database</h2></a>

Notice that in the previous step when you typed `juju run data-integrator/leader get-credentials` the command not only outputted the username, password, and database, but also outputted the URI.  
This means you do not have to generate the URI yourself. To connect to this URI first ssh into `mongodb/0`:
```shell
juju ssh mongodb/0
```
Then access `mongo` with the URI that you copied above:

```shell
charmed-mongodb.mongosh "<uri copied from juju run data-integrator/leader get-credentials>"
```
***Note: be sure you wrap the URI in `"` with no trailing whitespace*.**

You will now be in the mongo shell as the user created for this relation. When you relate two applications Charmed MongoDB automatically sets up a user and database for you. Enter `db.getName()` into the MongoDB shell, this will output:
```shell
test-database
```
This is the name of the database we specified when we first deployed the `data-integrator` charm. To create a collection in the "test-database" and then show the collection enter:
```shell
db.createCollection("test-collection")
show collections
```
Now insert a document into this database:
```shell
db.test_collection.insertOne(
  {
    First_Name: "Jammy",
    Last_Name: "Jellyfish",
  })
```
You can verify this document was inserted by running:
```shell
db.test_collection.find()
```

### Return to original shell
Leave the MongoDB shell by typing `exit`. 
You will be back in the host of Charmed MongoDB (`mongodb/0`). Exit this host by typing `exit` again. 

You should now be at the original shell where you can interact with Juju and LXD.

<a href="#heading--remove-user"><h2 id="heading--remove-user">Remove the user</h2></a>
To remove the user, remove the relation. Removing the relation automatically removes the user that was created when the relation was created. Enter the following to remove the relation:
```shell
juju remove-relation mongodb data-integrator
```

Now try again to connect to the same URI you just used to access the related database:
```shell
juju ssh mongodb/0
charmed-mongodb.mongosh "<uri copied from juju run-action data-integrator/leader get-credentials --wait>"
```
***Note: be sure you wrap the URI in `"` with no trailing whitespace*.**

This will output an error message:
```
Current Mongosh Log ID: 638f5ffbdbd9ec94c2e58456
Connecting to:    mongodb://<credentials>@10.23.62.38,10.23.62.219/mongodb?replicaSet=mongodb&authSource=admin&appName=mongosh+1.6.1
MongoServerError: Authentication failed.
```
As this user no longer exists. This is expected as `juju remove-relation mongodb data-integrator` also removes the user. 

### Return to original shell
Leave the MongoDB shell by typing `exit`. 
You will be back in the host of Charmed MongoDB (`mongodb/0`). Exit this host by typing `exit` again. 

You should now be at the original shell where you can interact with Juju and LXD.

<a href="#heading--recreate-user"><h2 id="heading--recreate-user">Recreate the user</h2></a>

If you wanted to recreate the user we just removed, all you would need to do is relate the two applications again:
```shell
juju integrate data-integrator mongodb
```
Re-relating generates a new password for this user, and therefore a new URI you can see the new URI with:
```shell
juju run data-integrator/leader get-credentials
```
Save the result listed with `uris:`.

You can connect to the database with this new URI:
```shell
juju ssh mongodb/0
charmed-mongodb.mongosh "<uri copied from juju run data-integrator/leader get-credentials>"
```
>**Note**: be sure you wrap the URI in quotation marks (`"`) with no trailing whitespace.

From there if you enter `db.test_collection.find()` you will see all of your original documents are still present in the database.

### Return to original shell
Leave the MongoDB shell by typing `exit`. 
You will be back in the host of Charmed MongoDB (`mongodb/0`). Exit this host by typing `exit` again. 

You should now be at the original shell where you can interact with Juju and LXD.


**Next step**: [7. Enable security ](/t/8628)