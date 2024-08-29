# Clean up your environment

In this tutorial we've successfully deployed PostgreSQL, added and removed cluster members, added and removed database users, and enabled a layer of security with TLS.

You may now keep your Charmed PostgreSQLdeployment running and write to the database or remove it entirely using the steps in this page. 


## Stop your virtual machine
If you'd like to keep your environment for later, stop your Multipass VM with the following command:
```shell
multipass stop my-vm
```

## Delete your virtual machine
If you're done with testing and would like to free up resources on your machine, you can remove the VM entirely.

```{important}
When you remove VM as shown below, **you will lose all the data in PostgreSQL and any other applications inside Multipass VM**! 

For more information, see the documentation for [`multipass delete`](https://multipass.run/docs/delete-command).
```

*Delete your VM and its data with the following command:
```shell
multipass delete --purge my-vm
```

## Next Steps
 If you're looking for what to do next, you can:
- Check out our Charmed offerings of [MySQL](https://charmhub.io/mysql) and [Kafka](https://charmhub.io/kafka?channel=edge).
- Read about [High Availability Best Practices](https://canonical.com/blog/database-high-availability)
- [Report](https://github.com/canonical/postgresql-operator/issues) any problems you encountered.
- [Give us your feedback](https://chat.charmhub.io/charmhub/channels/data-platform).
- [Contribute to the code base](https://github.com/canonical/postgresql-operator)