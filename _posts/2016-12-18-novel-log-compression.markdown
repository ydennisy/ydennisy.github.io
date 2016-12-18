---
title: Novel Log Compression Algorithm
layout: post
tags: compression logs algorithm
published: true
---

Log files are usually plain text files, in which every single line corresponds to a single event which has been "logged" by the application. Each event in usually described by at least a few tokens, separated usually by spaces or tabs. The most common logged tokens would be; IP address, date, time, unix ts, URL, user-agent etc

Typically lines in log files are very similar, as for example a single user visits a webpage and performs multiple actions on the same URL, from the same IP on the same day. The simplest transform for a compression algorithm dealing with such data would be to replace tokens with pointers to the previous line where they match. In large scale web logs I feel we can do better by evaluating more than just two lines of data at once, even though this increases the corresponding memory overhead.

### Online Compression

Online compression is the process each in which compression is carried out on a live stream of data, as opposed to offline compression which would equate to a batch based methodology. The combination of these two approaches will most of the time deliver the best compression ration possible on a set of logs. Consider the following two lines:

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

### Less Repetitive Logs

If the logs being compressed are less repetitive in nature, the second variant of the proposed algorithm helps to deal with this, by using a reference not to the previous line but to a block of them, usually 16. The index of the selected line is encoded as a single byte at the beggining of each line.

### Offline Compression

Having completed the above steps on streaming logs we can apply a regular dictionary based method on batches of log files to complete our transformation. We can use generalist algorithms such as `gzip` or go one better and employ a custom dictionary approach which also encodes some frequently occuring types of data in weblogs.

For example, dates in YYYY-MM-DD format can be denoted with a two byte long interger whose value is the difference in days from a specific point in time 1990-01-01 for example. IP addresses are replace with a flag representing this data type and excoded as a sequence of four bytes representing each octet.

### Conclusion

I will be looking to complete implementation and testing of this approach and will be happy to share the script so you too can test on your weblogs!
