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

Externally: **For user**

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

**For the cloud**:

1. Parallelize Map: easy! each map task is independent of the other!
   * All Map output records with same key assigned to same Reduce
2. Transfer data from Map to Reduce
   * **All Map output records with same key assigned to to same Reduce task**
   * **use partitioning function. e.g., hash(key)%number of reducers**
3. Parallelize Reduce: easy each reduce task is independent of the other !
4. **Implement Storage for Map input, Map output, Reduce input, and Reduce output**
   * **Map input:** from distributed file system
   * **Map output:** to local disk (at Map node); uses local file system
   * **Reduce input:** from (multiple) remote disks; uses local file systems
   * **Reduce output:** to distributed file system

**local file system** = Linux FS, etc

**distributed file system** = GFS (Google File System), HDFS (Hadoop Distributed File System)

![Screen Shot 2019-04-28 at 5.29.42 PM](/Users/Zack/Desktop/Screen Shot 2019-04-28 at 5.29.42 PM.png)



#### YARN Scheduler

* Used in Hadoop 2.x +
* YARN = Yet Another Resource Negotiator
* Treats each server as a collection of *containers*
  * Container = some CPU + some memory
* Has 3 main components
  * **Global Resource Manager (RM)**
    * Scheduling
  * **Per-server Node Manager (NM)**
    * Daemon and server-specific functions
  * **Per-application (job) *Application* Master (AM)**
    * Container negotiation with RM and NMs
    * Detecting task failures of that job

![Screen Shot 2019-04-28 at 6.25.02 PM](/Users/Zack/Desktop/Screen Shot 2019-04-28 at 6.25.02 PM.png)



#### Fault Tolerance

* Server Failure
  * NM heartbeats to RM
    * If server fails, RM lets all affected Ams know, and Ams take action
  * NM keeps track of each task running at its server
    * If task fails while in-progress, mark the task as idle and restart it
  * AM heartbeats to RM
    * On failure, RM restarts AM, which then syncs up with its running tasks
* RM Failure
  * Use old checkpoints and bring up secondary RM
  * Heartbeats also used to piggyback container requests
  * Avoids extra messages

#### Slow servers

Stragglers (slow nodes)

* The slowest machine slows the entire job down
* Due to Bad Disk, Network Bandwidth, CPU, or Memory
* Keep track of "progress" of each task (% done)
* Perform backup (replicated) execution of straggler task: task considered done when first replica complete. Called Speculative Execution.

#### Locality

* Since cloud has hierarchical topology (e.g., racks)
* GFS/HDFS stores 3 replicas of each of chunks (e.g., 64 MB in size)
  * Maybe on different racks, e.g., 2 on rack, 1 on a different rack
* Mapreduce attempts to schedule a map task on 
  * a machine that contains a replica of corresponding input data, or failing that
  * on the same rack as a machine containing the input, or failing that
  * Anywhere