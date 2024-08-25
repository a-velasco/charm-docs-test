# Enable Security in your MongoDB deployment 

This is part of the [Charmed MongoDB K8s Tutorial](/t/charmed-mongodb-k8s-tutorial/10592). Please refer to this page for more information and the overview of the content. 

## Transport Layer Security (TLS)
[TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security) is used to encrypt data exchanged between two applications; it secures data transmitted over the network. Typically, enabling TLS within a highly available database, and between a highly available database and client/server applications, requires domain-specific knowledge and a high level of expertise. Fortunately, the domain-specific knowledge has been encoded into Charmed MongoDB K8S. This means enabling TLS on Charmed MongoDB K8S is readily available and requires minimal effort on your end.

Again, relations come in handy here as TLS is enabled via relations; i.e. by relating Charmed MongoDB K8s to the [Self Signed Certificates Charm](https://charmhub.io/self-signed-certificates). The Self Signed Certificates Charm centralises self-signed certificate management in a consistent manner and handles providing, requesting, and renewing self-signed TLS certificates.


*Note: Only for the tutorial sake we will use [self-signed certificates](https://en.wikipedia.org/wiki/Self-signed_certificate) provided by [self-signed-certificates-operator](https://github.com/canonical/self-signed-certificates-operator).  For production environments you should use [tls-certificates-operator](https://charmhub.io/tls-certificates-operator).*

### Configure TLS
Before enabling TLS on Charmed MongoDB K8s we must first deploy the `tls-certificates-operator` charm:
```shell
juju deploy self-signed-certificates --channel=beta
```

Wait until the `self-signed-certificates` is ready to be configured. When it is ready to be configured `watch -n 1 -c juju status`. Will show:
```shell
Model     Controller  Cloud/Region        Version  SLA          Timestamp
tutorial  overlord    microk8s/localhost  3.1.6    unsupported  04:40:45Z

App                       Version  Status  Scale  Charm                     Channel  Rev  Address         Exposed  Message
data-integrator                    active      1  data-integrator           edge      13  10.152.183.196  no       
mongodb-k8s                        active      2  mongodb-k8s               6/edge    37  10.152.183.194  no       Primary
self-signed-certificates           active      1  self-signed-certificates  beta      33  10.152.183.116  no 

Unit                         Workload  Agent  Address       Ports  Message
data-integrator/0*           active    idle   10.1.137.151             
mongodb-k8s/0*               active    idle   10.1.137.145         Primary
mongodb-k8s/1                active    idle   10.1.137.149         
self-signed-certificates/0*  active    idle   10.1.137.152         
```

### Enable TLS
After configuring the certificates `watch -n 1 juju status` will show the status of `self-signed-certificates` as active. To enable TLS on Charmed MongoDB K8s, relate the two applications:
```shell
juju relate mongodb-k8s self-signed-certificates
```

### Connect to MongoDB with TLS
Like before, generate and save the URI that is used to connect to MongoDB:
```
export URI=mongodb://$DB_USERNAME:$DB_PASSWORD@$HOST_IP,$HOST_IP_1:27017/$DB_NAME?replicaSet=$REPL_SET_NAME
echo $URI
```
Now ssh into `mongodb-k8s/0`:
```
juju ssh --container=mongod mongodb-k8s/0
```
After `ssh`ing into `mongodb-k8s/0`, we are now in the unit that is hosting Charmed MongoDB K8s. 
Once TLS has been enabled we will need to change how we connect to MongoDB. Specifically we will need to specify the TLS CA file along with the TLS Certificate file. These are on the units hosting the Charmed MongoDB K8S application in the folder `/etc/mongod`. If you enter: `ls /etc/mongod/external*` you should see the external certificate file and the external CA file:
```shell
/etc/mongod/external-ca.crt  /etc/mongod/external-cert.pem
```

As before, we will connect to MongoDB via the saved MongoDB URI. Connect using the saved URI and the following TLS options:
```shell
mongosh <URI> --tls --tlsCAFile /etc/mongod/external-ca.crt  --tlsCertificateKeyFile /etc/mongod/external-cert.pem
```
***Note: be sure you wrap the URI in `"` with no trailing whitespace*.**


Congratulations, you've now connected to MongoDB with TLS. Now exit the MongoDB shell by typing:
```shell
exit
```
Now you should be back in the host of the Charmed MongoDB K8s's unit: `mongodb-k8s/0`. To exit this host type:
```shell
exit
```
You should now be in the shell you started where you can interact with Juju and Kubernetes (Microk8s).

### Disable TLS
To disable TLS, unrelate the two applications:
```shell
juju remove-relation mongodb-k8s self-signed-certificates
```
## Next Steps
[Charmed MongoDB K8s Tutorial - Cleanup your environment](https://discourse.charmhub.io/t/charmed-mongodb-k8s-tutorial-environment-cleanup/10615)