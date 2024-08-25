# How to view audit logs

[Audit logging](https://www.mongodb.com/docs/manual/core/auditing/) is enabled by default in Charmed MongoDB. You can view audit logs in two ways: by viewing `syslog` or by viewing them in Grafana.

## View logs using `syslog`
The audit log is sent to `syslog`. You can check it by performing the following steps:

First, `ssh` to the relevant unit. For example:
```shell
juju ssh mongodb/leader
``` 
In the unit's shell, run:
```shell
tail -f /var/log/syslog 
``` 
The console will now display audit log messages, for example:
```shell
Jan 26 13:22:56 juju-f6ba89-2 mongod: { "atype" : "updateOperation", "ts" : { "$date" : "2024-01-26T13:22:56.300+00:00" }, "local" : { "ip" : "10.55.47.232", "port" : 27017 }, "remote" : {}, "users" : [], "roles" : [], "param" : { "ns" : "local.replset.oplogTruncateAfterPoint", "doc" : { "_id" : "oplogTruncateAfterPoint", "oplogTruncateAfterPoint" : { "$timestamp" : { "t" : 1706275376, "i" : 1 } } } }, "result" : 0 }
```


## View logs with Grafana
Audit logs can be viewed with grafana using the [Canonical Observability Stack](https://charmhub.io/topics/canonical-observability-stack). 

To view these logs, you must first integrate Charmed MongoDB with the Canonical Observability Stack. Follow the instructions in [How-To > View metrics](https://charmhub.io/mongodb/docs/h-view-metrics) in the section titled “Accessing metrics with Grafana”

Once Grafana is active and you are able to access the GUI, navigate to the logging section. There you will be able to see your audit log messages.