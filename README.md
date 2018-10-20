# TurtlePay™ Service Delivery Architecture

## Objective

This document provides the working standard for the infrastructure design and service delivery model of the TurtlePay™ project. While this document does serve as an outline and model for how the services will be designed and ultimately provided, it is subject to change without notice and actual service delivery models may differ from this document.

Our goal; however, is to maintain concurrency between the service platform and this document for all to enjoy.

## Infrastructure

Connecting to the TurtleCoin™ network requires utilizing the `TurtleCoind` binary to provide core services. As this is our gateway to the network, the stability, reliability, and reachability of the node services is one of the most important things for TurtlePay™ services.

To ensure [99.999%](https://en.wikipedia.org/wiki/High_availability#Percentage_calculation) or higher uptime for the services the infrastructure must be designed with high-availability in mind. To accomplish this, multiple redundant layers including [inter-process communication](https://en.wikipedia.org/wiki/Inter-process_communication) between those layers is required.

### Node Deployment

Nodes will be deployed in localized clusters. The clusters will consist of not less than 3 nodes at any given time. For the highest availability on the current TurtleCoin™ software, Microsoft Windows hosts will be used.

Each cluster will consist of the nodes and a high-availability load balancer that verifies the state of the node as suitable. Each load balancer is charged with making sure that the cluster is always available.

***Note:*** TurtlePay™ and the underlying wallet(s) used for the service will, at no point, directly communicate with **any** nodes.

### Blockchain Database Storage System (BDSS)

To store the blockchain, multiple candidate database systems are being evaluated.

Core factors to consider in database system selection include [ACID](https://en.wikipedia.org/wiki/ACID_(computer_science)) properties:

* Atomicity:
* Consistency:
* Isolation:
* Durability:

Candidate systems include, but are not limited to:

* [ScyllaDB](https://www.scylladb.com/)
* [Cassandra](http://cassandra.apache.org/)
* [Amazon Elasticache](https://aws.amazon.com/elasticache/)
* [Amazon Redshift](https://aws.amazon.com/redshift/)
* [Google Cloud SQL](https://cloud.google.com/sql/)

***Note:*** The selection of a standard cloud provider for this portion of the service is acceptable as the information contained within their data store is **public** knowledge that exists already on the blockchain.

The highly scalable database will be used in lieu of working directly with the TurtleCoin™ software for a number of reasons, including:

* Provides a [DMZ](https://en.wikipedia.org/wiki/DMZ_(computing))-style buffer between the TurtleCoin™ network and the TurtlePay™ services
* Provides an [abstraction layer](https://en.wikipedia.org/wiki/Abstraction_layer) that does not rely on the underlying node software
* Provides faster random blockchain access than the node software permits at a much larger scale

### Standard Services Database

TurtlePay™ requires the same level of database control and ACID properties as the blockchain storage for managing and maintaing information such as the following:

* Developer Account Services
* Application Endpoint Information
* Transaction Information Through the Service
* Audit Trails & Logs as Required
* Debugging, Troubleshooting, & Support Information

## Software Services

### Blockchain Data Collection Agent (BDCA)

To mitigate any issues with the node software, data (blocks & transaction mempool) will be collected from the clustered regions at standard intervals. 

The BDCA(s) will poll the clusters for changes in the blockchain including re-organization events, new blocks, and transaction memory pool changes. Each instance of the BDCA will exist as close to the clusters as possible to minimize latency in the polling of the node.

This information, once collected, will be pushed into the BDSS for further consumption by TurtlePay™ services.

### Blockchain Relay Agent (BRA)

Each BRA is tasked with ensuring that new transactions from the TurtlePay™ platform are properly relayed to the TurtleCoin™ network for processing. Each transaction meant for network consumption is queued by the TurtlePay™ platform before being broadcast to the TurtleCoin™ network. The BRA is charged with verifying that the transaction has been accepted by a node and providing the state of such back to TurtlePay™.

***Note:*** Multiple outgoing transfers from the TurtlePay™ platform may be combined into a single network transaction to make the most efficient use of block space.

### TurtleCoin™ Wallets

As with any blockchain system, a wallet must be maintained. The core functionality of a wallet is as follows:

* Hold public & private keys
* Scan blockchain transactions
  * Retrieve outputs meant for our keys
  * Track spendable outputs
  * Create new inputs from spendable outputs
* Generate & Sign new transactions

Our wallet(s), instead of relying on a service such as `turtle-service` will live as native services that interact not with the node but with the data collected by the BDCAs. The wallet(s) will scan transactions and the transaction memory pool directly from the database.

Benefits of working with the data in the database include, but are not limited to:

* Provides a [DMZ](https://en.wikipedia.org/wiki/DMZ_(computing))-style buffer between the TurtleCoin™ network and the TurtlePay™ services
* Provides an [abstraction layer](https://en.wikipedia.org/wiki/Abstraction_layer) that does not rely on the underlying wallet software
* Fine grain atomic control of wallet operations
* Easy to scale and spin up on demand

***Note:*** Development of a [Node.js](https://nodejs.org/) native wallet is underway to support this effort.

