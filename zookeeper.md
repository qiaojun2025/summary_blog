# Overview



## ZooKeeper: A Distributed Coordination Service for Distributed Applications

ZooKeeper is a distributed, open-source coordination service for distributed applications. It exposes a simple set of primitives that distributed applications can build upon to implement higher level services for synchronization, configuration maintenance, and groups and naming. It is designed to be easy to program to, and uses a data model styled after the familiar directory tree structure of file systems. It runs in Java and has bindings for both Java and C.

Coordination services are notoriously hard to get right. They are especially prone to errors such as race conditions and deadlock. The motivation behind ZooKeeper is to relieve distributed applications the responsibility of implementing coordination services from scratch.



## Design Goals

- **ZooKeeper is simple.**

- **ZooKeeper is replicated.**


- **ZooKeeper is ordered.** 

- **ZooKeeper is fast.**



### Data model and the hierarchical namespace

The namespace provided by ZooKeeper is much like that of a standard file system. A name is a sequence of path elements separated by a slash (/). Every node in ZooKeeper's namespace is identified by a path.



### Nodes and ephemeral nodes

Unlike standard file systems, each node in a ZooKeeper namespace can have data associated with it as well as children. It is like having a file-system that allows a file to also be a directory. (ZooKeeper was designed to store coordination data: status information, configuration, location information, etc., so the data stored at each node is usually small, in the byte to kilobyte range.) We use the term *znode* to make it clear that we are talking about ZooKeeper data nodes.

Znodes maintain a stat structure that includes version numbers for data changes, ACL changes, and timestamps, to allow cache validations and coordinated updates. Each time a znode's data changes, the version number increases.

The data stored at each znode in a namespace is read and written atomically. Reads get all the data bytes associated with a znode and a write replaces all the data. Each node has an Access Control List (ACL) that restricts who can do what.

ZooKeeper also has the notion of ephemeral nodes. These znodes exists as long as the session that created the znode is active. When the session ends the znode is deleted.



### Conditional updates and watches

ZooKeeper supports the concept of *watches*. Clients can set a watch on a znode. A watch will be triggered and removed when the znode changes. When a watch is triggered, the client receives a packet saying that the znode has changed. If the connection between the client and one of the ZooKeeper servers is broken, the client will receive a local notification.

**New in 3.6.0:** Clients can also set permanent, recursive watches on a znode that are not removed when triggered and that trigger for changes on the registered znode as well as any children znodes recursively.



### Guarantees

ZooKeeper is very fast and very simple. Since its goal, though, is to be a basis for the construction of more complicated services, such as synchronization, it provides a set of guarantees. These are:

- Sequential Consistency - Updates from a client will be applied in the order that they were sent.
- Atomicity - Updates either succeed or fail. No partial results.
- Single System Image - A client will see the same view of the service regardless of the server that it connects to. i.e., a client will never see an older view of the system even if the client fails over to a different server with the same session.
- Reliability - Once an update has been applied, it will persist from that time forward until a client overwrites the update.
- Timeliness - The clients view of the system is guaranteed to be up-to-date within a certain time bound.



### Simple API

One of the design goals of ZooKeeper is providing a very simple programming interface

- *create* : creates a node at a location in the tree
- *delete* : deletes a node
- *exists* : tests if a node exists at a location
- *get data* : reads the data from a node
- *set data* : writes data to a node
- *get children* : retrieves a list of children of a node
- *sync* : waits for data to be propagated



### Implementation

The picture shows the high-level components of the ZooKeeper service. With the exception of the request processor, each of the servers that make up the ZooKeeper service replicates its own copy of each of the components.

The replicated database is an in-memory database containing the entire data tree. Updates are logged to disk for recoverability, and writes are serialized to disk before they are applied to the in-memory database.

Every ZooKeeper server services clients. Clients connect to exactly one server to submit requests. Read requests are serviced from the local replica of each server database. Requests that change the state of the service, write requests, are processed by an agreement protocol.

As part of the agreement protocol all write requests from clients are forwarded to a single server, called the *leader*. The rest of the ZooKeeper servers, called *followers*, receive message proposals from the leader and agree upon message delivery. The messaging layer takes care of replacing leaders on failures and syncing followers with leaders.

ZooKeeper uses a custom atomic messaging protocol. Since the messaging layer is atomic, ZooKeeper can guarantee that the local replicas never diverge. When the leader receives a write request, it calculates what the state of the system is when the write is to be applied and transforms this into a transaction that captures this new state.


# Programmer's Guide



## The ZooKeeper Data Model

ZooKeeper has a hierarchal namespace, much like a distributed file system. The only difference is that each node in the namespace can have data associated with it as well as children. It is like having a file system that allows a file to also be a directory.



### ZNodes

Every node in a ZooKeeper tree is referred to as a *znode*. Znodes maintain a stat structure that includes version numbers for data changes, acl changes.  The stat structure also has timestamps. The version number, together with the time stamp, allows ZooKeeper to validate the cache and to coordinate updates. Each time a znode's data changes, the version number increases.



Znodes are the main entity that a programmer access. They have several characteristics that are worth mentioning here.

#### Watches

Clients can set watches on znodes. Changes to that znode trigger the watch and then clear the watch. When a watch triggers, ZooKeeper sends the client a notification.



#### Data Access

The data stored at each znode in a namespace is read and written atomically. Reads get all the data bytes associated with a znode and a write replaces all the data. Each node has an Access Control List (ACL) that restricts who can do what.

ZooKeeper was not designed to be a general database or large object store. Instead, it manages coordination data.



#### Ephemeral Nodes

ZooKeeper also has the notion of ephemeral nodes. These znodes exists as long as the session that created the znode is active. When the session ends the znode is deleted. Because of this behavior ephemeral znodes are not allowed to have children



#### Sequence Nodes -- Unique Naming

When creating a znode you can also request that ZooKeeper append a monotonically increasing counter to the end of path. This counter is unique to the parent znode



#### Container Nodes

**Added in 3.6.0**

ZooKeeper has the notion of container znodes. Container znodes are special purpose znodes useful for recipes such as leader, lock, etc. When the last child of a container is deleted, the container becomes a candidate to be deleted by the server at some point in the future.



#### TTL Nodes

**Added in 3.6.0**

When creating PERSISTENT or PERSISTENT_SEQUENTIAL znodes, you can optionally set a TTL in milliseconds for the znode. If the znode is not modified within the TTL and has no children it will become a candidate to be deleted by the server at some point in the future.



### ZooKeeper Stat Structure

The Stat structure for each znode in ZooKeeper is made up of the following fields:

- **czxid** The zxid of the change that caused this znode to be created.
- **mzxid** The zxid of the change that last modified this znode.
- **pzxid** The zxid of the change that last modified children of this znode.
- **ctime** The time in milliseconds from epoch when this znode was created.
- **mtime** The time in milliseconds from epoch when this znode was last modified.
- **version** The number of changes to the data of this znode.
- **cversion** The number of changes to the children of this znode.
- **aversion** The number of changes to the ACL of this znode.
- **ephemeralOwner** The session id of the owner of this znode if the znode is an ephemeral node. If it is not an ephemeral node, it will be zero.
- **dataLength** The length of the data field of this znode.
- **numChildren** The number of children of this znode.



## ZooKeeper Sessions

A ZooKeeper client establishes a session with the ZooKeeper service by creating a **handle** to the service using a language binding.

Once created, the handle starts off in the **CONNECTING** state and the client library tries to connect to one of the servers that make up the ZooKeeper service at which point it switches to the **CONNECTED** state. During normal operation the client handle will be in one of these two states. If an unrecoverable error occurs, such as session expiration or authentication failure, or if the application explicitly closes the handle, the handle will move to the **CLOSED** state. The following figure shows the possible state transitions of a ZooKeeper client:



When a client gets a handle to the ZooKeeper service, ZooKeeper creates a ZooKeeper session, represented as a 64-bit number, that it assigns to the client. If the client connects to a different ZooKeeper server, it will send the session id as a part of the connection handshake. 

One of the parameters to the ZooKeeper client library call to create a ZooKeeper session is the session timeout in milliseconds. 

When a client (session) becomes partitioned from the ZK serving cluster it will begin searching the list of servers that were specified during session creation. 

Session expiration is managed by the ZooKeeper cluster itself, not by the client.

Another parameter to the ZooKeeper session establishment call is the default watcher. Watchers are notified when any state change occurs in the client.

The session is kept alive by requests sent by the client. If the session is idle for a period of time that would timeout the session, the client will send a PING request to keep the session alive.

Once a connection to the server is successfully established (connected) there are basically two cases where the client lib generates connection loss when either a synchronous or asynchronous operation is performed and one of the following holds:

1. The application calls an operation on a session that is no longer alive/valid
2. The ZooKeeper client disconnects from a server when there are pending operations to that server, i.e., there is a pending asynchronous call.



**Local session**. Added in 3.5.0, mainly implemented by [ZOOKEEPER-1147](https://issues.apache.org/jira/browse/ZOOKEEPER-1147).

- Background: The creation and closing of sessions are costly in ZooKeeper because they need quorum confirmations, they become the bottleneck of a ZooKeeper ensemble when it needs to handle thousands of client connections. So after 3.5.0, we introduce a new type of session: local session which doesn't have a full functionality of a normal(global) session, this feature will be available by turning on *localSessionsEnabled*.



## ZooKeeper Watches

All of the read operations in ZooKeeper - **getData()**, **getChildren()**, and **exists()** - have the option of setting a watch as a side effect. 

Here is ZooKeeper's definition of a watch: a watch event is **one-time trigger**, **sent to the client** that set the watch, which occurs when **the data for which the watch was set changes**. 

- **One-time trigger** One watch event will be sent to the client when the data has changed.
- **Sent to the client** This implies that an event is on the way to the client, but may not reach the client before the successful return code to the change operation reaches the client that initiated the change. Watches are sent asynchronously to watchers.
- **The data for which the watch was set** This refers to the different ways a node can change. 

Watches are maintained locally at the ZooKeeper server to which the client is connected. This allows watches to be lightweight to set, maintain, and dispatch. When a client connects to a new server, the watch will be triggered for any session events. Watches will not be received while disconnected from a server. When a client reconnects, any previously registered watches will be reregistered and triggered if needed. In general this all occurs transparently. There is one case where a watch may be missed: a watch for the existence of a znode not yet created will be missed if the znode is created and deleted while disconnected.


