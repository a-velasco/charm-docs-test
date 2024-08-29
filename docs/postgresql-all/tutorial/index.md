(tutorial)=
# Tutorial

This section of our documentation contains comprehensive, hands-on tutorials to help you learn how to deploy Charmed PostgreSQL and become familiar with its available operations.

## Prerequisites

While this tutorial intends to guide you as you deploy Charmed PostgreSQL for the first time, it will be most beneficial if:
- You have some experience using a Linux-based CLI
- You are familiar with PostgreSQL concepts such as replication and users.
- Your computer fulfills the [minimum system requirements](/t/11743)

| Step | Details |
| ------- | ---------- |
| 1. {doc}`t-set-up` | Set up a cloud environment for your deployment using [Multipass](https://multipass.run/) with [LXD](https://ubuntu.com/lxd) and [Juju](https://juju.is/).
| 2. {doc}`t-deploy` | Learn to deploy Charmed PostgreSQL K8s using a single command and access the database directly.
| 4. {doc}`t-scale` | Learn how to enable high availability with a [Patroni](https://patroni.readthedocs.io/en/latest/)-based cluster.
| 5. {doc}`t-manage-passwords` | Learn how to request and change passwords.
| 6. {doc}`t-integrate` | Learn how to integrate with other applications using the Data Integrator Charm, access the integrated database, and manage users.
| 7. {doc}`t-enable-tls` | Learn how to enable security in your PostgreSQL deployment via TLS.
| 8. {doc}`t-clean-up`| Free up your machine's resources.

```{toctree}
:maxdepth: 2
:hidden:
t-set-up
t-deploy
t-scale
t-manage-passwords
t-integrate
t-enable-tls
t-clean-up
```