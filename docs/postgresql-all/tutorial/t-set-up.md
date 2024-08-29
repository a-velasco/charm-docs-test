
# Set up the environment

In this step, we will set up a development environment with the required components for deploying Charmed PostgreSQL.

```{note}
Before you start, make sure your machine meets the [minimum system requirements](/t/11743).
```


## Set up Multipass

[Multipass](https://multipass.run/) is a quick and easy way to launch virtual machines running Ubuntu. It uses the [cloud-init](https://cloud-init.io/) standard to install and configure all the necessary parts automatically.

Install Multipass from the [snap store](https://snapcraft.io/multipass):
```none
sudo snap install multipass
```

Launch a new VM using the [charm-dev](https://github.com/canonical/multipass-blueprints/blob/main/v1/charm-dev.yaml) cloud-init config:
```none
multipass launch --cpus 4 --memory 8G --disk 50G --name my-vm charm-dev
```

```{note}
**Note**: All 'multipass launch' parameters are [described here](https://multipass.run/docs/launch-command).
```

The Multipass [list of commands](https://multipass.run/docs/multipass-cli-commands) is short and self-explanatory. For example, to show all running VMs, just run the command `multipass list`.

As soon as a new VM has started, access it using
```none
multipass shell my-vm
```

```{note}
**Note**:  If at any point you'd like to leave a Multipass VM, enter `Ctrl+D` or type `exit`.
```

All necessary components have been pre-installed inside VM already, like LXD and Juju. 

The files `/var/log/cloud-init.log` and `/var/log/cloud-init-output.log` contain all low-level installation details. 

## Set up Juju

````{tabs}
```{group-tab} VM
Let's bootstrap Juju to use the local LXD controller:

    juju bootstrap localhost lxd
```
```{group-tab} K8s
Let's bootstrap Juju to use the local MicroK8s controller:

    juju bootstrap localhost microk8s
```
````

A controller can work with different [models](https://juju.is/docs/juju/model). Set up a specific model for Charmed PostgreSQL named ‘tutorial’:
```none
juju add-model tutorial
```


You can now view the model you created above by running the command `juju status`.  You should see something similar to the following example output:

````{tabs}
```{group-tab} VM

    Model     Controller  Cloud/Region         Version  SLA          Timestamp
    tutorial  lxd         localhost/localhost  3.1.7    unsupported  09:38:32+01:00

    Model "admin/tutorial" is empty.

```
```{group-tab} K8s
    Model     Controller  Cloud/Region         Version  SLA          Timestamp
    tutorial  microk8s    microk8s/localhost   3.1.7    unsupported  09:38:32+01:00

    Model "admin/tutorial" is empty.

```
````

**Next step:** {doc}`t-deploy`