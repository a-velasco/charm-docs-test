> [Charmed MongoDB Tutorials](/t/8061) > [**Deploy a replica set**](/t/8622) > 5. Manage passwords

# Manage passwords
When we connected to the `mongo` shell in [4. Access MongoDB](/t/13237), we had to include a password in the URI to get authenticated. Passwords help secure our database and are essential for the overall security of our stored data. It is good practice to change the password frequently.

This part of the tutorial will guide you through the essential password operations in Charmed MongoDB.

## Summary
* [Retrieve the admin password](#heading--retrieve-admin-password)
* [Rotate the admin password](#heading--rotate-admin-password)
* [Set the admin password](#heading--set-admin-password)
---

<a href="#heading--retrieve-admin-password"><h2 id="heading--retrieve-admin-password">Retrieve the admin password</h2></a>
The admin password can be retrieved by running the `get-password` action on the Charmed MongoDB application:
```shell
juju run mongodb/leader get-password
```
Running the command should output:
```yaml
Running operation 9 with 1 task
  - task 10 on unit-mongodb-0

Waiting for task 10...
password: 9JLjd0tuGngW5xFKWWbo0Blxyef0oGec
```
The admin password is under the result: `password`.

<a href="#heading--rotate-admin-password"><h2 id="heading--rotate-admin-password">Rotate the admin password</h2></a>
You can change the admin password to a new random password by entering:
```shell
juju run mongodb/leader set-password
```
Running the command should output:
```yaml
Running operation 11 with 1 task
  - task 12 on unit-mongodb-0

Waiting for task 12...
password: f3QkHE6QgmDLdRBonrt4vToWB7IZd4JO
```
The admin password is under the result: `password`. It should be different from your previous password.

*Note when you change the admin password you will also need to update the admin password the in MongoDB URI; as the old password will no longer be valid.* Update the DB password used in the URI and update the URI:
```shell
export DB_PASSWORD=$(juju run mongodb/leader get-password | grep password|  awk '{print $2}')
export URI=mongodb://$DB_USERNAME:$DB_PASSWORD@$HOST_IP/$DB_NAME?replicaSet=$REPL_SET_NAME
```

<a href="#heading--set-admin-password"><h2 id="heading--set-admin-password">Set the admin password</h2></a>
You can change the admin password to a specific password by entering:
```shell
juju run mongodb/leader set-password password=<password>
```
Running the command should output:
```yaml
Running operation 17 with 1 task
  - task 18 on unit-mongodb-0

Waiting for task 18...
password: <password>
```
The admin password under the result: `password` should match whatever you wrote when you entered the command.

*Note that when you change the admin password you will also need to update the admin password in the MongoDB URI, as the old password will no longer be valid.* To update the DB password used in the URI:
```shell
export DB_PASSWORD=$(juju run mongodb/leader get-password | grep password|  awk '{print $2}')
export URI=mongodb://$DB_USERNAME:$DB_PASSWORD@$HOST_IP/$DB_NAME?replicaSet=$REPL_SET_NAME
```

**Next step:** [6. Integrate with other applications](/t/8629)