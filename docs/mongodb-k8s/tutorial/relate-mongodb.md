# Relate your MongoDB deployment 

This is part of the [Charmed MongoDB K8s Tutorial](/t/charmed-mongodb-k8s-tutorial/10592). Please refer to this page for more information and the overview of the content. 

## Relations
Relations, or what Juju documentation [describes as Integration](https://juju.is/docs/sdk/integration), 
are the easiest way to create a user for MongoDB in Charmed MongoDB K8s. Relations automatically create a username, password, and database for the desired user/application. As mentioned earlier in the [Access MongoDB section](https://discourse.charmhub.io/t/charmed-mongodb-k8s-tutorial-deploy-mongodb/10608#access-mongodb) it is a better practice to connect to MongoDB via a specific user rather than the admin user.

### Data Integrator Charm
Before relating to a charmed application, we must first deploy our charmed application. In this tutorial we will relate to the [Data Integrator Charm](https://charmhub.io/data-integrator).  
This is a bare-bones charm that allows for central management of database users, providing support for different kinds of data platforms products (e.g. MongoDB, MySQL, PostgreSQL, Kafka, etc) with a consistent, opinionated and robust user experience.  
In order to deploy the Data Integrator Charm we can use the command `juju deploy` we have learned previously:

```shell
juju deploy data-integrator --channel edge --config database-name=test-database
```
The expected output:
```
Located charm "data-integrator" in charm-hub...
Deploying "data-integrator" from charm-hub charm "data-integrator"...
```

### Relate to MongoDB
Now that the Database Integrator Charm has been set up, we can relate it to MongoDB. This will automatically create a username, password, and database for the Database Integrator Charm. Relate the two applications with:
```shell
juju relate data-integrator mongodb-k8s
```
Wait for `juju status --watch 1s` to show:
```shell
Model     Controller  Cloud/Region        Version  SLA          Timestamp
tutorial  overlord    microk8s/localhost 3.1.6   unsupported  17:24:23Z

App              Version  Status  Scale  Charm            Channel  Rev  Address         Exposed  Message
data-integrator           active      1  data-integrator  edge      11  10.152.183.190  no
mongodb-k8s               active      2  mongodb-k8s      6/beta    37  10.152.183.20   no

Unit                Workload    Agent  Address      Ports  Message
data-integrator/0*  active      idle   10.1.42.142
mongodb-k8s/0*      active      idle   10.1.42.137
mongodb-k8s/1       active      idle   10.1.42.140
```

To retrieve information such as the username, password, and database. Type:

```shell
juju run data-integrator/leader get-credentials
```
This should output something like:
```yaml
Running operation 19 with 1 task
  - task 20 on unit-data-integrator-0

Waiting for task 20...
mongodb:
  database: test-database
  endpoints: mongodb-k8s-0.mongodb-k8s-endpoints,mongodb-k8s-1.mongodb-k8s-endpoints
  password: iW19U0ClFbSDurxMju98Zsgy4BovY4uf
  replset: mongodb-k8s
  uris: mongodb://relation-2:iW19U0ClFbSDurxMju98Zsgy4BovY4uf@mongodb-k8s-0.mongodb-k8s-endpoints,mongodb-k8s-1.mongodb-k8s-endpoints/test-database?replicaSet=mongodb-k8s&authSource=admin
  username: relation-2
ok: "True"
```
Save the value listed under `uris:` *(Note: your hostnames, usernames, and passwords will likely be different.)*

### Access the related database
Notice that in the previous step when you typed `juju run-action data-integrator/leader get-credentials --wait` the command not only outputted the username, password, and database, but also outputted the URI. This means you do not have to generate the URI yourself. To connect to this URI first ssh into `mongodb-k8s/0`:
```shell
juju ssh --container=mongod mongodb-k8s/0
```
Then access `mongo` with the URI that you copied above:

```shell
mongosh "<uri copied from juju run-action data-integrator/leader get-credentials --wait>"
```
***Note: be sure you wrap the URI in `"` with no trailing whitespace*.**

You will now be in the mongo shell as the user created for this relation. When you relate two applications Charmed MongoDB K8s automatically sets up a user and database for you.  
Enter `db.getName()` into the MongoDB shell, this will output:
```shell
test-database
```

This is the name of the database we specified when we first deployed the `data-integrator` charm.  
To create a collection in the "test-database" and then show the collection enter:
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

Now exit the MongoDB shell by typing:
```shell
exit
```
Now you should be back in the host of the Charmed MongoDB K8s's unit: `mongodb-k8s/0`. To exit this host type:
```shell
exit
```
You should now be in the shell you started where you can interact with Juju and Kubernetes (Microk8s).

### Remove the user
To remove the user, remove the relation. Removing the relation automatically removes the user that was created when the relation was created. Enter the following to remove the relation:
```shell
juju remove-relation mongodb-k8s data-integrator
```

Now try again to connect to the same URI you just used in [Access the related database](#access-the-related-database):
```shell
juju ssh --container=mongod mongodb-k8s/0
mongosh "<uri copied from juju run-action data-integrator/leader get-credentials --wait>"
```
***Note: be sure you wrap the URI in `"` with no trailing whitespace*.**

This will output an error message:
```
Current Mongosh Log ID: <your connection id>
Connecting to:          mongodb://<credentials>@mongodb-k8s-0.mongodb-k8s-endpoints,mongodb-k8s-1.mongodb-k8s-endpoints/test-database?replicaSet=mongodb-k8s&authSource=admin&appName=mongosh+2.0.1
MongoServerError: Authentication failed.
```
As this user no longer exists. This is expected as `juju remove-relation mongodb-k8s data-integrator` also removes the user. 

Now exit the juju:
```shell
exit
```

You should now be in the shell you started where you can interact with Juju and Kubernetes (Microk8s).

If you wanted to recreate this user all you would need to do is relate the two applications again:
```shell
juju relate mongodb-k8s data-integrator
```
Re-relating generates a new password for this user, and therefore a new URI you can see the new URI with:
```shell
juju run data-integrator/leader get-credentials
```
Save the result listed with `uris:`.

You can connect to the database with this new URI:
```shell
juju ssh --container=mongod mongodb-k8s/0
mongosh "<uri copied from juju run-action data-integrator/leader get-credentials --wait>"
```
*Note: be sure you wrap the URI in `"` with no trailing whitespace*.

From there if you enter `db.test_collection.find()` you will see all of your original documents are still present in the database.From there if you enter `db.test_collection.find()` you will see all of your original documents are still present in the database.

Now exit the MongoDB shell by typing:
```shell
exit
```
Now you should be back in the host of the Charmed MongoDB K8Ss's unit: `mongodb-k8s/0`. To exit this host type:
```shell
exit
```
You should now be in the shell you started where you can interact with Juju and Kubernetes (Microk8s).

## Next Steps
[Charmed MongoDB K8s Tutorial - Enable security](https://discourse.charmhub.io/t/charmed-mongodb-k8s-tutorial-enable-security/10614)