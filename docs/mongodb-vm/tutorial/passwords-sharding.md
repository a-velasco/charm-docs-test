 [Charmed MongoDB Tutorials](/t/8061) > [**Deploy a sharded cluster**](/t/13290) > 5. Manage passwords

# Manage passwords
When we accessed MongoDB earlier in this tutorial, we needed to include a password in the URI. Passwords help secure our database and are essential for security. Over time, it is a good practice to change the password frequently.

This part of the tutorial will guide you through the essential password operations in Charmed MongoDB.

## Summary
* [Retrieve the admin password](#heading--retrieve-admin-password)
* [Rotate the admin password](#heading--rotate-admin-password)
* [Set the admin password](#heading--set-admin-password)

---

<a href="#heading--retrieve-admin-password"><h2 id="heading--retrieve-admin-password">Retrieve the admin password</h2></a>
The `operator` password can be retrieved by running the `get-password` action on the Charmed MongoDB application. Password management in shared Charmed MongoDB clusters is handled exclusively by the `config-server`, so this is where we will perform all password-related operations. 

To retrieve the cluster password, run `the `get-password` action on the `config-server`:
```shell
juju run config-server/leader get-password
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
You can change the `operator` password to a new random password by running the `set-password` action on the `config-server`:
```shell
juju run config-server/leader set-password
```
Running the command should output:
```yaml
Running operation 11 with 1 task
  - task 12 on unit-config-server-0

Waiting for task 12...
password: f3QkHE6QgmDLdRBonrt4vToWB7IZd4JO
```
The admin password is under the result: `password`. It should be different from your previous password.

[note]
Remember that when you change the `operator` password, you will also need to update the `DB_PASSWORD` environment variable used in the MongoDB URI. 

To update the URI:
```shell
export DB_PASSWORD=$(juju run config-server/leader get-password | grep password|  awk '{print $2}')
export URI=mongodb://$DB_USERNAME:$DB_PASSWORD@localhost:27018/admin
echo $URI
```
[/note]

<a href="#heading--set-admin-password"><h2 id="heading--set-admin-password">Set the admin password</h2></a>
You can change the `operator` password to a specific password by running `set-password` on the `config-server` with your provided password:
```shell
juju run config-server/leader set-password password=<password>
```
Running the command should output:
```yaml
Running operation 17 with 1 task
  - task 18 on unit-config-server-0

Waiting for task 18...
password: <password>
```
The `operator` password under the result: `password` should match whatever you passed in when you entered the command.

[note]
Remember that when you change the `operator` password, you will also need to update the `DB_PASSWORD` environment variable used in the MongoDB URI. 

To update the URI:
```shell
export DB_PASSWORD=$(juju run config-server/leader get-password | grep password|  awk '{print $2}')
export URI=mongodb://$DB_USERNAME:$DB_PASSWORD@localhost:27018/admin
echo $URI
```
[/note]

**Next step:** [6. Integrate with other applications](/t/13348)