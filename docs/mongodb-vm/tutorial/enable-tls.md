> [Charmed MongoDB Tutorials](/t/8061) > [**Deploy a replica set**](/t/8622) > 7. Enable security
# Enable security in your MongoDB deployment 

[Transport Layer Security (TLS)](https://en.wikipedia.org/wiki/Transport_Layer_Security) is a protocol used to encrypt data exchanged between two applications. Essentially, it secures data transmitted over a network.

Typically, enabling TLS internally within a highly available database or between a highly available database and client/server applications requires a high level of expertise. This has all been encoded into Charmed MongoDB so that configuring TLS requires minimal effort on your end.

TLS is enabled by integrating Charmed MongoDB with the [Self Signed Certificates Charm](https://charmhub.io/self-signed-certificates). This charm centralises TLS certificate management consistently and handles operations like providing, requesting, and renewing TLS certificates.

In this section, you will learn how to enable security in your MongoDB deployment using TLS encryption.

[note type="caution"]
**Disclaimer:** In this tutorial we  use [self-signed certificates](https://en.wikipedia.org/wiki/Self-signed_certificate) provided by the [`self-signed-certificates-operator`](https://github.com/canonical/self-signed-certificates-operator).  

**This is not recommended for a production environment.**

For production environments, check the collection of [Charmhub operators](https://charmhub.io/?q=tls-certificates) that implement the `tls-certificate` interface, and choose the most suitable for your use-case.
[/note]

## Summary
* [Configure TLS](#heading--configure-tls)
* [Enable TLS](#heading--enable-tls)
* [Connect to MongoDB with TLS](#heading--connect-with-tls)
* [Disable TLS](#heading--disable-tls)

---

<a href="#heading--configure-tls"><h2 id="heading--configure-tls">Configure TLS</h2></a>
Before enabling TLS on Charmed MongoDB we must first deploy the `self-signed-certificates` charm:
```shell
juju deploy self-signed-certificates
```

Wait until the `self-signed-certificates` app is `active` with `juju status --watch 1s`, like in the output below.
```
Model              Controller  Cloud/Region         Version  SLA          Timestamp
tutorial  overlord    localhost/localhost  3.4.0    unsupported  09:35:14+01:00

App                       Version  Status  Scale  Charm                     Channel  Rev  Exposed  Message
mongodb                            active      1  mongodb                   6/beta   149  no       Primary
self-signed-certificates           active      1  self-signed-certificates  stable    72  no

Unit                         Workload  Agent  Machine  Public address  Ports      Message
mongodb/0*                   active    idle   0        10.67.56.90     27017/tcp  Primary
self-signed-certificates/1*  active    idle   2        10.67.56.137    

Machine  State    Address       Inst id        Base          AZ  Message
0        started  10.67.56.90   juju-fab81d-0  ubuntu@22.04      Running
2        started  10.67.56.137  juju-fab81d-2  ubuntu@22.04      Running
```

Now that `self-signed-certificates` has finished deploying, we can configure it with:
```shell
juju config self-signed-certificates ca-common-name="Tutorial CA" 
```

<a href="#heading--enable-tls"><h2 id="heading--enable-tls">Enable TLS</h2></a>
To enable TLS on Charmed MongoDB, integrate the two applications:
```shell q q
juju integrate self-signed-certificates mongodb
```

<a href="#heading--connect-with-tls"><h2 id="heading--connect-with-tls">Connect to MongoDB with TLS</h2></a>
Like before, generate and save the URI that is used to connect to MongoDB:
```
export URI=mongodb://$DB_USERNAME:$DB_PASSWORD@$HOST_IP/$DB_NAME?replicaSet=$REPL_SET_NAME
echo $URI
```
Now ssh into `mongodb/0`:
```
juju ssh mongodb/0
```
We are now in the unit that is hosting Charmed MongoDB. 

Once TLS has been enabled, we will need to change how we connect to MongoDB. We will need to specify the TLS CA file along with the TLS Certificate file that were automatically created when we integrated the two charms.

You will find these files on the units hosting the Charmed MongoDB application in the folder `/var/snap/charmed-mongodb/common/etc/mongod`. 

If you enter: 
```shell
ls /var/snap/charmed-mongodb/current/etc/mongod/external*
``` 
you should see the following external certificate file and external CA files:

```shell
/var/snap/charmed-mongodb/current/etc/mongod/external-ca.crt  
/var/snap/charmed-mongodb/current/etc/mongod/external-cert.pem
```

As before, we will connect to MongoDB via the saved MongoDB URI. Connect using the saved URI and the following TLS options:
```shell
sudo charmed-mongodb.mongosh mongodb://$DB_USERNAME:$DB_PASSWORD@$HOST_IP/$DB_NAME?replicaSet=$REPL_SET_NAME --tls --tlsCAFile /var/snap/charmed-mongodb/current/etc/mongod/external-ca.crt  --tlsCertificateKeyFile /var/snap/charmed-mongodb/current/etc/mongod/external-cert.pem
```

You have successfully connected to MongoDB with TLS! 

When you are ready, leave the MongoDB shell by typing `exit`. You will be back in the host of Charmed MongoDB (`mongodb/0`). Exit this host by typing `exit` again.

You should now be at the original shell where you can interact with Juju and LXD.

<a href="#heading--disable-tls"><h2 id="heading--disable-tls">Disable TLS</h2></a>
To disable TLS, simply remove the integration between the two applications:
```shell
juju remove-relation mongodb self-signed-certificates
```

**Next step:** [8. Clean up the environment](/t/8627)