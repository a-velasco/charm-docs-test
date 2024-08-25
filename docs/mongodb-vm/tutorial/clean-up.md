> This page is from the [Charmed MongoDB Tutorials](/t/8061)
# Clean up the environment

In this tutorial, we’ve successfully:
* Deployed MongoDB on LXD
* Scaled our deployment 
* Added and removed database users
* Enabled and disabled TLS

You may now keep your MongoDB deployment running to continue experimenting, or remove it entirely to free up resources on your machine.

<!-- TODO: if using multipass, change this section to just remove mp vm -->
## Remove Charmed MongoDB 
**Warning**: If you remove Charmed MongoDB as shown below, you will lose all data stored in MongoDB. 

To remove Charmed MongoDB, delete the juju model it is hosted on:
```shell
juju destroy-model tutorial --destroy-storage --force
```
Then, remove the `overlord` Juju controller:
```shell
juju destroy-controller overlord
```

## Remove Juju
**Warning:** If you remove Juju as shown below, you will lose access to any other applications you have hosted on Juju.

To remove Juju altogether, enter:
```shell
sudo snap remove juju --purge
```

You've successfully completed our Charmed MongoDB tutorial :tada: 

---
## What next?
- Check out our other [Charmed MongoDB Tutorials](/t/8061)
- Run [Charmed MongoDB on Kubernetes](https://github.com/canonical/mongodb-k8s-operator).
- Look into some of our other charms, like [PostgreSQL](https://charmhub.io/postgresql?channel=edge) and [Kafka](https://charmhub.io/kafka?channel=edge).
- Read about [High Availability Best Practices](https://canonical.com/blog/database-high-availability)

## Contact us!
- Meet the community and chat with us on [Matrix](https://matrix.to/#/#charmhub-data-platform:ubuntu.com)
- [Report](https://github.com/canonical/mongodb-operator/issues) any problems you encountered.
- [Contribute to the code base](https://github.com/canonical/mongodb-operator)
- [Contribute to the documentation](https://discourse.charmhub.io)