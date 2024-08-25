> [Charmed MongoDB Tutorials](/t/8061) > [**Deploy a replica set**](/t/8622) > 2. Deploy MongoDB
# Deploy MongoDB
Deploying a Charmed MongoDB replica set is a quite straightforward operation with `juju`. In this page, you will learn how to deploy and track the charm's status as `juju` set it up in the background 

## Summary
* [Deploy a replica set](#heading--deploy-replica-set)
* [Track deployment status](#heading--track-deployment)
---

<a href="#heading--deploy-replica-set"><h2 id="heading--deploy-replica-set">Deploy a replica set</h2></a>

Deploy MongoDB with the following command:
```shell
juju deploy mongodb
```
Juju  will fetch the charm from [Charmhub](https://charmhub.io/mongodb?channel=6/beta) and begin deploying it to the LXD cloud. This process can take several minutes depending on how provisioned (RAM, CPU, etc) your machine is. 

<a href="#heading--track-deployment"><h2 id="heading--track-deployment">Track deployment status</h2></a>
You can check the status of your deployment by running:
```shell
juju status --watch 1s --relations
```
This will display a table with an overview of all the status info of the elements in your juju model, like IP addresses, ports, state, and other useful data. The `--watch 1s` flag means that it will update every 1s. 

For this tutorial, it is recommended to have a separate terminal permanently set to `juju status --watch 1s --relations` so that you can see what is happening every time you make changes to your juju model.  The `--relations` flag will display additional information regarding integrations (previously known as "relations"), which will be useful later on.

When your MongoDB application is ready, `juju status --watch 1s` will show:
```
Model     Controller  Cloud/Region         Version  SLA          Timestamp
tutorial  overlord    localhost/localhost  3.4.0    unsupported  11:24:30Z

App      Version  Status  Scale  Charm    Channel   Rev  Exposed  Message
mongodb           active      1  mongodb  6/stable  158  no

Unit        Workload  Agent  Machine  Public address  Ports      Message
mongodb/0*  active    idle   0        10.23.62.156    27017/tcp

Machine  State    Address       Inst id        Series  AZ  Message
0        started  10.23.62.156  juju-d35d30-0  jammy       Running
```
To exit the screen with `juju status --watch 1s`, enter `Ctrl + C`.  

**Next step:** [3. Access MongoDB](/t/13237)