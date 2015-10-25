-------------------------------------------

What is this repository for?

The specification of the Chord protocol can be found in the paper Chord: A Scalable Peer-to-peer Lookup Service for Internet Applications by Ion Stoica, Robert Morris, David Karger, M. Frans Kaashoek, Hari Balakrishnan.
The goal of this project is to implement in Scala using the actor model the Chord protocol and a simple object access service to prove its usefulness.

-------------------------------------------

How do I get set up? 

* Install SBT.
* Install Scala

-------------------------------------------

How to run ?

>sbt
>compile
>run numNodes numRequests 

To run with deletion of nodes enabled,

>run numNodes numRequests deletionEnabled

Input: The input provided (as command line to your project3.scala) will be of the form:

project3.scala numNodes numRequests

Where numNodes is the number of peers to be created in the peer to peer system and numRequests the number of requests each peer has to make. When all peers performed that many requests, the program can exit. Each peer should send a request/second.

-------------------------------------------
Who do I talk to?

* Repo owner or admin

-------------------------------------------
Largest network ? 

19500 Nodes.

Detailed output:
[WITHOUT FAILURE ENABLED]
NUMBER OF NODES       :19500
NUMBER OF REQUESTS    :10
FAILURE ENABLED:      :false
TIME TAKEN TO JOIN    :512442
TIME TAKEN TO SEARCH  :70069
NUMBER OF FILES FOUND :195000
NUMBER OF TOTAL HOPS  :1743448
AVERAGE NUMBER OF HOPS:8
LOG(NUMNODES)         :14.251186503524336
-------------------------------------------

[WITH FAILURE ENABLED]
NUMBER OF NODES       :1000
NUMBER OF REQUESTS    :10
FAILURE ENABLED:      :true
TIME TAKEN TO JOIN    :21521
TIME TAKEN TO SEARCH  :11818
NUMBER OF FILES FOUND :9900
NUMBER OF TOTAL HOPS  :67513
NUMBEROF DELETED NODES:10
AVERAGE NUMBER OF HOPS:6
LOG(NUMNODES)         :9.951284714966972

*Time in milliseconds

-------------------------------------------

[WITH FAILURE ENABLED]
NUMBER OF NODES       :10000
NUMBER OF REQUESTS    :10
FAILURE ENABLED:      :true
TIME TAKEN TO JOIN    :220031
TIME TAKEN TO SEARCH  :74917
NUMBER OF FILES FOUND :98919
NUMBER OF TOTAL HOPS  :839885
NUMBEROF DELETED NODES:100
AVERAGE NUMBER OF HOPS:8
LOG(NUMNODES)         :13.273212809854334

-------------------------------------------

1. We have been able to simulate sequential joining of actors to the Chord. Our Hash space is constant. It is upto the largest integer value possible (generally 2^30).
2. After all the nodes are joined, akka scheduler will schedule each actor to make one request per second (numRequests will be controlled using CL variables).
3. Each node will perform designated numRequests and will increment the global variable of files found so far.
4. System will print the average number of hops after all the file search requests are over. System will exit after this.

If you randomly shutdown a node, system will still be able to search the given file after it recovers using stabilize routine.


1. The flag Constants.deletionEnabled is set to false by default. To run with deltion enabled, SET THIS FLAG TO TRUE. 
2. The number of nodes to be deleted will be calculated using Math.ceil(numNodes/10) by default. This can be supressed if you use a third CL argument as explained above.

How the failure model is implemented?

Instead of having a single successor, we maintain multiple successors (successorList as explained in the paper) for every node to stabilize the system in case a node dies.

In our case, we have two successors for every node, which are immediate successor and supersuccessor(immediate successor's successor). Every node watches their successors. Consider the case when a node dies, the successor and predecessor relation between two nodes on the Chord is BROKEN at this point. Thanks to the watch on the successor nodes, each node in system will know when its successor has died and it will update its successor to the next available successor in the successorList. It will also intimate its new successor to update its stale predecessor information. By doing this the system will be JOINED again. After which the finger table is modified accordingly.

We have tested our logic by deleting up to ten percent of the nodes in a huge (~15000) Chord topology. The file search took relatively longer number of hops (due to some outdated information in finger entries initially, which will be updated eventually) but the system is resilient and was able to find the files in logarthmic number of hops.

Observations: 
Nodes keeping the information of multiple successors is crucial.
The robust nature of chord lies in its ability to find hashKeys in logarthimic time even when multiple nodes join/leave the network