[![Build Status](https://travis-ci.org/debezium/debezium.svg?branch=master)](https://travis-ci.org/debezium/debezium)
[![License](http://img.shields.io/:license-apache-brightgreen.svg)](http://www.apache.org/licenses/LICENSE-2.0.html)

Copyright 2008-2016 Debezium Authors. Licensed under the [Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0).

# Debezium

Debezium is an open source project that provides a near real-time data streaming platform for change data capture (CDC). You setup and configure Debezium to monitor your databases, and then your application uses Debezium libraries to receive detailed events for each row-level change made to the database. Only committed changes are visible, so your application doesn't have to worry about transactions or changes that are rolled back. Debezium provides a single model of all change events, so your application does not have to worry about the intricacies of each kind of database management system. Additionally, since Debezium records the history of data changes in its logs, your application can be stopped and restarted at any time, and it will be able to consume all of the events it missed while it was not running, ensuring that all events are processed correctly and completely.

Monitoring databases and being notified when data changes has always been complicated. Relational database triggers can be useful, but are specific to each database and often limited to updating state within the same database (not communicating with external processes). Some databases offer APIs or frameworks for monitoring changes, but there is no standard so each database's approach is different and requires a lot of knowledged and specialized code. It still is very challenging to ensure that all changes are seen and processed in the same order while minimally impacting the database.

Debezium provides modules that do this work for you. Some modules are generic and work with multiple database management systems, but are also a bit more limited in functionality and performance. Other modules are tailored for specific database management systems, so they are often far more capable and they leverage the specific features of the system. 

## Common use cases

There are a number of scenarios in which Debezium can be extremely valuable, but here we outline just a few of them that are more common.

### Cache invalidation

Automatically invalidate entries in a cache as soon as the record(s) for entries change or are removed. If the cache is running in a separate process (e.g., Redis, Memcache, Infinispan, and others), then the simple cache invalidation logic can be placed into a separate process or service, simplifying the main application. In some situations, the logic can be made a little more sophisticated and can use the updated data in the change events to update the affected cache entries.

### Simplifying monolithic applications

Many applications update a database and then do additional work after the changes are committed: update search indexes, update a cache, send notifications, run business logic, etc. This is often called "dual-writes" since the application is writing to multiple systems outside of a single transaction. Not only is the application logic complex and more difficult to maintain, dual writes also risk losing data or making the various systems inconsistent if the application were to crash after a commit but before some/all of the other updates were performed. Using change data capture, these other activities can be performed in separate threads or separate processes/services when the data is committed in the original database. This approach is more tolerant of failures, does not miss events, scales better, and more easily supports upgrading and operations.

### Sharing databases

When multiple applications share a single database, it is often non-trivial for one application to become aware of the changes committed by another application. One approach is to use a message bus, although non-transactional message busses suffer from the "dual-writes" problems mentioned above. However, this becomes very straightforward with Debezium: each application can monitor the database and react to the changes.

### Data integration

Data is often stored in multiple places, especially when it is used for different purposes and has slightly different forms. Keeping the multiple systems synchronized can be challenging, but simple ETL-type solutions can be implemented quickly with Debezium and simple event processing logic.

### CQRS

The [Command Query Responsibility Separation (CQRS)](http://martinfowler.com/bliki/CQRS.html) architectural pattern uses a one data model for updating and one or more other data models for reading. As changes are recorded on the update-side, those changes are then processed and used to update the various read representations. As a result CQRS applications are usually more complicated, especially when they need to ensure reliable and totally-ordered processing. Debezium and CDC can make this more approachable: writes are recorded as normal, but Debezium captures those changes in durable, totally ordered streams that are consumed by the services that asynchronously update the read-only views. The write-side tables can represent domain-oriented entities, or when CQRS is paired with [Event Sourcing](http://martinfowler.com/eaaDev/EventSourcing.html) the write-side tables are the append-only event log of commands.



## Building Debezium

The following software is required to work with the Debezium codebase and build it locally:

* [Git 2.2.1](https://git-scm.com) or later
* [JDK 8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) or [OpenJDK 8](http://openjdk.java.net/projects/jdk8/)
* [Maven 3.2.1](https://maven.apache.org/index.html) or later
* [Docker Engine 1.6](http://docs.docker.com/engine/installation/) or later

See the links above for installation instructions on your platform. You can verify the versions are installed and running:

    $ git --version
    $ javac -version
    $ mvn -version
    $ docker --version


### Why Docker?

Many open source software projects use Git, Java, and Maven, but requiring Docker is less common. Debezium is designed to talk to a number of external systems, such as various databases and services, and our integration tests verify Debezium does this correctly. But rather than expect you have all of these software systems installed locally, Debezium's build system uses Docker to automatically download or create the necessary images and start containers for each of the systems. The integration tests can then use these services and verify Debezium behaves as expected, and when the integration tests finish, Debezum's build will automatically stop any containers that it started.

Debezium also has a few modules that are not written in Java, and so they have to be required on the target operating system. Docker lets our build do this using images with the target operating system(s) and all necessary development tools.

Using Docker has several advantages:

1. You don't have to install, configure, and run specific versions of each external services on your local machine, or have access to them on your local network. Even if you do, Debezium's build won't use them.
1. We can test multiple versions of an external service. Each module can start whatever containers it needs, so different modules can easily use different versions of the services.
1. Everyone can run complete builds locally. You don't have to rely upon a remote continuous integration server running the build in an environment set up with all the required services. 
1. All builds are consistent. When multiple developers each build the same codebase, they should see exactly the same results -- as long as they're using the same or equivalent JDK, Maven, and Docker versions. That's because the containers will be running the same versions of the services on the same operating systems. Plus, all of the tests are designed to connect to the systems running in the containers, so nobody has to fiddle with connection properties or custom configurations specific to their local environments.
1. No need to clean up the services, even if those services modify and store data locally. Docker *images* are cached, so building them reusing them to start containers is fast and consistent. However, Docker *containers* are never reused: they always start in their pristine initial state, and are discarded when they are shutdown. Integration tests rely upon containers, and so cleanup is handled automatically.

### Building the code

First obtain the code by cloning the Git repository:

    $ git clone https://github.com/debezium/debezium.git
    $ cd keycloak

Then build the code using Maven:

    $ mvn clean install

The build starts and uses several Docker containers for different DBMSes. Note that if Docker is not running, you'll likely get an arcane error -- if this is the case, always verify that Docker is running, perhaps by using `docker ps` to list the running containers.


## Contributing

The Debezium community welcomes anyone that wants to help out in any way, whether that includes reporting problems, helping with documentation, or contributing code changes to fix bugs, add tests, or implement new features. See [this document](CONTRIBUTE.md) for details.
