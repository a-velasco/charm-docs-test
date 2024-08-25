# Charm Testing reference

> **:information_source: Hint**: Use [Juju 3](/t/5064). (Charmed MongoDB dropped support for juju 2.9)

There are [a lot of test types](https://en.wikipedia.org/wiki/Software_testing) available and most of them are well applicable for Charmed MongoDB. Here is a list prepared by Canonical:

* Unit tests
* Integration tests
* Performance tests

## Unit tests:
Please check the "[Contributing](https://github.com/canonical/mongodb-operator/blob/main/CONTRIBUTING.md#testing)" guide and follow `tox run -e unit` examples there.

## Integration tests:
The integration tests coverage is rather rich in the MongoDB charm. 
Please check the "[Contributing](https://github.com/canonical/mongodb-operator/blob/main/CONTRIBUTING.md#testing)" guide and follow `tox run -e integration` examples there.

For tests related to complex features, and HA (High Availability) - each test serves as an integration as well as a smoke test with continuous writes routines being perpetually ran in parallel of whatever operation the test is involved in. 
These continuous writes ensure the availability of the service under different conditions. 

Those tests make use of the following fixture:
-  `continuous_writes`: creates a replicated collection and continuously stores data into it.

After each test completes, the collection gets deleted.

## Performance test
Refer to the [charmed MongoDB VM benchmark](https://discourse.charmhub.io/t/how-to-charmed-mongodb-performance-testing/13942) guide for charmed Charmed MongoDB.