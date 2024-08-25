> [Charmed MongoDB Tutorials](/t/8061) > [**Deploy a replica set**](/t/8622) > 4. Scale your replicas

# Scale your replicas
A replica set in MongoDB is a group of processes that copy stored data in order to make a database highly available. Replication provides redundancy, which means the application can provide self-healing capabilities in case one replica fails. 

[note type="caution"]
**Disclaimer:** This tutorial hosts all shards on the same machine. **This should never be done in a production environment**. 

To enable high availability in a production environment, replicas should be hosted on different servers to [maintain isolation.](https://canonical.com/blog/database-high-availability)
[/note]

## Summary
* [Add replicas](#heading--add-replicas) to your MongoDB cluster
* [Remove replicas](#heading--remove-replicas) from your MongoDB cluster

---

<a href="#heading--add-replicas"><h2 id="heading--add-replicas">Add replicas</h2></a>
You can add two replicas to your deployed MongoDB application with:
```shell
juju add-unit mongodb -n 2
```

You can now watch the replica set add these replicas with: `juju status --watch 1s`. It usually takes several minutes for the replicas to be added to the replica set. You’ll know that all three replicas are ready when `juju status --watch 1s` reports:
```
Model     Controller  Cloud/Region         Version  SLA          Timestamp
tutorial  overlord    localhost/localhost  3.1.6   unsupported  14:42:04Z

App      Version  Status  Scale  Charm    Channel   Rev  Exposed  Message
mongodb           active      3  mongodb  5/edge   96  no       Replica set primary

Unit        Workload  Agent  Machine  Public address  Ports      Message
mongodb/0*  active    idle   0        10.23.62.156    27017/tcp  Replica set primary
mongodb/1   active    idle   1        10.23.62.55     27017/tcp  Replica set secondary
mongodb/2   active    idle   2        10.23.62.243    27017/tcp  Replica set secondary

Machine  State    Address       Inst id        Series  AZ  Message
0        started  10.23.62.156  juju-d35d30-0  jammy       Running
1        started  10.23.62.55   juju-d35d30-1  jammy       Running
2        started  10.23.62.243  juju-d35d30-2  jammy       Running
```

You can trust that Charmed MongoDB added these replicas correctly. But if you wanted to verify the replicas got added correctly you could connect to MongoDB via `charmed-mongodb.mongo`. Since your replica set has 2 additional hosts you will need to update the hosts in your URI. You can retrieve these host IPs with:
```shell
export HOST_IP_1=$(juju show-unit mongodb/1 | awk '/public-address:/{print $NF;exit}')
export HOST_IP_2=$(juju show-unit mongodb/2 | awk '/public-address:/{print $NF;exit}')
```

Then recreate the URI using your new hosts and reuse the `username`, `password`, `database name`, and `replica set name` that you previously used when you *first* connected to MongoDB:
```shell
export URI=mongodb://$DB_USERNAME:$DB_PASSWORD@$HOST_IP,$HOST_IP_1,$HOST_IP_2/$DB_NAME?replicaSet=$REPL_SET_NAME
```

Now view and save the output of the URI:
```shell
echo $URI
```

Like earlier we access `mongo` by `ssh`ing into one of the Charmed MongoDB hosts:
```shell
juju ssh mongodb/0
```

While `ssh`d into `mongodb/0`, we can access `mongo` with `charmed-mongodb.mongo`, using our new URI that we saved above.
```shell
charmed-mongodb.mongosh <saved URI>
```

Now, run `rs.status()` and you should see your replica set configuration. The first few lines should look something like this:
```json
{
  set: 'mongodb',
  date: ISODate("2022-12-02T14:39:52.732Z"),
  myState: 1,
  term: Long("1"),
  syncSourceHost: '',
  syncSourceId: -1,
  heartbeatIntervalMillis: Long("2000"),
  majorityVoteCount: 2,
  writeMajorityCount: 2,
  votingMembersCount: 3,
  writableVotingMembersCount: 3,
  optimes: {
    lastCommittedOpTime: { ts: Timestamp({ t: 1669991990, i: 1 }), t: Long("1") },
    lastCommittedWallTime: ISODate("2022-12-02T14:39:50.020Z"),
    readConcernMajorityOpTime: { ts: Timestamp({ t: 1669991990, i: 1 }), t: Long("1") },
    appliedOpTime: { ts: Timestamp({ t: 1669991990, i: 1 }), t: Long("1") },
    durableOpTime: { ts: Timestamp({ t: 1669991990, i: 1 }), t: Long("1") },
    lastAppliedWallTime: ISODate("2022-12-02T14:39:50.020Z"),
    lastDurableWallTime: ISODate("2022-12-02T14:39:50.020Z")
  },
  ...
```
### Return to original shell
Leave the MongoDB shell by typing `exit`. 
You will be back in the host of Charmed MongoDB (`mongodb/0`). Exit this host by typing `exit` again. 

You should now be at the original shell where you can interact with Juju and LXD.

<a href="#heading--remove-replicas"><h2 id="heading--remove-replicas">Remove replicas</h2></a>
Removing a unit from the application, scales the replicas down. Before we scale down the replicas, list all the units with `juju status`, here you will see three units `mongodb/0`, `mongodb/1`, and `mongodb/2`. Each of these units hosts a MongoDB replica. To remove the replica hosted on the unit `mongodb/2` enter:
```shell
juju remove-unit mongodb/2
```

You’ll know that the replica was successfully removed when `juju status --watch 1s` reports:
```
Model     Controller  Cloud/Region         Version  SLA          Timestamp
tutorial  overlord    localhost/localhost  3.1.6   unsupported  14:44:25Z

App      Version  Status  Scale  Charm    Channel   Rev  Exposed  Message
mongodb           active      2  mongodb  5/edge   96  no       Replica set primary

Unit        Workload  Agent  Machine  Public address  Ports      Message
mongodb/0*  active    idle   0        10.23.62.156    27017/tcp  Replica set primary
mongodb/1   active    idle   1        10.23.62.55     27017/tcp  Replica set secondary

Machine  State    Address       Inst id        Series  AZ  Message
0        started  10.23.62.156  juju-d35d30-0  jammy       Running
1        started  10.23.62.55   juju-d35d30-1  jammy       Running

```

You can also see that the replica was successfully removed by using the new URI (where the removed host has been excluded).

**Next step:** [5. Manage passwords](https://discourse.charmhub.io/t/charmed-mongodb-tutorial-manage-passwords/8630)