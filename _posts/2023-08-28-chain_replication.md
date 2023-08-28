---
title: Chain_replication
author: highcloud100
date: 2023-08-28 16:01:12 +0900
categories: [ mit6.5840 , Distributed_Systems ]
tags: [chain_replication , Distributed_Systems]
render_with_liquid: false
---

# Chain Replication for Supporting High Throughput and Availability
https://pdos.csail.mit.edu/6.824/papers/cr-osdi04.pdf

## Intro

- One challenge when building a large scale storage service is maintaining "high availability" and "high throughput"
  - despite failures and concomitant changes to the storage service's configuration
  
- Consistency guarantees also can be crucial but ...
  - large-scale storage service are not incompatible with high throughput and availability.
  - ex) GFS declined to support strong consistency

$$\text{Strong consistency} \propto {1 \over \text{System throughput or availability}}$$

- new chain replication
  - coordinating fail-stop servers
  - simultaneously supports high throughput, availability, and strong consistency.


## Storage service interface 


![[Pasted image 20230828200235.png]]