---
title: Compression & data storage for log files
layout: post
tags: data hive logs
---



As the internet grows, and the applications we build increase in complexity by utilising user data to make smarter more personalised decisions; storing and processing logs from "Log File Applications" is an increasing challenge.

## Real Time Bidding

Online real time advertising systems are wholly reliant on the user logs they collect both from their own activities, as well as the ingestion of logs from other providers. This data is then modelled to create subsets of users, find patterns in domain activities or even identify fraudulent bot traffic.

It is most important in this industry to store and process these logs in such a manner as to reduce on computation, transfer and storage costs. 

### Compression

Below is a nice graph from Yahoo about how the different common compression algorithms rank in compression ratio vs compress/decompress time.


![space time tradeoff compression options]({{ site.github.url }}/images/yahoo-compression.png "Space-Time Tradeoff Compression")


As can be expected there is a severe tradeoff between performace of compression ratio and CPU utilization. Log files are often written once, but used multiple times therefore the decompression hit can be significant. When analysing the log files in Hive or Hadoop, there is also an understanding of "splittable" which should be considered allowing parallelization for decompression via map/reduce. 


![compression options in hadoop]({{ site.github.url }}/images/yahoo-compression-algos.png "Compression Options in Hadoop")


So what is the best way to compress your data for consumption in "Big Data Applications" such as Hadoop?

### Format Choices

Sequence Files (along with Map Files) are a binary format which is splittable. The main use is to club your small log files into larger files, Hive uses two formats RCFile and ORCFile which can be queried across multiple rows and have a columnar layout inside. This is the best option if your logs are mainly analysed within Hive or derivative applications.

Keeping data compressed in Hive tables has shown to help with performance for most workloads, when looking at both disk and CPU. Text Files can be imported and the both gzip and bzip2 files will be decompressed on the fly during query execution. For example: 

```sql
CREATE TABLE my_table (line STRING)
   ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' LINES TERMINATED BY '\n';
 
LOAD DATA LOCAL INPATH '/app/weblogs/20161206-clicks.log.gz' INTO TABLE my_table;
```
The new table `my_table` is stored as TextFile which is the default Hive storage format, but this is not a "splittable" format so maps cannot run in parallel, which means the mapping power of your cluster is not fully utilized.

The Hive documentation recommends inserting data into another table, which is stored as a SequenceFile and can then be split by Hadoop and distributed, for example:

```sql
CREATE TABLE my_table (line STRING)
   ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' LINES TERMINATED BY '\n';
 
CREATE TABLE raw_sequence (line STRING)
   STORED AS SEQUENCEFILE;
 
LOAD DATA LOCAL INPATH '/app/weblogs/20161206-clicks.log.gz' INTO TABLE my_table;
 
SET hive.exec.compress.output=true;
SET io.seqfile.compression.type=BLOCK;
INSERT OVERWRITE TABLE raw_sequence SELECT * FROM my_table;
```
This method should allow for a much speedier decompression when executing queries on large data sets running on multi node clusters.
