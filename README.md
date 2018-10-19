# TurtlePay:tm: Service Delivery Architecture

## Objective

This document provides the working standard for the infrastructure design and service delivery model of the TurtlePay:tm: project. While this document does serve as an outline and model for how the services will be designed and ultimately provided, it is subject to change without notice and actual service delivery models may differ from this document.

Our goal; however, is to maintain concurrency between the service platform and this document for all to enjoy.

## Infrastructure

Connecting to the TurtleCoin:tm: network requires utilizing the `TurtleCoind` binary to provide core services. As this is our gateway to the network, the stability, reliability, and reachability of the node services is one of the most important things for TurtlePay:tm: services.

To ensure [99.999%](https://en.wikipedia.org/wiki/High_availability#Percentage_calculation) or higher uptime for the services the infrastructure must be designed with high-availability in mind. To accomplish this, multiple redundant layers including [inter-process communication](https://en.wikipedia.org/wiki/Inter-process_communication) between those layers is required.

### Node Deployment

Nodes will be deployed in localized clusters. The clusters will consist of not less than 3 nodes at any given time. For the highest availability on the current TurtleCoin:tm: software, Microsoft Windows hosts will be used.

Each cluster will consist of the nodes and a high-availability load balancer that verifies the state of the node as suitable. Each load balancer is charged with making sure that the cluster is always available.

***Note:*** TurtlePay:tm: and the underlying wallet(s) used for the service will, at no point, directly communicate with **any** nodes.
