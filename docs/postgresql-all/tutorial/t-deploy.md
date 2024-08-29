---
myst:
  substitutions:
    vm:
      charmhub_url: '[charmhub.io/postgresql](https://charmhub.io/postgresql)'
    k8s:
      charmhub_url: '[charmhub.io/postgresql-k8s](https://charmhub.io/postgresql-k8s)'
---
# Deploy Charmed PostgreSQL

In this section, you will deploy Charmed PostgreSQL VM, access a unit, and interact with the PostgreSQL databases that exist inside the application.


## Deploy

::::{tab-set}
:::{tab-item} VM
:sync: vm
To deploy Charmed PostgreSQL on machines, run

    juju deploy postgresql

Juju will now fetch Charmed PostgreSQL from {{vm.charmhub_url}} and deploy it to the LXD cloud.
:::

:::{tab-item} K8s
:sync: k8s
To deploy Charmed PostgreSQL on Kubernetes, run

    juju deploy postgresql-k8s --trust
> The `--trust` flag is required because the charm and Patroni need to create some K8s resources.

Juju will now fetch Charmed PostgreSQL from {{k8s.charmhub_url}} and deploy it to MicroK8s. 
:::
::::

This process can take several minutes depending on how provisioned (RAM, CPU, etc) your machine is. 

You can track the progress by running:
```none
juju status --watch 1s
```

> This command is useful for checking the real-time information about the state of a charm and the machines hosting it. Check the [`juju status` documentation](https://juju.is/docs/juju/juju-status) for more information about its usage.

When the application is ready, `juju status` will show something similar to the sample output below:

::::{tab-set}
:::{tab-item} VM
:sync: vm
    Model     Controller  Cloud/Region         Version  SLA          Timestamp
    tutorial  lxd         localhost/localhost  3.1.7    unsupported  09:41:53+01:00

    App         Version  Status  Scale  Charm       Channel    Rev  Exposed  Message
    postgresql           active      1  postgresql  14/stable  281  no       

    Unit           Workload  Agent  Machine  Public address  Ports  Message
    postgresql/0*  active    idle   0        10.89.49.129           

    Machine  State    Address       Inst id        Series  AZ  Message
    0        started  10.89.49.129  juju-a8a31d-0  jammy       Running
:::

:::{tab-item} K8s
:sync: k8s
    Model     Controller  Cloud/Region         Version  SLA          Timestamp
    tutorial  microk8s    microk8s/localhost   3.1.7    unsupported  09:41:53+01:00

    App         Version  Status  Scale  Charm           Channel    Rev  Exposed  Message
    postgresql-k8s       active      1  postgresql-k8s  14/stable  281  no       

    Unit               Workload  Agent  Machine  Public address  Ports  Message
    postgresql-k8s/0*  active    idle   0        10.89.49.129           

    Machine  State    Address       Inst id        Series  AZ  Message
    0        started  10.89.49.129  juju-a8a31d-0  jammy       Running
:::
::::

You can also watch juju logs with the [`juju debug-log`](https://juju.is/docs/juju/juju-debug-log) command.

## Access PostgreSQL

```{caution}
This part of the tutorial accesses PostgreSQL via the `operator` user. 

**Do not directly interface with the `operator` user in a production environment.**

In a later section {doc}`t-integrate` we will cover how to safely access PostgreSQL by creating a separate user.
```

### Retrieve credentials

Connecting to the database requires that you know the values for `host`, `username` and `password`. 

To retrieve these values, run the Charmed PostgreSQL action `get-password`:

::::{tab-set}
:::{tab-item} VM
:sync: vm
```none
juju run postgresql/leader get-password
```
Example output:
```yaml
unit-postgresql-0:
UnitId: postgresql/0
id: "2"
results:
  operator-password: <password>
status: completed
timing:
  completed: 2023-03-20 08:42:22 +0000 UTC
  enqueued: 2023-03-20 08:42:19 +0000 UTC
  started: 2023-03-20 08:42:21 +0000 UTC
```

To request a password for a different user, use the option `username`:
```none
juju run postgresql/leader get-password username=replication
```
:::
:::{tab-item} K8s
:sync: k8s
```none
juju run postgresql-k8s/leader get-password
```
Example output:
```yaml
unit-postgresql-k8s-0:
UnitId: postgresql-k8s/0
id: "2"
results:
  operator-password: <password>
status: completed
timing:
  completed: 2023-03-20 08:42:22 +0000 UTC
  enqueued: 2023-03-20 08:42:19 +0000 UTC
  started: 2023-03-20 08:42:21 +0000 UTC
```

To request a password for a different user, use the option `username`:
```none
juju run postgresql-k8s/leader get-password username=replication
```
:::
::::


The IP address of the unit hosting the PostgreSQL application, also referred to as the "host", can be found with `juju status`:

::::{tab-set}
:::{tab-item} VM
:sync: vm

    ...
    Unit           Workload  Agent  Machine  Public address  Ports  Message
    postgresql/0*  active    idle   0        10.89.49.129       
    ...
:::
:::{tab-item} K8s
:sync: k8s

    ...
    Unit               Workload  Agent  Machine  Public address  Ports  Message
    postgresql-k8s/0*  active    idle   0        10.89.49.129       
    ...

:::
::::


### Access PostgreSQL via `psql`

To access the units hosting Charmed PostgreSQL, run
::::{tab-set}
:::{tab-item} VM
:sync: vm
    juju ssh postgresql/leader
:::
:::{tab-item} K8s
:sync: k8s
    juju ssh postgresql-k8s/leader
:::
::::

>If at any point you'd like to leave the unit hosting Charmed PostgreSQL, enter `Ctrl+D` or type `exit`.

The easiest way to access PostgreSQL is via the [PostgreSQL interactive terminal `psql`](https://www.postgresql.org/docs/14/app-psql.html), which is already installed here. 

To list all available databases, run:
```none
psql --host=10.89.49.129 --username=operator --password --list
```
When requested, enter the `<password>` for charm user `operator` that you obtained earlier.

Example output:
```none
                              List of databases
   Name    |  Owner   | Encoding | Collate |  Ctype  |   Access privileges   
-----------+----------+----------+---------+---------+-----------------------
 postgres  | operator | UTF8     | C.UTF-8 | C.UTF-8 | 
 template0 | operator | UTF8     | C.UTF-8 | C.UTF-8 | =c/operator          +
           |          |          |         |         | operator=CTc/operator
 template1 | operator | UTF8     | C.UTF-8 | C.UTF-8 | =c/operator          +
           |          |          |         |         | operator=CTc/operator
(3 rows)
```

You can now interact with PostgreSQL directly using [PostgreSQL SQL Queries](https://www.postgresql.org/docs/14/queries.html). For example, entering `SELECT version();` should output something like:
```none
> psql --host=10.89.49.129 --username=operator --password postgres
Password: 
psql (14.7 (Ubuntu 14.7-0ubuntu0.22.04.1))
Type "help" for help.

postgres=# SELECT version();
                                                               version                                                                
--------------------------------------------------------------------------------------------------------------------------------------
 PostgreSQL 14.7 (Ubuntu 14.7-0ubuntu0.22.04.1) on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 11.3.0-1ubuntu1~22.04) 11.3.0, 64-bit
(1 row)
```

Feel free to test out any other PostgreSQL queries. 

When you’re ready to leave the PostgreSQL shell, you can just type `exit`. This will take you back to the host unit of Charmed PostgreSQL. Exit this host by once again typing `exit`. Now you will be in your original shell where you first started the tutorial. Here you can interact with Juju and LXD.