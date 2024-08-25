> [Charmed MongoDB Tutorials](/t/8061) > [**Deploy a sharded cluster**](/t/13290) > 7. Enable security
# Enable security in your MongoDB deployment 

[Transport Layer Security (TLS)](https://en.wikipedia.org/wiki/Transport_Layer_Security) is a protocol used to encrypt data exchanged between two applications. Essentially, it secures data transmitted over a network.

Typically, enabling TLS internally within a highly available database or between a highly available database and client/server applications, requires domain-specific knowledge and a high level of expertise. This has all been encoded into Charmed MongoDB. This means (re-)configuring TLS on Charmed MongoDB is readily available and requires minimal effort on your end.

TLS is enabled by relating Charmed MongoDB to the [Self Signed Certificates Charm](https://charmhub.io/self-signed-certificates). This charm centralises TLS certificate management consistently and handles operations like providing, requesting, and renewing TLS certificates.

In this part of the tutorial, you will learn how to enable security in your MongoDB deployment using TLS encryption.

[note type="caution"]
**Disclaimer:** In this tutorial, we use [self-signed certificates](https://en.wikipedia.org/wiki/Self-signed_certificate) provided by the [`self-signed-certificates-operator`](https://github.com/canonical/self-signed-certificates-operator).

**This is not recommended for a production environment.**

For production environments, check the collection of [Charmhub operators](https://charmhub.io/?q=tls-certificates) that implement the `tls-certificate` interface, and choose the most suitable for your use-case.
[/note]

---

To enable encryption via TLS in a sharded cluster, we will first set up the `self-signed-certificates` certificate provider.

**Deploy the `self-signed-certificates` charm** as follows:
```shell
juju deploy self-signed-certificates --config ca-common-name="Tutorial CA"
```

Wait until the `self-signed-certificates` app is `active` with `juju status --watch 1s`.

Then, **integrate the certificates provider with all cluster components**:
```shell
juju integrate config-server self-signed-certificates
juju integrate shard0 self-signed-certificates
juju integrate shard1 self-signed-certificates
```

Your replica set now has encryption enabled via TLS.

<!--- <a href="#heading--disable-tls"><h2 id="heading--disable-tls">Disable TLS</h2></a>

-->

**Next step:** [8. Clean up the environment](/t/8627)