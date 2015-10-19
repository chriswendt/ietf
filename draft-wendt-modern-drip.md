## Distributed Registry Protocol

**draft-wendt-modern-drip-00**

#### Abstract

This document describes a protocol for allowing a distributed set of nodes to synchronize a set of information in real-time with minimal amount of delay.  This is useful for registry types of information like identity and telephone numbers with associated routing and ownership information, although could be extended to support other distributed real-time information updates as well.

#### 1. Introduction

This document describes the Distributed Registry Protocol (DRiP).  DRiP defines a set of peer protocols for how an arbitrary number of nodes arranged in a distributed mesh architecture can be used to synchronize data in real-time across a network.

#### 2. Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC2119].

Initiator node - A node that initiates key-value data propagation.

Receiver node - A node that receives the propagated key-value data.

#### 3. DRiP Overview

DRiP uses a mix of a gossip protocol with update counters for distribution of key-value data with the addition of a voting system to avoid race conditions on writing of key-value data.

#### 3.1 Distributed MESH Architecture

The DRiP architecture is based on a peer-to-peer communication model where a given node associated with a data store is not necessarily aware of the total number of nodes in the entire network. Minimally, every node should reachable by at least one multi-node path from every other node. Each node in the DRiP network maintains a list of peer nodes from which it receives and transmits updates. Information is propagated by forwarding to it's peer nodes until the information received by a node has already been received.

       ___           ___                         ___           ___ 
      |DB |_________|DB |                       |DB |_________|DB |
      |___|         |___|                       |___|         |___|
        |   Data      |                           |   Data      |  
        |   Store     |                           |   Store     |  
       _|_  Cluster  _|_                         _|_  Cluster  _|_ 
      |DB |_________|DB |                       |DB |_________|DB |
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
         |DB |_________|DB |                 |DB |_________|DB |
         |___|         |___|                 |___|         |___|
           |   Data      |                     |   Data      |  
           |   Store     |                     |   Store     |  
          _|_  Cluster  _|_                   _|_  Cluster  _|_ 
         |DB |_________|DB |                 |DB |_________|DB |
         |___|         |___|                 |___|         |___|

**Figure 1: Distributed Mesh Architecture**

#### 4. DRiP procedures

#### 4.1 Distributed Registry Rules

All nodes in the distributed mesh MUST agree upon a specific key-value data model. The choice of data store is implementation specific.

All nodes MUST be configured with at least one peer node before propagation. 

A node MUST have the ability to receive updates from all configured peer nodes. 

A node MUST NOT receive any updates or commands from peer nodes it is not configured, or MUST ignore any updates or commands it receives that are not from any configured peer nodes.

All nodes MUST send a periodic heartbeat or keep-alive messages via HTTPS to the respective peer nodes, if a heartbeat is not received the peer node is removed from the list of active peer nodes.

#### 4.2 Node State Types

A DRiP node should have mulitple peer nodes configured with either IP or FQDN association.  It should maintain a number of states of each peer node that impact when transactions can and can not be initiated to peer nodes or processed from peer nodes.

#### 4.2.1 Authorized

The Authorized peer node state defines whether a configured peer node can send any API commands to the node.  If the authorized state is false, all communications from that peer node to the node MUST be ignored.  (May not need this)

#### 4.2.2 Active

The Active peer node state defines whether the node should send any API command to the peer node.  If the active state is false, no API commands should be sent.

#### 4.2.2.1 API

Request: 

POST /active/:nodeid

TBD


#### 4.3 Key-Value Data Update

When a node has new data it wants to propogate to the distributed mesh, it initiates an Update.  This node is called the Initiator Node.

The Update consists of a Two-Phase Commit procedure in order to guarentee there are no race conditions for updating the same key's data, as well as for any error conditions in the distributed mesh that would cause the update to not complete for all nodes in the network.

The two phases are called the "voting" phase and the "commit" phase.

#### 4.3.1 Voting Phase

The voting phase is the phase where all nodes are queried to "vote" whether they are aware of any potential conflict that would cause the transaction not to complete.

The Initiator node MUST set a timeout period to get response from its peer nodes, for data propagated. This response (see section 4.5.1.2) will indicate a "vote" for whether the transaction can be completed based on any conflicting updates to the same entry. If all peer nodes vote "yes", then the second phase or commit phase in the local node is initiated. If any one of the peer nodes votes "no" or if there is no response from one or more peer nodes, then the commit of the information MUST not be completed. No action is taken for responses (see section 4.5.1.2) received after the timeout period.

       ___           ___      ___           ___      ___           ___ 
      |DB |_________|DB |    |DB |_________|DB |    |DB |_________|DB |
      |___|         |___|    |___|         |___|    |___|         |___|
        |   Data      |        |   Data      |        |   Data      |  
        |   Store     |        |   Store     |        |   Store     |  
       _|_  Cluster  _|_      _|_  Cluster  _|_      _|_  Cluster  _|_ 
      |DB |_________|DB |    |DB |_________|DB |    |DB |_________|DB |
      |___|         |___|    |___|         |___|    |___|         |___|
          \                     \                      |
           \                     \                     |
           _\___    Vote(HTTPS)  _\___    Vote(HTTPS)  |____
          |Node |  <----------  |Node |  ---------->  |Node |
          |  B  |---------------|  A  |---------------|  C  |
          |_____|  ---------->  |_____|  <----------  |_____|
                     Yes/No                Yes/No     
                     
**Figure 2: Voting Phase**  

The peer nodes known to the initiator node will continue propagate the information to their peer nodes and so on. However, these peer nodes beyond the initiator node will no longer need to keep track of the time interval for responses. A node will stop continuing to propagate information when it determines it has received the same information again. This can be determined by keeping track of a counter and originating node id.

#### 4.3.1.1 API - POST /voting

Request:

POST /voting

Description:

A post from either Initiator node or subsequent peer nodes to request a vote of "yes" or "no" whether the key-value data could be committed without error or conflict.

**Example (using cURL)**

Request:

    $ curl -i  -H "Content-Type: application/json" -H 
    	"DRiP-Node-ID: nodeA" -H "DRiP-Node-Counter: 1234" 
    	-H "DRiP-Node-Counter-reset: false" -X POST -d '<data>' 
    	https://peernode.com/voting

Response:

	HTTP/1.1 200 OK

#### 4.3.1.2 API - POST /voting/peernode/:nodeid/response/:response

Request:

POST /voting/peernode/:nodeid/response/:response

Description:

A POST from peer node back to node with response of vote.  Note: the voting procedure is intentionally split into two seperate full HTTP transactions for reliability.

**Example (using cURL)**

Request

	$ curl -i -X POST http://nodearegistry.com/node/nodeA/response/yes

Response

	HTTP/1.1 200 OK

#### 4.3.2 Commit Phase

The Initiator node upon receiving a succesful aggregated "yes" vote from all the peer registries should start the commit phase the registry that originated the gossip will now commit the data in the HTTPS request payload to its data store. Subsequently, this information is propagated to all the nodes so that each node in the mesh will commit the same information in to their respective data stores.

       ___           ___                         ___           ___ 
      |DB |_________|DB |                       |DB |_________|DB |
      |___|         |___|                       |___|         |___|
        |   Data      |                           |   Data      |  
        |   Store     |                           |   Store     |  
       _|_  Cluster  _|_                         _|_  Cluster  _|_ 
      |DB |_________|DB |                       |DB |_________|DB |
      |___|         |___|                       |___|         |___|
                        \                       /
                         \COMMIT               /COMMIT
                         _\___     COMMIT   _ /__
                        |Node |------------|Node |
                        |  A  |    HTTPS   |  C  |
                        |_____|            |_____|
                            \H           H/
                             \T         T/
                        COMMIT\T       P/COMMIT 
                               \P     P/  
                                \S   S/   
                               __\__ /  COMMIT    _____  
                              |Node |------------|Node |
                              |  B  |   HTTPS    |  D  |
                              |_____|            |_____|
                             /                   /
                            /COMMIT             /COMMIT
          ___           ___/                  _/__          ___ 
         |DB |_________|DB |                 |DB |_________|DB |
         |___|         |___|                 |___|         |___|
           |   Data      |                     |   Data      |  
           |   Store     |                     |   Store     |  
          _|_  Cluster  _|_                   _|_  Cluster  _|_ 
         |DB |_________|DB |                 |DB |_________|DB |
         |___|         |___|                 |___|         |___|

      
Figure 4: Commit Phase

#### 4.3.2.1 API - POST /commit

Request:

POST /commit

Description:

A commit command is sent from Initiator or subsequent peer nodes to signal the commit of the data to the data base.  Once a 200OK response is received back, the key-value data should be written to the local data store.

**Example (using cURL)**

Request:

	$ curl -i  -H "Content-Type: application/json" -H 
		"DRiP-Node-ID: nodeA" -H "DRiP-Node-Counter: 1234" 
		-H "DRiP-Node-Counter-reset: false" -X POST 
		-d '<key-value data>' https://peernode.com/commit

Response:

	HTTP/1.1 200 OK


#### 4.4 Node Sync Operation

A node, either newly added to the ditsributed mesh or re-activated after being out-of-service due to network issues or other anomalies, will first inform its peer nodes to add this node to their list of peer nodes via the active state API. The node should be prepared to start receiving Updates at this point.

The node should immediately select a peer node and send a Sync transaction.  The peer node MUST start propogating a comprehensive and complete set of Updates of key-value data from data store. 

The two phase commit does NOT apply here as the contents of the nodes's data store is either outdated or empty. During this phase (HTTPS requests received will have DRiP-Sync-Complete field value set to false), this node SHOULD NOT become an Initiator node to provision data. While this transaction is going on, this node MUST vote "yes" to all real-time updates.

#### 4.3.2.1 API

Description: API call for initiating a full registry synchronization from node to peer-node.

Request:

PUT /sync/node/:nodeid

Response:

HTTP/1.1 200OK

#### 4.3.3 Transaction State Diagram

	                           _________ 
	  ----------------------->|         |
	 |                        | Waiting |
	 |                        | For     |
	 |   ---------------------| Events  |
	 |  | (Update,            |_________|
	 |  |  Start Timer)           
	 |  |                            --------------------------------  
	 |  |                           | Received Update From Peer Node |
	 |  |                           |                                |
	 |  |             ______________|_  If key matches an            |
	 |  |            |                |  in-progress update          |
	 |   ----------->|                |  vote "no".                  |
	 |               | Waiting For    |  Otherwise, vote "yes".      |
	 |               | Response From  |                              |
	 |               | Peer Nodes     |  Received Sync               |
	 |               |                |<-----------------------------
	 |               |                |                                
	 |           ----|                |----                            
	 |  Timer   |    |________________|    |                           
	 |  Expired |                          | Received Votes
	 |          |                          | From All Peer 
	 |          |                          | Nodes         
	 |          |      _______________     |
	 |          |    |                |    |
	 |          |    |                |    |
	 |           --->|                |<---
	 |               |  Validating    |    
	 | (If all Votes |  Votes         |    
	 |  are "YES",   |                |    
	 | propagate     |                |        
	 | commit)       |                |        
	  ---------------|________________|        
                          


### 4.4 Key-Value Data Propagation Rules

A node propagates key-value data to all its peer nodes except the the node from which it received data. For example, in Figure 1, when node B receives key-value data from node A, it will propagate the data received to nodes C and D but not back to node A.

For each transaction type (Update or Sync), the following action MUST take place when a node receives a HTTPS request with propagated key-value data

If DRiP-Node-ID field value (in the HTTP header) contains Initiator node ID that has never been seen, both DRiP-Node-ID and DRiP-Node-Counter field values MUST be stored for future reference and the key-value data is propagated to all peer nodes.

If DRiP-Node-ID field value ((in the HTTP header) matches with a stored node ID and DRiP-Node-Counter-reset field value is false.

The received key-value data MUST be propagated to the peer nodes if DRiP-Node-Counter field value is greater than the saved counter value. The DRiP-Node-Counter field value MUST be saved as the new counter for the stored node ID.

- If DRiP-Node-Counter field value is less than or equal to saved counter value, then no action is necessary (key-value data MUST NOT be propagated to peer nodes). This ensures that propagation stops when all nodes have received the key-value data from the Initiator node.
- If DRiP-Node-ID field value matches with a stored node ID and DRiP-Node-Counter-reset field value is true.
- The received key-value data MUST be propagated to the peer nodes. The DRiP-Node-Counter field value MUST be saved as the new counter for the stored node ID.

#### 4.4.1 Key-Value Data Update Entitlement Verification

When a node owner would like to create or modify particular key-value data, generally in the context of a registry, there MAY be a verification procedure that key-value data write or modification can be performed.  This could include validating whether key-value data is entitled to be written, modified or subsequently propogated based on application policy.  For example, identity or telephone number ownership or porting.  The exact mechanics of this are out of scope of this document and are generally application specific.

### 4.5 Custom HTTP header fields

Custom HTTP header fields will be used to carry node specific information.

| Field Name       | Description                                        | Example              |
| :--------------- | :------------------------------------------------- | :------------------- |
| DRiP-Node-ID | Each node in the mesh MUST have a unique identifier.  An Initiator node MUST set its own node ID as the field value. A Receiver Node MUST NOT change the DRiP-Node-ID field value in the HTTPS request as it is propagated to its peer nodes (nodes it is aware of). | DRiP-Node-ID: xyz123 |

| Field Name       | Description                                        | Example              |
| :--------------- | :------------------------------------------------- | :------------------- |
DRiP-Node-Counter | Every node maintains a count of the number of times it initiates key-value data propagation. This counter MUST be an unsigned type, typically, a 64 bit integer. The Initiator node MUST set this count as the field value. The Receiver node MUST NOT change the DRiP-Node-Counter field value in the HTTPS request as it is propagated to its peer nodes (nodes it is aware of).| DRiP-Node-Counter: 123

| Field Name       | Description                                        | Example              |
| :--------------- | :------------------------------------------------- | :------------------- |
DRiP-Node-Counter-reset | A node can reset the count (to zero) of the number of times it initiates key-value data propagation. If the counter value is reset, prior to initiating data propagation, then this field value MUST be set to true. Otherwise, it MUST be set to false, at all times. A typical use case to reset the counter value is when the counter (of unsigned type) value wraps around. The Initiator node MUST set this field value to either true or false. The Receiver node MUST NOT change the DRiP-Node-Counter-reset field value in the HTTPS request as it is propagated to its peer nodes (nodes it is aware of). | DRiP-Node-Counter-reset: false

| Field Name       | Description                                        | Example              |
| :--------------- | :------------------------------------------------- | :------------------- |
DRiP-Transaction-Type | The Initiator node MUST set this field value to be either "update" or "sync". See section 3.4. The Receiver node MUST NOT change the DRiP-Transaction-Type field value in the HTTPS request as it is propagated to its peer nodes (nodes it is aware of). | DRiP-Transaction-Type: update

| Field Name       | Description                                        | Example              |
| :--------------- | :------------------------------------------------- | :------------------- |
DRiP-Sync-Complete | For sync on demand transaction type, the Initiator node MUST set this field value to be true, if no more key-value data is left to be propagated. Otherwise, this field value MUST be set to false. The Receiver node MUST NOT change the DRiP-Sync-Complete field value in the HTTPS request as it is propagated to its peer nodes (nodes it is aware of). | DRiP-Sync-Complete: false

### 4.5 Error Conditions

- A node does not receive periodic heartbeat from one or more peer nodes.

### 4.6 Handling Corner Cases

- TBD


## 5. Transport-Specific Guidelines

### 5.1 HTTPS

The specifics of secure communication is beyond the scope of this document. However, at a minimum, all nodes communicate via HTTPS and must contain CA signed certificates installed on them.

### 5.2 Authentication

The specifics of authentication is beyond the scope of this document.

## 6. Acknowledgements

## 7. References

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
