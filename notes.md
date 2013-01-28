TODO:
* chop off some graphs of the evaluation

## Context & Background ##############################################

### Data Intensive Computing Background

* the volumn of processing data is growing exponentially
  - more than a T per day (common)
* some example:
  - google search
  - tweets push

### MapReduce vs. Parallel DBMS

* Facebook, Google;  Twitter

* brief intro to Parallel DBMS
  - database management system allows you to store, modify, extract
	information from a database
  - parallel: use parallelization of operations to improve the
	perfomrnance

* talk breifly about the literature of the debate

* concreate comparison
  - structured data
    - DBMS says it's good:
	  - preventing data corruption (DBMS enforce integrity)
	  - seperation from application --> portability, easy for data
		sharing
	- MR
	  - no explicit schema
	  - but the data structure is programmed logically
  - programming language
    - DBMS
	  - high-level, descriptive
	  - but has far less expressive power 
	  - thus Ruby on Rails etc. 
	- MR
	  - low-level, procedural
	  - powerful to do complicated caculation (but rare)
	  - Pig, Hive
  - fault tolerance
    - DBMS
	  - enforce data integrity
	  - do not save intermediate data on disk
	  - fails --> whole transaction fails, redo (for transactional
		workloads)
	  - efforts to improve it: pipelining intermediate result, check
		pointing (but harm performance)
	- MR
	  - have intermedate data stored on disk
	  - fails --> part of the computation redo
	  - rely on the DFS underneath
	- fact:
	  - DBMS does not scale as MR (<100 nodes) --> fewer failure
	  - DBMS require homogeneity
  - performance
    - unfair competition
	- depend largly on implementation

* small examples to illustrate the comparison
  - TODO


### Hybrid System

* Combine DBMS and MR to one, trying to approximate the efficiency of
  Parallel DBMS and the scalability, flexibility and fault tolerance of
  Map Reduce

* basic idea:
  - connect multiple single node database systems using Hadoop as task
	coordinator and network communication layer
  - actual computing work is done in a DBMS

* start-up company begin to take this project 


### Drawbacks in MapReduce


## Sailfish Basic ####################################################

### Reducing #disk_seeks

* in the reduce phase! no the map phase
* ideal situation vs practical usage

### I file Abstraction and API

* supprot three record-based I/O operations
  - create: return a fd
  - apending
  - scan: for the reducer to retrieve records; key range

* I file picture
  - an I file is divided to fix-sized (but configurable) chunks
  - a mapper is bound to multiple chunks in corresponding I files, which
	resides in different chuncknode
	- hash(key)
  - an I file is bound to some writers (namely, mappers)
    - constrained number of writers -> max parallelism

* how to append
  - allocate
  - send the record
  - assign offset, buffer in mem
  - ack
  - failure detection

* atomic appending
  - lock-free

* retrieve
  - augment each chunk with an index
  - sparse -> lower overhead
  - an index entry for every 64KB
  - the first level when looking up data in the chunk 
    (rather than scanning the whole file)
  
* Implementation issues
  - in KFS (DFS supporting high throughput access to large files)
  - no replication in this paper
  - record size <= chunk size (128MB)


### Handling Skew

* problem:
  - blocking
  - sorting in the map phase --> weakest link effect
    e.g the intermediate data is too big to be held in memory, it needs
	multi-pass sort to generate the output file

* sol: decouple the sorting from map task
  - aggregate --> sort (outside the context of map) --> index
  - pros:
    - use the index to self-tune: dynamic; data-dependent
    - programmers do not need to tune (#parameters: -4)
	

## Dataflow ##########################################################



## Evaluation ########################################################

* Cluster Setup

* Synthetic Benchmark
  - job description
    - no job input/output
	- similar to a benchmark used in the sorting alg competition
	- no skew
  - x-axis: volumn of intermediate data (1TB-64TB)
  - explain the figure9:
    - concurrent retrieval
	  - Hadoop: 30 machines by default
	  - Sailfish: all chunks in the I-file (accross several machines)
	- efficiency of each retrieval: data read per retrieval --> disk
	  throughput
	- why shrinks?
	  - Hadoop: data read per seek proportional to 1GB/R
	  - Sailfish: fixed #ifiles; increasing #chunks_in_ifile
	- performance gains from reducing #I/O
  - overheads of sorting
    - 128MB of data
    - radix trie
    - several seconds
  - impact of network transfer
    - in the map task: intermediate data is committed to remote RAM
    - higher 10% than Hadoop
	- high intra-rack connectivity (1Gps); 
	  and will be increased in the future (10Gps)

* Actual Job Mix
  - taxonomy
  - explain the graph
  - performance gain
    - using i-files for aggregation: except LogProc, LogRead
	- decoupling: address the skew (LogProc, NdayModel)
	  - Hadoop is slow whereever there is data skew
	- dynamic: #reducers depend on data volumn


## Issues ############################################################

* HDFS does not support concurrent writes
  - is the performance gain actually come from the concurrent append?
  - just another viewpoint of batching I/O

* Intermediate data are written twice, read twice

* Doubt the impact of data loss
