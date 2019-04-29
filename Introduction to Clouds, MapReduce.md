## Introduction to Clouds, MapReduce

#### Introduction to Clouds

**Categories of Clouds**

1. Private cloud
   Only accessible to company employees

2. Public cloud

   Provide service to any paying customers

   * AWS S3
   * AWS EC2
   * Google App Engine

**Definition**

**Cloud = Lots of storage + compute cycles nearby**

* Single-site cloud consists of 
  * Compute nodes
  * Switches, connecting the racks
  * A network topology
  * Storage nodes connected to the network
  * Front-end for submitting jobs and receiving clients requests
  * Software Services
* A geographically distributed cloud consists of
  * Multiples such sites
  * Each site perhaps with a different structure and services

**Definition of Distributed system**

A distributed system is a collection of entities, each of which is autonomous, programmable, asynchronous and failure prone, and which communicate through an unreliable communication medium



#### Map Reduce

**MAP**: Parallelly process a large number of individual records to generate intermediate key/value pairs.

**REDUCE**: Reduce processes and merges all intermediate values associated per key

* Each key assigned to one Reduce
* Parallelly Processes and merges all intermediate values by partitioning keys
* Popular: *Hash* partitioning, i.e., key is assigned to reduce # = hash(key) % number of reduce servers

Externally: For user

1. Write Map program (short), write a Reduce program (short)
2. Submit job; wait for result
3. Need to know nothing about parallel/distributed programming!

Internally: For the Paradigm and Scheduler

1. Parallelize Map
2. Transfer data from Map to Reduce
3. Parallelize Reduce
4. Implement Storage for Map input, Map output, Reduce input, and Reduce output.

**(Ensure that no Reduce starts before all Maps are finished. That is, ensure the barrier between the Map phase and Reduce phase)**

The shuffle phase between the map phase and the reduce phase can be started parrellely while map tasks are still continuing.



For the cloud:

1. Parallelize Map: easy! each map task is independent of the other!
   * All Map output records with same key assigned to same Reduce
2. Transfer data from Map to Reduce
   * All Map output records with same key assigned to to same Reduce task
   * use partitioning function. e.g., hash(key)%number of reducers
3. Parallelize Reduce: easy each reduce task is independent of the other !
4. Implement Storage for Map input, Map output, Reduce input, and Reduce output