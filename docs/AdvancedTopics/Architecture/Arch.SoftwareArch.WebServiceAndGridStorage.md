# Web Service and Grid Storage

## Web Service (WS) Module

CM-Well supports a REST API for reading, writing, streaming and querying infotons, and more. The WS component is responsible for managing API requests and responses. The WS uses the 3rd-part Netty package and asynchronous IO to manage requests and responses. 

!!! note
	The Netty package is used for IO instead of the more standard Akka HTTP package, because Netty handles the scenario of many short-term connections better than Akka (whose connection setup time is too long for this case).

Requests to the WS module are served by a small thread pool, whose size is adjusted according to the machine's parallelism capability. Read and write operations are handled separately asynchronously, due to their different costs in processing time. Read requests are served directly by the WS JVM. Write requests undergo minimal analysis and verification, and are then transferred to the BG JVM for background processing. Some lengthier/more-complex processes, e.g. SPARQL queries, are handled by the Analytics JVM.

## Grid Storage (GS) Logic Layer

The GS-Logic module provides a unification layer for the different storage engines (each is also abstracted, and can be potentially be replaced transparently), and provides a unified API to the higher-level modules. Each of the storage engines has its own driver for interfacing with the 3rd-party storage package. These drivers hide the implementation details and dependencies of the specific underlying 3rd-party storage packages, which are Apache Cassandra for data storage ("direct") and ElasticSearch ("inverted") for field indexing and search queries. Storage requests are sometimes handled by one engine and sometimes by both, depending on the specific request and implementation interplay. Since the entire software stack is installed on each machine, both engines are quickly accessible. The design of the GS-Logic layer also enables expansion and addition of additional databases in the future (if there should prove to be a gap in functionality or insufficient performance in the existing ones).
