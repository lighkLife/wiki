# MapReduce

## 1. `MapReduce`是什么？

**programing model** and an associated **implantation** for processing and generating large data sets.

## 2. `MapReduce`解决了什么问题？

Simplified Data processing on Large Cluster.

## 3. `MapReduce`怎么解决的？

Users specify a map function that processes a key/value pair to generate a set of intermediate key/value
pairs, and a reduce function that merges all intermediate values associated with the same intermediate key.

## 4. `MapReduce`由包含哪些组成部分？

   - partitioning the input data
   - scheduling the program's execution across a set of machines
   - handing machine failures
   - managing the required inter-machine communication. 

## 5. Example

   - counting the number of occurrences of each word in a large collection of documents.
   - distributed grep
   - count of URL access frequency 
   - reverse web-link graph
   - term-vector per host

## 6. Execution Overview

![image-20220128145701204](https://lighk.oss-cn-beijing.aliyuncs.com/wyny/image-20220128145701204.png)

