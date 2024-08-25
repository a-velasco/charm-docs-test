> [Charmed MongoDB Tutorials](/t/8061) > [**Deploy a replica set**](/t/8622) > 3. Access a replica set

# Access a replica set
MongoDB databases can be accessed via the  `mongosh`, the [MongoDB shell](https://www.mongodb.com/docs/mongodb-shell/). This shell comes pre-installed in the units hosting the Charmed MongoDB application as `charmed-mongodb.mongosh`.

Connecting to the database requires a Uniform Resource Identifier (URI). MongoDB expects a [MongoDB-specific URI](https://www.mongodb.com/docs/manual/reference/connection-string/) that contains information used to authenticate us to the database. 

In this part of the tutorial, we will fetch all the information needed to generate a URI that will grant us access to MongoDB, connect to the `mongo` shell, and create a test database.

[note type="caution"]
**Warning:** This part of the tutorial accesses the sharded MongoDB cluster via the `operator` user. This is the admin user in Charmed MongoDB. 

**Do not directly interface with the `operator` user in a production environment.** 

Always connect to MongoDB via a separate user. We will cover this later in this tutorial when we learn how to [integrate with other applications](/t/8629).
[/note]

## Summary
* [Retrieve URI Data](#heading--retrieve-uri-data)
  * Username | `$DB_USERNAME`
  * Password | `$DB_PASSWORD`
  * Hosts |  `$HOST_IP`
  * Database name | `$DB_NAME`
  * Replica set name | `$REPL_SET_NAME`
* [Generate a URI](#heading--generate-mongodb-uri) 
* [Connect to the `mongo` shell](#heading--connect-via-uri) via URI
* [Create a database](#heading--create-database)

---
<a href="#heading--retrieve-uri-data"><h2 id="heading--retrieve-uri-data">Retrieve URI Data</h2></a>
To access MongoDB via `mongos`, we need to build a URI of the format
```
mongodb://$DB_USERNAME:$DB_PASSWORD@$HOST_IP/$DB_NAME?replicaSet=$REPL_SET_NAME
```
Connecting via URI requires that you know the values for `username`, `password`, `hosts`, `database name`, and the `replica set name`. We will show you how to retrieve the necessary fields and set them to environment variables. 

### Username | `$DB_USERNAME`
**Retrieving the username:** In this case, we are using the `operator` user to connect to MongoDB. Use `operator` as the username:
```shell
export DB_USERNAME="operator"
```
The `operator` user is the admin user in Charmed MongoDB. 
### Password | `$DB_PASSWORD`
Run the `get-password` action on the Charmed MongoDB application:
```shell
juju run mongodb/leader get-password
```
Running the command should output:
```yaml
Running operation 5 with 1 task
  - task 6 on unit-mongodb-0

Waiting for task 6...
password: 9JLjd0tuGngW5xFKWWbo0Blxyef0oGec
```
Use the password under the result: `password`:
```shell
export DB_PASSWORD=$(juju run mongodb/leader get-password  | grep password|  awk '{print $2}')
```

### Hosts | `$HOST_IP`
The hosts are the units hosting the MongoDB application. The host’s IP address can be found with `juju status`:
```
Model     Controller  Cloud/Region         Version  SLA          Timestamp
tutorial  overlord    localhost/localhost  3.1.6   unsupported  11:31:16Z

App      Version  Status  Scale  Charm    Channel   Rev  Exposed  Message
mongodb           active      1  mongodb  5/edge   96  no       Replica set primary

Unit        Workload  Agent  Machine  Public address  Ports      Message
mongodb/0*  active    idle   0        <host IP>    27017/tcp  Replica set primary

Machine  State    Address       Inst id        Series  AZ  Message
0        started  10.23.62.156  juju-d35d30-0  jammy       Running

```
Set the variable `HOST_IP` to the IP address for `mongodb/0`:
```shell
export HOST_IP=$(juju exec --unit mongodb/0 -- hostname -I | awk '{print $1}')
```
### Database name | `$DB_NAME`
 In this case we are connecting to the `admin` database. Use `admin` as the database name. Once we access the database via the MongoDB URI, we will create a `test-db` database to store data.
```shell
export DB_NAME="admin"
```

### Replica set name | `$REPL_SET_NAME`
The replica set name is the name of the application on Juju hosting MongoDB. The application name in this tutorial is `mongodb`. Use `mongodb` as the replica set name. 
```shell
export REPL_SET_NAME="mongodb"
```
### Summary
```shell
export DB_USERNAME="operator"
export DB_PASSWORD=$(juju run mongodb/leader get-password  | grep password|  awk '{print $2}')
export HOST_IP=$(juju exec --unit mongodb/0 -- hostname -I | awk '{print $1}')
export DB_NAME="admin"
export REPL_SET_NAME="mongodb"
```

<a href="#heading--generate-mongodb-uri"><h2 id="heading--generate-mongodb-uri">Generate a MongoDB URI</h2></a>
Now that we have the necessary fields to connect to the URI, we can connect to MongoDB with `charmed-mongodb.mongo` via the URI. We can create the URI with:
```shell
export URI=mongodb://$DB_USERNAME:$DB_PASSWORD@$HOST_IP/$DB_NAME?replicaSet=$REPL_SET_NAME
```
Now echo the URI and save it, as you won't be able to rely on this environment variable when you `ssh` into a unit in the next step.
```shell
echo $URI
```

<a href="#heading--connect-via-uri"><h2 id="heading--connect-via-uri">Connect via MongoDB URI</h2></a>
To access the unit hosting MongoDB, `ssh` into it:
```shell
juju ssh mongodb/0
```

While `ssh`'d into `mongodb/0`, we can access `mongo`, using the URI that we saved in the [previous step](#heading--generate-mongodb-uri).

```shell
charmed-mongodb.mongosh <saved URI>
```

You should now see something similar to:
```
Current Mongosh Log ID: 6389e2adec352d5447551ae0
Connecting to:    mongodb://<credentials>@10.23.62.156/admin?replicaSet=mongodb&appName=mongosh+1.6.1
Using MongoDB:    5.0.14
Using Mongosh:    1.6.1

For mongosh info see: https://docs.mongodb.com/mongodb-shell/


To help improve our products, anonymous usage data is collected and sent to MongoDB periodically (https://www.mongodb.com/legal/privacy-policy).
You can opt-out by running the disableTelemetry() command.

------
   The server generated these startup warnings when booting
   2022-12-02T11:24:05.416+00:00: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine. See http://dochub.mongodb.org/core/prodnotes-filesystem
------

------
   Enable MongoDB's free cloud-based monitoring service, which will then receive and display
   metrics about your deployment (disk utilization, CPU, operation statistics, etc).

   The monitoring data will be available on a MongoDB website with a unique URL accessible to you
   and anyone you share the URL with. MongoDB may use this information to make product
   improvements and to suggest MongoDB products and deployment options to you.

   To enable free monitoring, run the following command: db.enableFreeMonitoring()
   To permanently disable this reminder, run the following command: db.disableFreeMonitoring()
------

mongodb [primary] admin>
```

You can now interact with MongoDB directly using any [MongoDB commands](https://www.mongodb.com/docs/manual/reference/command/). For example, entering 
```shell
show dbs
``` 
should output something like:

```shell
admin   172.00 KiB
config  120.00 KiB
local   404.00 KiB
```

<a href="#heading--create-database"><h2 id="heading--create-database">Create a database</h2></a>
Now that we have access to MongoDB, we can create a database named `test-db`.

In the MongoDB shell, run:
```shell
use test-db
```
Now create a user called `testUser` with read/write access to `test-db`:
```shell
db.createUser({
  user: "testUser",
  pwd: "password",
  roles: [
    { role: "readWrite", db: "test-db" }
  ]
})
```
You can verify that you added the user correctly by entering the command `show users` into the `mongo` shell. This will output:
```json
[
  {
    _id: 'test-db.testUser',
    userId: new UUID("6e841e28-b1bc-4719-bf42-ba4b164fc546"),
    user: 'testUser',
    db: 'test-db',
    roles: [ { role: 'readWrite', db: 'test-db' } ],
    mechanisms: [ 'SCRAM-SHA-1', 'SCRAM-SHA-256' ]
  }
]
```
When you’re ready to leave the MongoDB shell, just type `exit`. You will be back in the host of Charmed MongoDB (`mongodb/0`). 

Exit this host by once again typing `exit`. Now you will be in your original shell where you can interact with Juju and LXD. 

[note]
If you accidentally exit one more time or close your terminal session at some point, note that all the environment variables used in the URI will be removed. 

If this happens, simply re-export these variables as described in [Retrieve URI data](#heading--retrieve-uri-data).
 [/note]


**Next step:** [4. Scale your replicas](/t/8620)