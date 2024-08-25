# Charmed MongoDB Tutorials

This section of our documentation contains comprehensive, hands-on tutorials to help you learn how to deploy Charmed MongoDB and become familiar with its available operations. 

## Prerequisites
While the tutorials intend to guide you as you deploy Charmed MongoDB for the first time, they will be most beneficial if:
- You have some experience using a Linux-based CLI
- You are familiar with basic Juju concepts such as models and controllers
- You are familiar with MongoDB concepts such as users, replica sets, or sharding

## Tutorial contents

There are two tutorials available depending on the database storage method you'd like to learn about:
- [**Deploy a MongoDB replica set**](/t/8622)
- [**Deploy a MongoDB sharded cluster**](/t/13290)

Both tutorials will guide you through the same principles, while noting the details that apply specifically to replication or sharding.

| Step                                              | ...for a replica set                         | ...for a sharded cluster                      |
|---------------------------------------------------|----------------------------------------------|-----------------------------------------------|
| 1. Set up a cloud environment for your deployment | [Set up the environment](/t/8622)            | [Set up the environment](/t/13290)            |
| 2. Deploy Charmed MongoDB in a virtual container  | [Deploy MongoDB](/t/8621)                    | [Deploy MongoDB](/t/13291)                    |
| 3. Access a MongoDB database directly             | [Access a replica set](/t/13237)             | [Access a sharded cluster](/t/13295)          |
| 4. Scale the amount of replicas or shards         | [Scale your replicas](/t/8620)               | [Add and remove shards](/t/13302)             |
| 5. Request and change passwords                   | [Manage passwords](/t/8630)                  | [Manage passwords](/t/13303)                  |
| 6. Access MongoDB from a client application       | [Integrate with other applications](/t/8629) | [Integrate with other applications](/t/13348) |
| 7. Enable security in your deployment via TLS     | [Enable security](/t/8628)                   | [Enable security](/t/13349)                   |
| 8. Free up your machine's resources               | [Clean up the environment](/t/8627)          | [Clean up the environment](/t/8627)           |

[note]
**Tip:** If you're not sure which tutorial to go for, we recommend getting started with 
[**Deploy a MongoDB replica set**](/t/8622)
[/note]

## See also
* Our video tutorial: [Getting Started with Charmed MongoDB](https://youtu.be/dAGhfd6vwaE?feature=shared)