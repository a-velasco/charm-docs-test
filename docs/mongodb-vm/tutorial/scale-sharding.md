 [Charmed MongoDB Tutorials](/t/8061) > [**Deploy a sharded cluster**](/t/13290) > 4. Add and remove shards

# Add and remove shards

Shards spread data evenly throughout the cluster, making a database highly available. This part of the tutorial will teach you how to scale your cluster by adding and removing shards.

## Summary
* [Add shards](#heading--add-shards) to your MongoDB cluster  
* [Remove shards](#heading--remove-shards) from your MongoDB cluster

[note type="caution"]
**Disclaimer:** This tutorial hosts all shards on the same machine. **This should never be done in a production environment**. 

To enable high availability in a production environment, replicas should be hosted on different servers to [maintain isolation.](https://canonical.com/blog/database-high-availability)
[/note]

---

<a href="#heading--add-shards"><h2 id="heading--add-shards">Add shards</h2></a>
Sharded clusters cannot be scaled via juju the way replica sets can. So, before adding a shard to the cluster, we need to create it. 

To create a new shard, deploy a Charmed MongoDB application with the `shard` config option role:
```shell
juju deploy  mongodb --config role="shard" shard2
```

<details>
<summary>Use <code>juju status --watch 1s --relations</code> to wait until <code>shard2</code> is blocked due to missing relation.</summary>
    
```shell
Model     Controller  Cloud/Region         Version  SLA          Timestamp
tutorial  overlord    localhost/localhost  3.1.7    unsupported  15:15:53+01:00

App            Version  Status  Scale  Charm    Channel  Rev  Exposed  Message     
config-server           active      1  mongodb  6/beta   149  no       Primary     
shard0                  active      1  mongodb  6/beta   149  no       Primary     
shard1                  active      1  mongodb  6/beta   149  no       Primary     
shard2                  blocked     1  mongodb  6/beta   149  no       missing relation to config server                                  

Unit              Workload  Agent      Machine  Public address  Ports            Message   
config-server/0*  active    idle       6        10.17.247.150   27017-27018/tcp  Primary   
shard0/0*         active    idle       7        10.17.247.50    27017/tcp        Primary   
shard1/0*         active    idle       8        10.17.247.214   27017/tcp        Primary   
shard2/0*         blocked   executing  9        10.17.247.144                    missing relation to config server                                

Machine  State    Address        Inst id        Base          AZ  Message
6        started  10.17.247.150  juju-3acea1-6  ubuntu@22.04      Running
7        started  10.17.247.50   juju-3acea1-7  ubuntu@22.04      Running
8        started  10.17.247.214  juju-3acea1-8  ubuntu@22.04      Running
9        started  10.17.247.144  juju-3acea1-9  ubuntu@22.04      Running           

Integration provider          Requirer                      Interface      Type     Message
config-server:config-server   shard0:sharding               shards         regular
config-server:config-server   shard1:sharding               shards         regular
config-server:database-peers  config-server:database-peers  mongodb-peers  peer
shard0:database-peers         shard0:database-peers         mongodb-peers  peer
shard1:database-peers         shard1:database-peers         mongodb-peers  peer
shard2:database-peers         shard2:database-peers         mongodb-peers  peer 
```
</details>

Now that our shard is deployed, it is ready to be added to the cluster. For the shard to be added to the cluster it must be integrated with the `config-server`:
```shell
juju integrate config-server:config-server shard2:sharding
```
<details><summary>Use <code>juju status --watch 1s --relations</code> to watch the cluster until the new shard is active.</summary>
    
```shell
Model     Controller  Cloud/Region         Version  SLA          Timestamp
tutorial  overlord    localhost/localhost  3.1.7    unsupported  15:29:18+01:00

App            Version  Status  Scale  Charm    Channel  Rev  Exposed  Message     
config-server           active      1  mongodb  6/beta   149  no       Primary
shard0                  active      1  mongodb  6/beta   149  no       Primary     
shard1                  active      1  mongodb  6/beta   149  no       Primary     
shard2                  active      1  mongodb  6/beta   149  no       Primary                           

Unit              Workload  Agent  Machine  Public address  Ports            Message       
config-server/0*  active    idle   6        10.17.247.150   27017-27018/tcp  Primary                                    
shard0/0*         active    idle   7        10.17.247.50    27017/tcp        Primary       
shard1/0*         active    idle   8        10.17.247.214   27017/tcp        Primary       
shard2/0*         active    idle   9        10.17.247.144   27017/tcp        Primary                                   

Machine  State    Address        Inst id        Base          AZ  Message
6        started  10.17.247.150  juju-3acea1-6  ubuntu@22.04      Running
7        started  10.17.247.50   juju-3acea1-7  ubuntu@22.04      Running
8        started  10.17.247.214  juju-3acea1-8  ubuntu@22.04      Running
9        started  10.17.247.144  juju-3acea1-9  ubuntu@22.04      Running           

Integration provider          Requirer                      Interface      Type     Message
config-server:config-server   shard0:sharding               shards         regular
config-server:config-server   shard1:sharding               shards         regular
config-server:config-server   shard2:sharding               shards         regular
config-server:database-peers  config-server:database-peers  mongodb-peers  peer
shard0:database-peers         shard0:database-peers         mongodb-peers  peer
shard1:database-peers         shard1:database-peers         mongodb-peers  peer            
shard2:database-peers         shard2:database-peers         mongodb-peers  peer
 
```
</details>

To verify that the shard is present in the cluster, we can check the cluster configuration through the `mongos` router inside `config-server` and listing shards like we did in [4. Access a sharded cluster | Connect via MongoDB URI]().

To summarize:
* `echo $URI` to get your URI
* `juju ssh config-server/0` to enter the config-server
* `charmed-mongodb.mongosh <your-URI>` to enter the mongodb shell
* `sh.status()`
    
This will confirm that shard `shard2` is in the cluster configuration:
```yaml
shards
[
  {
    _id: 'shard0',
    host: 'shard0/10.17.247.50:27017',
    state: 1,
    topologyTime: Timestamp({ t: 1708523939, i: 7 })
  },
  {
    _id: 'shard1',
    host: 'shard1/10.17.247.214:27017',
    state: 1,
    topologyTime: Timestamp({ t: 1708523945, i: 5 })
  },
  {
    _id: 'shard2',
    host: 'shard2/10.17.247.144:27017',
    state: 1,
    topologyTime: Timestamp({ t: 1708525386, i: 4 })
  }
]

...
```
When you’re ready to leave the MongoDB shell, just type `exit`. You will be back in the host of Charmed MongoDB (`mongodb/0`).

Exit this host by once again typing `exit`. Now you will be in your original shell where you can interact with Juju and LXD.

<a href="#heading--remove-shards"><h2 id="heading--remove-shards">Remove shards</h2></a>
To remove a shard from Charmed MongoDB, we remove the integration from the shard and the config-server. This signals to the cluster that it is time to drain the shard and remove it from the config. 

First, break the integration:
```shell
juju remove-relation config-server:config-server shard2:sharding
```
<details>
<summary>Watch <code>juju status --watch 1s --relations</code> until <code>shard2</code> has finished being drained from the cluster and is no longer listed as a requirer of the <code>config-server</code> integration provider. </summary>

```shell
Model     Controller  Cloud/Region         Version  SLA          Timestamp
tutorial  overlord    localhost/localhost  3.1.7    unsupported  15:52:10+01:00

App            Version  Status  Scale  Charm    Channel  Rev  Exposed  Message     
config-server           active      1  mongodb  6/beta   149  no       Primary                                  
shard0                  active      1  mongodb  6/beta   149  no       Primary     
shard1                  active      1  mongodb  6/beta   149  no       Primary     
shard2                  active      1  mongodb  6/beta   149  no       Primary                                      

Unit              Workload  Agent  Machine  Public address  Ports            Message       
config-server/0*  active    idle   6        10.17.247.150   27017-27018/tcp  Primary                                    
shard0/0*         active    idle   7        10.17.247.50    27017/tcp        Primary       
shard1/0*         active    idle   8        10.17.247.214   27017/tcp        Primary       
shard2/0*         active    idle   9        10.17.247.144   27017/tcp        Primary                                          

Machine  State    Address        Inst id        Base          AZ  Message
6        started  10.17.247.150  juju-3acea1-6  ubuntu@22.04      Running
7        started  10.17.247.50   juju-3acea1-7  ubuntu@22.04      Running
8        started  10.17.247.214  juju-3acea1-8  ubuntu@22.04      Running
9        started  10.17.247.144  juju-3acea1-9  ubuntu@22.04      Running           

Integration provider          Requirer                      Interface      Type     Message
config-server:config-server   shard0:sharding               shards         regular
config-server:config-server   shard1:sharding               shards         regular
config-server:database-peers  config-server:database-peers  mongodb-peers  peer   
shard0:database-peers         shard0:database-peers         mongodb-peers  peer
shard1:database-peers         shard1:database-peers         mongodb-peers  peer
shard2:database-peers         shard2:database-peers         mongodb-peers  peer        
```
</details>

Now that the shard has been drained and removed from the configuration, it is safe to remove it altogether. To do so, run:
```shell
juju remove-application shard2
```
Watch the cluster with `juju status --watch 1s --relations` until `shard2` is no longer listed anywhere.

**Next Step:** [5. Manage passwords](/t/13303)