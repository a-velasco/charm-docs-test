## Internal users

Charmed MongoDB has the following internal users:


| user       | function                                                   |
|------------|------------------------------------------------------|
| `operator` | Admin user that manages database/cluster (i.e. admin) |
| `monitor`  | Manages COS integration                             |
| `backup`   | Manages all backup operations                  |


The full list of internal users is available in charm [source code](https://github.com/canonical/mongodb-operator/blob/main/src/charm.py). 

Full dump of internal users on a newly installed Charmed MongoDB replica set:

```shell
mongodb:PRIMARY> db.system.users.find()

{ "_id" : "admin.operator", "userId" : UUID("40057ebb-1ab0-4534-afcf-1c01d0d23f7d"), "user" : "operator", "db" : "admin", "credentials" : { "SCRAM-SHA-256" : { "iterationCount" : 15000, "salt" : "JPTkW0Rpvm7ajlOw0dxs2icyWRhx+TC5VnthLQ==", "storedKey" : "A47RNOFNb/FZQvqnGRR4DLTMk43xdnmSZotZ33ehn/M=", "serverKey" : "weAbYuS8poCB8aaDeWy1l7iFLJH85ozN0HjW6I4gxOQ=" } }, "roles" : [ { "role" : "userAdminAnyDatabase", "db" : "admin" }, { "role" : "clusterAdmin", "db" : "admin" }, { "role" : "readWriteAnyDatabase", "db" : "admin" } ] }
{ "_id" : "admin.monitor", "userId" : UUID("c0e215a6-e2db-483e-a11b-244fac46d3e6"), "user" : "monitor", "db" : "admin", "credentials" : { "SCRAM-SHA-256" : { "iterationCount" : 15000, "salt" : "jOcAgMd7fc+62y2RrqBH5vr4UHPi7N8qJEGvIg==", "storedKey" : "Cx8ozkCp5pKG34wLpcrZxuz9H/hdHrBGdxoRUadDVSI=", "serverKey" : "C8plW72KIbn/VRDD9l8dJ18XKuevu80DVzGGIRhFRxw=" } }, "roles" : [ { "role" : "read", "db" : "local" }, { "role" : "explainRole", "db" : "admin" }, { "role" : "clusterMonitor", "db" : "admin" } ] }
{ "_id" : "admin.backup", "userId" : UUID("71816eb4-d51a-4437-b9d9-012fd66bf463"), "user" : "backup", "db" : "admin", "credentials" : { "SCRAM-SHA-256" : { "iterationCount" : 15000, "salt" : "SU/q8qPMjUlJSqzM1EmZkfFFusN4TsOa2M+E4Q==", "storedKey" : "Aw3t0gTW6A0E3+SlB6kK8UVQUXq1/rpUiCFo5oTeCAw=", "serverKey" : "xXRcVUNeiAgDGTMZU+BFexo7Y/yhpFRf7j8BjS4DFBI=" } }, "roles" : [ { "role" : "backup", "db" : "admin" }, { "role" : "pbmAnyAction", "db" : "admin" }, { "role" : "readWrite", "db" : "admin" }, { "role" : "clusterMonitor", "db" : "admin" }, { "role" : "restore", "db" : "admin" } ] }
```
[note type="caution"]
**Note**: These users are dedicated to the operator's logic, and **using them incorrectly could damage your deployment.**

Use the [data-integrator](https://github.com/canonical/data-integrator) charm to manage external credentials. To learn more about users and client connections, see [How To > Manage client connections](/t/8634)
[/note]