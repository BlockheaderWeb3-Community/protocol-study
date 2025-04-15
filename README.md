# ETHEREUM EXECUTION TRANSACTION: GOSSIP PROTOCOL, ENGINE API, EXECUTION SPECIFICATION

## INTRODUCTION
The execution layer is responsible for processing transactions, executing smart contract, and maintaining the state of Ethereum.

**The role of the execution layer can be seen as performing a state transition function:** Meaning that the execution layer of Ethereum is essentially responsible for transforming the blockchain from one valid state to another.

The Ethereum Execution layer (Formerly know as the "main chain" before the Merge) handles transaction processing. Here's how it works:

#### Transaction Submission and Propagation

**1. Transaction Creation:** A user creates a transaction with parameters (nonce, gas price, gas limit, recipient, value, data, and signature).

**2. Gossiping Mechanism:** After submission, the transaction is propagated through the network using a peer-to-peer gossip protocol. Each node validates the transaction upon receipt before forwarding it to peers.

**3. Local Validation:** Nodes perform preliminary checks:
- Signature verification
- Nonce correctness
- Sufficient account balance to cover value + gas costs
- Gas limit sufficiency for basic operations

**4. Mempool Addition:** Valid transactions are added to the node's local mempool, organized primarily by gas price.

#### Block Construction

**1. Transaction Selection:** Validators select transactions from their mempool, typically priotizing those with higher gas prices first(maximizing their rewards). 

**2. Ordering:** Transactions are ordered by gas price and nonce to prevent double-spending and ensure sequential execution of transactions from the same account.

#### Transaction Execution

**1. Initial State:** The execution begins with the current world state(a Merkle Patricia Tree).

**2. Gas Payment:** The gas limit * gas price amonut is deducted from the senders account upfront.

**3. EVM Execution:** For contract interactions, the EVM executes the codes:
- The bytecode is processed, instruction by instruction.
- Each operation consumes a predetermined amount of gas.
- Storage operations are tracked for state updates.
- Contract calls may trigger further executions in a nested manner.

**4. State Changes:** The execution modifies:
- Account balance
- Contract storage values.
- Contract code(In case of contract creation).
- Account Nonces.

**5. Gas Calculation:** Gas is consumed based on the complexity of operations:
- Basic operations(ADD, MUL): low gas cost.
- Storage operations(SSTORE): high gas cost.
- Contract creation: very high gas cost.

**6.Gas Refund:** Unused gas is refunded to the sender's account.
**7. Reciept Generation:** A transaction receipt is created containing:
- Transaction hash
- Block information 
- Gas used
- Logs(events)emitted.
- Execution Status(Success or Failure)

#### State Finalization
**1. State Transition:** The execution produces a new state root.
**2. State Commitment:** The new state is commited to the blockchain.
**3. Receipt Trie:** Transaction receipts are organized into trie structure for efficient querying.


These processes are handled by Ethereum clients(like Geth or Nethermind)implementing the execution layer specifications. After the Merge, the execution layer works in conjunction with the consensus layer(Beacon Chain)to acheive finality.![Screenshot from 2025-04-12 18-06-21](https://hackmd.io/_uploads/B1ssaG_R1g.png). 

---------




## GOSSIP PROTOCOL

The Ethereum execution layer(formerly called the Ethereum 1.0 layer)uses a specialized implementation of gossip protocol to propagate transactions and blocks across the network.



#### Transaction Propagation
**1. Initial Broadcast:** When a user submits a transaction, their node broadcasts it to a subset of connected peers(typically 13 peers).

**2. Mempool Management:** Each receiving nodes validates the transaction against criteria like gas price, nonce and signature before adding it to their local mempool.

**3. Selective Forwarding:** Nodes only propagate transactions that pass and meet minimum gas price requirements.

**4. De-Duplication:** Nodes track transaction hashes they've seen to avoid re-processing duplicates.



#### Block Propagation
**1. Block Announcement:** When a miner produces a block, they announce it to peers with a NewBlockMsg.

**2. Headers-First Approach:** Initially, only block headers are sent, allowing quick validation before full block download.

**3. Validation and Relay:** After validation, nodes propagate the blocks to their peers who hasn't seen it.


#### Efficiency Mechanisms

**1. Transaction Pools:** Pending transactions are stored in prioitized pools based on Gas price and other transactions.

**2. Canonical Chain Rules:** Nodes follow the chain with highest total difficulty(pre-merge)or follow the consensus layer's canonical chain(post-Merge).

**3. Peer Management:** Nodes maintain connnections to diverse peers accross different network locations.

**4. Message Rate Limiting:** Controls bandwit usage to prevent network flooding.



#### DevP2P Network
1. Ethereum's execution layer uses the DevP2P protocol or peer discovery and communication.
2. RLPx transport protocol handles encrypted communication between nodes.
3. Peers are discovered through DNS node list andn the Kademila-inspired node table.

This gossip-based system ensures Ethereum's execution layer can maintain consensus across thousands of global nodes while handling high transaction volumes efficiently.






## ENGINE API

The Engine API is a crucial component introduced during Ethereum's transition to proof of stake(The Merge). It enables communication between two critical layers of Ethereum: 

#### CORE PURPOSE

- Serves as interface between the consensus layer(Beacon Chain)and the execution layer(formerly known as Eth1).
- Allows the consensus client to instruct the execution client on which block to create and validate.


#### KEY FUNCTION

**1. BLOCK PRODUCTION:**
- Consensus layer requests execution payload via engine_forkchoiceupdatedV1.
- Execution client builds a block with transaction and returns it.
- Consensus client wraps this payload in a beacon block.

**2. BLOCK VALIDATION:**
- When a receivinng a block, consensus client extracts the execution payload.
- Sends it to the execution client via engine_newpayloadV1 for validation.
- Execution clent verifies state transitions and returns validity status.


**3. FORK CHOICE UPDATES:**
- Consensus layer regularly informs execution layer about chain head.
- Ensures both layers remain synchronized on canonical chain.


#### API METHODS
**-  engine_forkchoiceupdatedV1:** Updates head of chain and optionally requests block building.
**- engine_newpayloadV1:** Validates and executes a new block payload.
**- engine_getpayloadV1:** Retrieves a previously requested execution payload.
- Plus newer versions(V2, V3) that add support for withdrawals and blobs.


#### TECHNICAL IMPLEMENTATION
- JSON-RPC interface, typically on a separate authenticated port (8551)
- Uses JWT authentication tokens for security
- Follows versioned specifications allowing for backwards compatibility.

This API was critical for Ethereum's successful transition to Proof of Stake and continues to be refined through protocol upgrades.



## ETHEREUM EXECUTION SPECIFICATIONS

The Ethereum execution specs define how the blockchain processes transaction and updates state. Here's an overview of my the execution specs:

#### CORE COMPONENTS
#### STATE TRANSITIONS
![Screenshot from 2025-04-15 13-05-14](https://hackmd.io/_uploads/rJNKaTo0Jx.png)
This is a simple diagram of the state.

- The Ethereum state is a mapping of addresses to account states.
- Each transaction triggers a state transition following specific rules.
- Changes move the system from one valid state to another.



#### BLOCK PROCESSING

![Screenshot from 2025-04-15 16-04-53](https://hackmd.io/_uploads/H1TwIg2Cke.png)
![Screenshot from 2025-04-15 16-06-56](https://hackmd.io/_uploads/S1RDIlhAyg.png)
![Screenshot from 2025-04-15 16-07-42](https://hackmd.io/_uploads/r1CvLghCke.png)





**1. TRANSACTION VALIDATION:** Each transaction is validated for  proper format, sufficient gas, and valid signatures.

**2. EXECUTION:** Valid transactions are executed in sequences, modifying state.

**3. STATE ROOT CALCULATION:** A Merkle Patricia trie root represents the entire state

**4. RECIEPT GENERATION:** Execution results are stored in transaction reciepts.



#### KEY SPECIFICATION

##### EVM(ETHEREUM VIRTUAL MACHINE)

- Turing-complete execution environment.
- Stack-based architecture with well defined opcodes.
- Executes smart contract bytecode with gas metering.



##### GAS MECHANICS

- Gas limits prevent infinite loops and resource exhaustion.
- Different operations cost different amount of gas.
- Block gas limits cap total computation per block.


##### STATE STRUCTURE

- **Account contains:** nonce, balance, code and storage.
- **Two Account Types:** Externally owned account(EOA) and contract account.
- Storage is a key-value mapping specific to each contract.



##### CONSENSUS RULES

- Rules for determining valid blocks and canonical chain.
- **Post-Merge:** Executiion validity is checked by execution clients while block odering comes from consensus layer.


These specifications ensure all Ethereum clients maintain consensus by following identical rules for processing transactions and updating state.