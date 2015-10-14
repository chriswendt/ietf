### Distributed Registry Protocol

**draft-wendt-modern-drip-00**

**Abstract**

This document describes a protocol for allowing a distributed set of nodes to synchronize a set of information in real-time with minimal amount of delay.  This is useful for registry types of information like identity and telephone numbers with associated routing and ownership information, although could be extended to support other distributed real-time information updates as well.

**1. Introduction**

This document describes the Distributed Registry Protocol (DRiP).  DRiP defines a set of peer protocols for how an arbitrary number of nodes arranged in a distributed mesh architecture can be used to synchronize data in real-time across a network.

**2. DRiP Overview**

DRiP uses a mix of Gossip protocol with the addition of a light weight voting system.

**2.1 Distributed MESH Architecture**

The DRiP architecture is based on a peer-to-peer communication model where a given node associated with a data store is not necessarily aware of the total number of nodes in the entire network. Minimally, every node should reachable by at least one multi-node path from every other node. Each node in the DRiP network maintains a list of peer nodes from which it receives and transmits updates. Information is propogated by forwarding to it's peer nodes until the information received by a node has already been received.

       ___           ___                         ___           ___ 
      | D |_________| D |                       | D |_________| D |
      |___|         |___|                       |___|         |___|
        |   Data      |                           |   Data      |  
        |   Store     |                           |   Store     |  
       _|_  Cluster  _|_                         _|_  Cluster  _|_ 
      | D |_________| D |                       | D |_________| D |
      |___|         |___|                       |___|         |___|
                        \                       /
                         \                     /
                         _\___     DRIP     _ /__
                        |Node |------------|Node |
                        |  A  |    HTTPS   |  C  |
                        |_____|            |_____|
                            \H           H/
                            D\T         T/D
                             R\T       P/R 
                              I\P     P/I  
                               P\S   S/P   
                               __\__ /   DRIP     _____  
                              |Node |------------|Node |
                              |  B  |   HTTPS    |  D  |
                              |_____|            |_____|
                             /                   /
          ___           ___ /                 __/_          ___ 
         | D |_________| D |                 | D |_________| D |
         |___|         |___|                 |___|         |___|
           |   Data      |                     |   Data      |  
           |   Store     |                     |   Store     |  
          _|_  Cluster  _|_                   _|_  Cluster  _|_ 
         | D |_________| D |                 | D |_________| D |
         |___|         |___|                 |___|         |___|

**Figure 1: Distributed Mesh Architecture**

**3. DRiP procedures**

**3.1 Entry Entitlement Verification**

When a node owner would like to create or modify a set of information, generally in the context of a registry, there MAY be a verification procedure that an entry can be created.  This could include validating whether an entry is entitled to be written or modified in the first place.  For example, identity or telephone number ownership or porting.  The exact mechanics of this are out of scope of this document and are generally application specific.

       ___           ___ 
      | D |---------| D |
      |___|         |___|
        |   Data      |  
        |   Store     |      Query      _____ 
       _|_  Cluster  _|_   <---------- |     |
      | D |---------| D |______________|Node |
      |___|         |___|  ----------> |_____|
                            Response
                            
 **Figure 2: Verify information on data store**
 
**CW:** is this needed? does provisioning go through DRiP node or talk to Data Store in parallel?

**3.2 Voting Phase**

Assuming the update is determined to be allowed, the information propogation begins.  This starts with the creation of a JSON object containing a common set of information,

* node id
* node specific counter
* auth info?

In addition, the specific entry information is added to the JSON object.
Once created, the JSON object is then transmitted to all of the nodes peer nodes and a corresponding transmission initiation time is saved.  This time is used to set the timeout period for each transmission.

For each transmission to a peer node, the node waits for a certain time interval for a response.  This response will indicate a "vote" for whether the transaction can be completed based on any conflicting updates to the same entry. If all peer nodes vote "yes", then a commit of the information to the local node is initiated. If any one of the peer nodes votes "no" or if there is no response from one or more peer nodes, then the commit of the information MUST not be completed.

       ___           ___      ___           ___      ___           ___ 
      | D |_________| D |    | D |_________| D |    | D |_________| D |
      |___|         |___|    |___|         |___|    |___|         |___|
        |   Data      |        |   Data      |        |   Data      |  
        |   Store     |        |   Store     |        |   Store     |  
       _|_  Cluster  _|_      _|_  Cluster  _|_      _|_  Cluster  _|_ 
      | D |_________| D |    | D |_________| D |    | D |_________| D |
      |___|         |___|    |___|         |___|    |___|         |___|
          \                     \                         /
           \                     \                       /
           _\___    Vote(HTTPS)  _\___    Vote(HTTPS)  _/___
          |Node |  <----------  |Node |  ---------->  |Node |
          |  B  |---------------|  A  |---------------|  C  |
          |_____|   ----------> |_____|  <----------  |_____|
                     Yes/No                Yes/No     
                     
**Figure 3: Voting Phase**  

The peer nodes known to the originating node will continue propagate the information to their peer nodes and so on. However, these peer nodes will no longer need to keep track of the time interval for responses. A node will stop continuing to propagate information when it determines it has received the same information again. This can be determined by keeping track of the counter and originating node id.

**3.3 Commit Phase**

The commit phase is similar to the voting phase except upon receiving a "yes" from all the peer registries, the registry that originated the gossip will now commit the public ID, service ID and routing ID information to its data store. Subsequently, this information is propagated to all the nodes so that each node in the mesh will commit the same information in to their respective data stores.

       ___           ___      ___           ___      ___           ___ 
      | D |_________| D |    | D |_________| D |    | D |_________| D |
      |___|         |___|    |___|         |___|    |___|         |___|
        |   Data      |        |   Data      |        |   Data      |  
        |   Store     |        |   Store     |        |   Store     |  
       _|_  Cluster  _|_      _|_  Cluster  _|_      _|_  Cluster  _|_ 
      | D |_________| D |    | D |_________| D |    | D |_________| D |
      |___|         |___|    |___|         |___|    |___|         |___|
          \                     \                         /
           \                     \                       /
           _\___  Commit(HTTPS)  _\___  Commit(HTTPS)  _/__
          |Node | <----------   |Node |  ---------->  |Node |
          |  B  |---------------|  A  |---------------|  C  |
          |_____|   ----------> |_____|  <----------  |_____|
                     Yes/No                Yes/No     
     
Figure 4: Commit Phase 

       ___           ___                         ___           ___ 
      | D |_________| D |                       | D |_________| D |
      |___|         |___|                       |___|         |___|
        |   Data      |                           |   Data      |  
        |   Store     |                           |   Store     |  
       _|_  Cluster  _|_                         _|_  Cluster  _|_ 
      | D |_________| D |                       | D |_________| D |
      |___|         |___|                       |___|         |___|
                        \                       /
                         \COMMIT               /COMMIT
                         _\___     commit   _ /__
                        |Node |------------|Node |
                        |  A  |    HTTPS   |  C  |
                        |_____|            |_____|
                            \H           H/
                             \T         T/
                       COMMIT \T       P/COMMIT 
                               \P     P/  
                                \S   S/   
                               __\__ /   COMMIT   _____  
                              |Node |------------|Node |
                              |  B  |   HTTPS    |  D  |
                              |_____|            |_____|
                             /                   /
                            /COMMIT                   / COMMIT
          ___           ___/                  _/__          ___ 
         | D |_________| D |                 | D |_________| D |
         |___|         |___|                 |___|         |___|
           |   Data      |                     |   Data      |  
           |   Store     |                     |   Store     |  
          _|_  Cluster  _|_                   _|_  Cluster  _|_ 
         | D |_________| D |                 | D |_________| D |
         |___|         |___|                 |___|         |___|

      
Figure 5: Propagate final Commit  

**4. DRiP transaction types**

**4.1 Real-Time Updates**

When a node has new information, the update is propagated to its peers immediately to provide timely updates of information.

**4.2 Full Synchronization**

For maximum reliability and validation of information contained in the distributed registry, periodically, a node SHOULD propagates all of the entries of information to its peers.

In practice, this could be done in low traffic times or other convienient times that would have less chance to slow down any concurrent real-time updates being processed.

**5. Transport-Specific Guidelines**

**5.1 HTTPS**

**5.2 Authentication** 

**6. Acknowledgements**

**7. References**

**Author's Address**

Harsha Bellur

Comcast

One Comcast Center
   
Philadelphia, PA 19103
   
US

Email: harsha_bellur@cable.comcast.com


Chris Wendt (editor)

Comcast

One Comcast Center
   
Philadelphia, PA 19103
   
US

Email: chris-ietf@chriswendt.net
