# JustBid bidding system [![Build Status](https://travis-ci.org/LouisChang/JustBid.svg?branch=master)](https://travis-ci.org/LouisChang/JustBid)
* Insight Data Engineering Fellow Project


## Pipeline

The pipeline is designed for both batch and real-time processing. The data is generated from EventSim Generator, which is used for simulating people's behavior of listening to music. I changed the format of the data and make it useful for my system. Kafka is used as a queue/broker and Zookeeper is responsible for distributing data in queue to their customers.

![alt tag](pics/pipeline.png)

## Real-time processing stage
* Streaming

  The first stage is real time streaming with Spark Streaming. I use Scala to access data from Kafka and clean the data, get item_id, user_id, bid_price and also the timestamp from Kafka, with the Cassandra Connector, after processing, data is stored in Cassandra NoSQL Database.
* Database

  Cassandra is used as my database. I need to handle millions of data and need to SELECT bid with the same item_id when I choose item_id as the PRIMARY KEY. Also, I do not need to use JOIN function of different TABLES. Last but not least, timestamp is needed to store and used to sort, so Cassandra is my top choice.
* UI

  Flask is used for data display. The website is no longer live on because the web server of Insight is down and [video demo](https://vimeo.com/173849615) is ready on Vimeo.
  
## Batch processing stage
  * Jaccard Similarity 
  
    Jaccard Similarity is widely used for finding similar items or finding similarity between several sets. It is quite simple to understand and quite simple to apply.

    <img src="pics/jaccard.png" height="300" align="middle" />
    
    We can see two sets S and T, the SIM(S,T) = 3/5+6-3 = 3/8.
  
  * MinHash
  
    Though Jaccard Similarity is simple, easy to apply. But when we need to calculate between large amount of sets, each set needs to be calculated with the other sets once to get the most similar set to itself which is quite expensive in time aspect. So MinHash algorithm is implemented in my system to do the batch processing in Spark. This algorithm is introduced in Stanford [CS246](http://web.stanford.edu/class/cs246/) class and the [Mining of Massive Datasets book](http://infolab.stanford.edu/~ullman/mmds/book.pdf).

    Here is an example:
    
    <img src="pics/minhash1.png" height="300" align="middle" />
    
    Assume we have five items and four users and name them starting from 1 respectively. And I give two Hash Function examples, the first Hash Function is x+1 mod 5, the second one is 3x+1 mod 5 which can give me a permutation of numbers between 0 and 4.
    
    <img src="pics/minhash2.png" height="300" align="middle" />
    <img src="pics/minhash3.png" height="300" align="middle" />
    <img src="pics/minhash4.png" height="300" align="middle" />
    
    Then look at User 1, he bids item 1 and 4 in Row 0 and 3, the corresponding number in Hash function 1 is 1 and 4, 1 and 0 in Hash function 2. Then we need to get the minimum one and update in the new Signature matrix which is shown below.
    
    <img src="pics/minhash5.png" height="300" align="middle" />
    
    From the Signature Matrix, we can see the highest similarity for user 1 is user 4 with SIMILARITY = 1 which is obvious in my matirx, the real SIM(1,4) = 2/3. When we increase the number of Hash functions, it will become more accurate. And this algorithm will highly decrease the dimension of the matrix because the number of items is much larger than number of hash functions which is between 1 and 400. Also, Hash Function generator and calculation can be done in distributed system such as Hadoop or Spark.
    
  * HDFS
  
    I got the real-time streaming simulated data from Kafka and use [Gobblin](https://github.com/linkedin/gobblin) to put that into HDFS, then I run the Spark Job and get the data from my HDFS.

## Real-time stage data performance evaluation

  * Data Info
    The simulator will simulate an initial 10k bidder with a increase rate of 30% per year, almost 15 million bidding is simulated if we choose the program to run a 10-year period data.
  * AWS Info
    Four clusters are used, the master is m4.xlarge, the three worker is m4.large. Those clusters are almost enough for running all my program so I do not increase all the worker to m4.large.
  * UI update Info
    Real-time stage is running through Spark Streaming, all the data will be stored in Cassandra and flask UI will refresh(access) Cassandra every per second to update.

## Batch stage performance evaluation

  * A small sample test based on MinHash has been done on Error Rate and running time has been done.
    
    <img src="pics/accuracy.jpg" height="300" align="middle" />
    <img src="pics/running.jpg" height="300" align="middle" />

    From the two figures above, we can see the more hash functions we use, the less error rate we will have but the running time will be higher. To achive a tradeoff, we need to make hash functions close to 100-150.
  

    
