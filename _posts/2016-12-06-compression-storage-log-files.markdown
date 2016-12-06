---
title: Compression & data storage for log files
layout: post
---

As the internet grows, and the applications we build increase in complexity by utilising user data to make smarter more personalised decisions, storing and processing logs from "Log File Applications" is an increasing challenge.

## Real Time Bidding

Online real time advertising systems are wholly reliant on the user logs they collect both from their own activities, as well as the ingestion of logs from other providers. This data is then modelled to create subsets of users, find patterns in domain activities or even identify fraudulent bot traffic.

It is most important in this industry to store and process these logs in such a manner as to reduce on computation, transfer and storage costs. 

### Compression

Below is a nice graph from Yahoo about how the different common compression algorithms rank in compression ratio vs compress/decompress time.

![space time tradeoff compression options]({{ site.github.url }}/images/yahoo-compression.png "Space-Time Tradeoff Compression")

As can be expected there is a severe tradeoff between performace of compression ratio and CPU utilization. Log files are often written once, but used multiple times therefore the decompression hit can be significant. When analysing the log files in HIVE or Hadoop, there is also an understanding of "splittable" which should be considered allowing parallelization for decompression via map/reduce. 

![compression options in hadoop]({{ site.github.url }}/images/yahoo-compression-algos.png "Compression Options in Hadoop")

