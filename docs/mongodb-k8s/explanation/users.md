# Internal users explanations

The operator uses the following internal DB users:

* `operator` - the user that charm.py uses to manage database/cluster.
* `monitor` - the user for [COS integration](https://charmhub.io/mongodb/docs/h-view-metrics).
* `backup` - the user to [perform/list/restore backups](https://charmhub.io/mongodb/docs/h-create-and-list-backups).

```{only} mongodb_k8s

   We're in K8s
```

```{only} mongodb_vm

   We're in vm
```

The full list of internal users is available in charm [source code](https://github.com/canonical/mongodb-k8s-operator/blob/main/lib/charms/mongodb/v0/users.py). The full dump of internal users (on the newly installed charm):

```none
ongodb-k8s [primary] admin> db.system.users.find()
```

```yaml
[
  {
    _id: 'admin.operator',
    userId: new UUID("f4466ca5-7640-4af3-b9eb-6b5a2313696a"),
    user: 'operator',
    db: 'admin',
    credentials: {
      'SCRAM-SHA-256': {
        iterationCount: 15000,
        salt: 'F6HeW8j0Vseza/r3anU6cHBy+Uiu1f8nmMn8TA==',
        storedKey: 'yHi6aWAs0blfXiePgh9nkzXw/Vr0JBhBUV0VmXysx+s=',
        serverKey: 'sJpXiPUrEEV9qlf8ZFng0zolN0Ii3MXKnta1mvaVTEI='
      }
    },
    roles: [
      { role: 'readWriteAnyDatabase', db: 'admin' },
      { role: 'clusterAdmin', db: 'admin' },
      { role: 'userAdminAnyDatabase', db: 'admin' }
    ]
  },
  {
    _id: 'admin.backup',
    userId: new UUID("ece48d5e-985a-4863-8660-8fa2e1a0aa7d"),
    user: 'backup',
    db: 'admin',
    credentials: {
      'SCRAM-SHA-256': {
        iterationCount: 15000,
        salt: 'QktAJGMGslnfpHg6p/Dd7eGlYb8uzBdBsCEiwg==',
        storedKey: '/abOk7NijI9u7rf8leRcnpBYImGshmGW7BsXwVpFYrQ=',
        serverKey: 'QWYXFJm1GhqrGjvqZTAtdKGitL+8B1LNIF3OmkiYyiU='
      }
    },
    roles: [
      { role: 'readWrite', db: 'admin' },
      { role: 'clusterMonitor', db: 'admin' },
      { role: 'backup', db: 'admin' },
      { role: 'pbmAnyAction', db: 'admin' },
      { role: 'restore', db: 'admin' }
    ]
  },
  {
    _id: 'admin.monitor',
    userId: new UUID("59560831-741d-4417-938a-68056bc80c47"),
    user: 'monitor',
    db: 'admin',
    credentials: {
      'SCRAM-SHA-256': {
        iterationCount: 15000,
        salt: 'QElyLpAw5PFmc4juu6Nk94Hv/4H1RxQfDgVEyg==',
        storedKey: 'bW2wC8DheiNkSMdMtZcq149qE9hKIErgcJVtRRghhqA=',
        serverKey: 'AU271CqwEMTFDorYRPVgyU1J5SvNLIVFWAItw19TvOg='
      }
    },
    roles: [
      { role: 'clusterMonitor', db: 'admin' },
      { role: 'explainRole', db: 'admin' },
      { role: 'read', db: 'local' }
    ]
  }
]
```
**Note**: it is forbidden to use/manage described above users! They are dedicated to the operators logic!
Please use [data-integrator](https://charmhub.io/postgresql/docs/t-integrations) charm to generate/manage/remove an external credentials.
