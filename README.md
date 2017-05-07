# gen\_server\_cluster

[A Framework For Clustering Generic Server Instances](http://www2.erlangcentral.org/wiki/index.php?title=A_Framework_for_Clustering_Generic_Server_Instances)

[From ErlangCentral Wiki](http://www2.erlangcentral.org/wiki)

The following is quoted from the original article by Christoph Dornheim followed by a critique of behavior on netsplit.

### Author
Christoph Dornheim

### Overview
This tutorial describes a simple framework developed for running servers in a cluster. The server to be clustered must be an instance of the gen_server behaviour which is the common way of designing servers according to the Erlang OTP principles (see the gen_server design principle and API).

We first explain what server clustering means and why it makes sense to run a server in a cluster. After showing the key ideas of the framework, we describe in detail how server clustering with this framework works. This is illustrated by a short example. The framework consists of only one module gen_server_cluster. Before presenting the complete code, we make clear how to use its API functions. Finally, we give another application of gen_server_cluster: a small but highly available chat system.

### Motivation
In general, clustering a server means running several instances of this server simultaneously in a way that the cluster consisting of these servers appears to clients to be a single server process dispatching their requests. One of the main reasons for clustering is to increase service availability: if one of the servers in the cluster crashes or is no longer reachable due to network failures, the service is kept alive by some other server in the cluster. Thus, to provide high availability, the servers in the cluster are located preferably on different machines.

Due to Erlang's built-in mechanism for fault-detection, reaching a certain level of availability for an Erlang gen_server instance is made easy: just let the server process be monitored by some other process, e.g. by an OTP supervisor, that restarts the server when it has terminated. In case that the node the server has been running on went down, you can simply restart the server on some different node, if available. Obviously, the monitoring process needs to be observed as well.

There is, however, a problem when the server is holding some state needed for servicing the client requests. To provide continuous service, the state of the restarted server process should be initialized to the last version of the terminated server state. As a solution, the server can persist its state in a database or file system the restarted server must read, but this makes the service availability depend entirely on the database availability.

Instead of restarting a server to make it available again, the framework we will describe allows a server to be run in a cluster. The cluster consists of a dynamically extensible set of server processes each running on a different Erlang node. The key idea is that exactly one server process is responsible for dispatching client requests and updating the states of all servers to keep them in sync. If the active process dies, all other background processes compete for becoming the new active one, but only one of them wins. Thus, the service is available as long as the cluster consists of at least one server at any time. Availability is increased just by adding background processes to the cluster.

See [A Framework For Clustering Generic Server Instances](http://www2.erlangcentral.org/wiki/index.php?title=A_Framework_for_Clustering_Generic_Server_Instances) for complete description.

### Example
To run id_server in a cluster of two connected Erlang nodes, we first start two shells using erl -sname node1 and erl -sname node2, respectively, and then establish a connection between them, e.g. using net_adm:ping.

We start the cluster at node1 by calling id_server:start_cluster:

Next we increase the cluster by starting a second server at node2 which becomes a local server. To demonstrate server availability provided by this cluster, we stop the current global server using a function of gen_server_cluster: gen_server_cluster:stop(id_server,global) (alternatively, we can simply cancel node1.) As the output at node2 shows, the former local server at node2 is now the new global server. The value returned when calling id_server:next_value proves that the state of id_server was correctly synchronized.

### Comments
See [this critique](http://levgem.livejournal.com/283022.html) by [Макс Лапшин](http://levgem.livejournal.com/)  on behavior after a netsplit and the disconnected node is reconnected.

> The whole article on trapexit doesn't tell anything about hard situation, when disconnected earlier node is connected back. Imagine: network looses connectivity, chat rooms get separated and there appears two separated infrastructures. Later admin fixes broken router, connects back two clusters (for example 4 nodes in each) and there appears situation when two global processes exists.

> Easiest way is not to do anything: erlang will kill one of global processes, second will be registered as a new global process.
It will be very good if all clients from first cluster are disconnected and then connect back. Much worser if shadow copies on
first cluster remain in unsynced state.

> I see the only way: not use gen_server:start_link({global, chat_server}.., but use 

> global:register_name(chat_server, self(), {?MODULE, resolve_global}).

> Declare function resolve_global/3, that will resolve such conflicts on node connection. You are required to 
implement a merge-state function, that will take whole state from second global process, pass it to first process and than
tell second process to become a shadow. All his shadow copies must know that they have changed master.

### Credits
[Christoph Dornheim](cd5@gmx.de)  
[Макс Лапшин](http://levgem.livejournal.com/)  

