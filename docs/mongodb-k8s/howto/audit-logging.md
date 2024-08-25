## Viewing Audit Logs

[Audit logging](https://www.mongodb.com/docs/manual/core/auditing/) is enabled by default in Charmed MongoDB and you can view audit logs in two ways: by viewing audit log file inside `mongod` container  or by viewing them in grafana.



### Viewing audit log file inside `mongod` container

At the current moment, the audit log is being written to `/var/lib/mongodb/audit.log` so, can check it by performing the following steps:

1. ssh to mongod container of a unit

```
juju ssh --container=mongod mongodb-k8s/leader
```

2 inside container execute

```
tail -f /var/lib/mongodb/audit.log
```

in the console you will see audit log messages, for example

```
Jan 26 13:22:56 juju-f6ba89-2 mongod: { "atype" : "updateOperation", "ts" : { "$date" : "2024-01-26T13:22:56.300+00:00" }, "local" : { "ip" : "10.55.47.232", "port" : 27017 }, "remote" : {}, "users" : [], "roles" : [], "param" : { "ns" : "local.replset.oplogTruncateAfterPoint", "doc" : { "_id" : "oplogTruncateAfterPoint", "oplogTruncateAfterPoint" : { "$timestamp" : { "t" : 1706275376, "i" : 1 } } } }, "result" : 0 }
```


### Accessing audit logs with grafana

Audit logs can be viewed with grafana using the [Canonical Observability Stack](https://charmhub.io/topics/canonical-observability-stack). To view these logs:

1. First integrate Charmed MongoDB with the Canonical Observability Stack. To do this follow the instructions on the How-To [View Metrics](https://discourse.charmhub.io/t/view-metrics/10650) in the section titled “Accessing metrics with grafana”
2. Once grafana is active and you are able to access the GUI, navigate to the logging section and you will you will be able to see your audit log messages