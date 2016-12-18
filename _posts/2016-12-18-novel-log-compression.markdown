---
title: Novel Log Compression Algorithm
layout: post
tags: compression logs algorithm
published: True
---

Log files are usually plain text files, in which every single line corresponds to a single event which has been "logged" by the application. Each event in usually described by at least a few tokens, separated usually by spaces or tabs. The most common logged tokens would be; IP address, date, time, unix ts, URL, user-agent etc

Typically lines in log files are very similar, as for example a single user visits a webpage and performs multiple actions on the same URL, from the same IP on the same day. The simplest transform for a compression algorithm dealing with such data would be to replace tokens with pointers to the previous line where they match. In large scale web logs I feel we can do better by evaluating more than just two lines of data at once, even though this increases the corresponding memory overhead.

### Online Compression

Consider the following two lines:
```
89.123.105.255 [12/Dec/20016:07:09:53 +0200] "GET /images/thumbnail_2.jpg"
89.123.105.255 [12/Dec/20016:07:09:54 +0200] "GET /images/new_car_66.jpg"
```
An online algorithm which is tackling the stream of logs as they are received by the machine, can easily begin to compare the current line with the previously compressed. The algorithm matches characters from the line beginning, which in the example above equates to 36 chars including spaces:
```
89.123.105.255 [12/Dec/20016:07:09:5
```
Therefore we are now left with, the below whilst the first chunk is encoded as 128+36 which is 164.
```
3 +0200] "GET /images/thumbnail_2.jpg"
4 +0200] "GET /images/new_car_66.jpg"
```
The first unmatched char is left up to the first space and the process is repeated, this time matching 21 chars and encoding them as 149. No more spaces left in the log entry therefore the compression is complete and our log entry now looks like the below:
```
(164)4(149)new_car_66.jpg"
```
Which means the new compressed line contains 27 instead of 74 characters, which equates to a circa 64% compression.
