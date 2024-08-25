> [Charmed MongoDB Tutorials](/t/8061) > [**Deploy a sharded cluster**](/t/13290) > 2. Deploy MongoDB


# Deploy MongoDB

Sharding is MongoDB's solution to horizontal scaling. [MongoDB's documentation](https://www.mongodb.com/basics/horizontal-vs-vertical-scaling#:~:text=The%20sharding%20method%20of%20horizontal,the%20shards%20across%20multiple%20machines.) describes horizontal scaling in databases as the process of “ dividing a large database into smaller, more manageable pieces (called shards) and then distributing the shards across multiple machines.”

In Charmed MongoDB, we create sharded clusters by deploying several applications of Charmed MongoDB as individual cluster components, and then integrating them. This part of the tutorial will guide you through a practical example of a sharded cluster deployment.

## Summary
* [Deploy cluster components](#heading--deploy-cluster-components), i.e. shards and  a config server
* [Integrate shards with the config server](#heading--integrate-shards)

--- 
<a href="#heading--deploy-cluster-components"><h2 id="heading--deploy-cluster-components">Deploy cluster components</h2></a>

The first step to creating your sharded cluster is to deploy cluster components. 

If you run `juju deploy mongodb` without specifying a role, the application will be automatically deployed as a replica set. **To deploy a sharded cluster, we must manually define the roles of the individual cluster components:**
```shell
juju deploy mongodb --config role="config-server" config-server
juju deploy mongodb --config role="shard" shard0
juju deploy mongodb --config role="shard" shard1
```
[note]
For a deeper explanation about the difference between replication and sharding see [Explanation | Replication vs. sharded cluster deployments](/t/13558#heading--comparison).
[/note]

Use `juju status --watch 1s` to wait until these components are ready. 

When the components are ready, your output should look similar to the example below:
```shell
Model           Controller  Cloud/Region         Version  SLA          Timestamp
sharding-model  overlord    localhost/localhost  3.1.7    unsupported  19:06:51+01:00

App            Version  Status   Scale  Charm    Channel    Rev  Exposed  Message    
config-server           blocked      1  mongodb  6/stable   158  no       missing relation to shard(s) 
shard0                  blocked      1  mongodb  6/stable   158  no       missing relation to config server
shard1                  blocked      1  mongodb  6/stable   158  no       missing relation to config server

Unit              Workload  Agent  Machine  Public address  Ports            Message    
config-server/0*  blocked   idle   0        10.56.148.138    27017-27018/tcp  missing relation to shard(s)        
shard0/0*         blocked   idle   1        10.56.148.149    27017/tcp        missing relation to config server
shard1/0*         blocked   idle   2        10.56.148.60     27017/tcp        missing relation to config server   

Machine  State    Address       Inst id        Base          AZ  Message
0        started  10.125.59.63  juju-573669-0  ubuntu@22.04      Running
1        started  10.125.59.57  juju-573669-1  ubuntu@22.04      Running
2        started  10.125.59.90  juju-573669-2  ubuntu@22.04      Running
```
All statuses are `blocked` because the shards are not yet integrated/related with the config-server.

<a href="#heading--integrate-shards"><h2 id="heading--integrate-shards">Integrate shards with the config server</h2></a>
Juju [integrations](https://juju.is/docs/juju/integration) are a way to connect applications such that they can communicate with one another through compatible endpoints.

To integrate the shards with the config server, run
```shell
juju integrate config-server:config-server shard0:sharding
juju integrate config-server:config-server shard1:sharding
```

<details>
<summary> You’ll know your cluster is active and ready when <code>juju status --watch 1s</code> shows all apps as <code>active</code>. </summary>

```shell
Model           Controller  Cloud/Region         Version  SLA          Timestamp
sharding-model  overlord    localhost/localhost  3.1.7    unsupported  19:15:36+01:00

App            Version  Status  Scale  Charm    Channel  Rev  Exposed  Message     
config-server           active      1  mongodb  6/beta   149  no       Primary                                     
shard0                  active      1  mongodb  6/beta   149  no       Primary                           
shard1                  active      1  mongodb  6/beta   149  no       Primary                           

Unit              Workload  Agent  Machine  Public address  Ports            Message       
config-server/0*  active    idle   0        10.56.148.138    27017-27018/tcp  Primary                                       
shard0/0*         active    idle   1        10.56.148.149    27017/tcp        Primary                             
shard1/0*         active    idle   2        10.56.148.60    27017/tcp        Primary                             

Machine  State    Address       Inst id        Base          AZ  Message
0        started  10.125.59.63  juju-573669-0  ubuntu@22.04      Running
1        started  10.125.59.57  juju-573669-1  ubuntu@22.04      Running
2        started  10.125.59.90  juju-573669-2  ubuntu@22.04      Running
```
</details>

**Next step:** [3. Access a sharded cluster](/t/13295)