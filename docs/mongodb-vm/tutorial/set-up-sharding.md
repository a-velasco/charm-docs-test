 [Charmed MongoDB Tutorials](/t/8061) > [**Deploy a sharded cluster**](/t/13290) > 1. Set up the environment

# Set up the environment

Charmed MongoDB is operated via Juju, a charm orchestration engine that makes them easy to operate. For this tutorial, we will set up the tools needed to deploy and use Charmed MongoDB.

## Summary
* [Minimum requirements](#heading--minimum-requirements) for your machine
* [Set up LXD](#heading--set-up-lxd)
* [Set up Juju](#heading--set-up-juju)

---
<a href="#heading--minimum-requirements"><h2 id="heading--minimum-requirements">Minimum requirements</h2></a>
Before we start, make sure your machine meets the following requirements:
- Ubuntu 20.04 (Focal) or later.
- 16GB of RAM.
- 4 CPU threads.
- At least 40GB of available storage.
- Access to the internet for downloading the required snaps and charms.

<a href="#heading--set-up-lxd"><h2 id="heading--set-up-lxd">Set up LXD</h2></a>

[LXD](https://documentation.ubuntu.com/lxd/en/latest/) is a system container and virtual machine manager. The simplest way to get started using the MongoDB charm is to set up a local LXD cloud, then running the charm in a LXD container.

### Install
Verify if your Ubuntu system already has LXD installed by entering the command `which lxd` into the command line. If there is no output, then simply install LXD with
```
sudo snap install lxd
```
### Configure
After installation, we need to run `lxd init` to perform post-installation tasks. For this tutorial, the default parameters are preferred and the network bridge should be set to have no IPv6 addresses, since Juju does not support IPv6 addresses with LXD:
```shell
lxd init --auto
lxc network set lxdbr0 ipv6.address none
```

You can list all LXD containers by entering the command `lxc list`. If you have not used any LXD containers before, the output will be blank at this stage in the tutorial:
```
+------+-------+------+------+------+-----------+
| NAME | STATE | IPV4 | IPV6 | TYPE | SNAPSHOTS |
+------+-------+------+------+------+-----------+
```

<a href="#heading--set-up-juju"><h2 id="heading--set-up-juju">Set up Juju</h2></a>
[Juju](https://juju.is/) is an Operator Lifecycle Manager (OLM) for clouds, bare metal, LXD or Kubernetes. We will be using it to deploy and manage Charmed MongoDB. 

### Install

As with LXD, Juju is installed from a snap package:
```shell
sudo snap install juju --channel 3.1/stable
```

Juju already has a built-in knowledge of LXD and how it works, so there is no additional setup or configuration needed, however,  because Juju 3.x is a a [strictly confined snap](https://snapcraft.io/docs/classic-confinement), and is not allowed to create ~/.local/share we need to create it manually.

```shell
mkdir -p ~/.local/share
```
### Bootstrap a controller
A controller will be used to deploy and control Charmed MongoDB. All we need to do is run the following command to bootstrap a Juju controller named ‘overlord’ to LXD. This bootstrapping processes can take several minutes depending on how provisioned (RAM, CPU, etc.) your machine is:
```shell
juju bootstrap localhost overlord --agent-version 3.1.7
```

The Juju controller should exist within an LXD container. You can verify this by entering the command `lxc list` and you should see the following:
```
+---------------+---------+-----------------------+------+-----------+-----------+
|     NAME      |  STATE  |         IPV4          | IPV6 |   TYPE    | SNAPSHOTS |
+---------------+---------+-----------------------+------+-----------+-----------+
| juju-<id>     | RUNNING | 10.105.164.235 (eth0) |      | CONTAINER | 0         |
+---------------+---------+-----------------------+------+-----------+-----------+
```
where `<id>` is a unique combination of numbers and letters such as `9d7e4e-0`

### Add a model
The controller can work with different models; models host applications such as Charmed MongoDB. 

Set up a specific model for Charmed MongoDB named `tutorial`:
```shell
juju add-model tutorial
```

You can now view the model you created above by entering the command `juju status` into the command line. You should see the following:
```
Model    Controller  Cloud/Region         Version  SLA          Timestamp
tutorial  overlord    localhost/localhost  3.1.7   unsupported  23:20:53Z

Model "admin/tutorial" is empty.
```
[note]
For the rest of this tutorial, we recommend keeping open a separate shell running `juju status --watch 1s --relations`. This way, you'll be able to see the way statuses and integrations change as you go along, without having to enter `juju status` all the time.
[/note]
**Next step:** [2. Deploy MongoDB](/t/13291)