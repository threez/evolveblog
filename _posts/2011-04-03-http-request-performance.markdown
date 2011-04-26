---
layout: post
title: HTTP Request Performance
tags: http rest performance request ruby java io json curl
---

When you deal with a lot *HTTP* requests in your application, you want to have them as fast and light as possible. Also one want to take advantage of the *HTTP* keep alive functionality to reduce the number of reconnects to the hosting server if possible. Gzipping and other techniques are very welcome also. All this is especially interesting if your infrastructure is relying on this http communication heavily. *HTTP* is very good in providing a stable and flexibel solution, but sometimes suffers from the performance of native protocols.

The project I'm currently working on involves a lot of internal http communication. Therefore I wanted to have a look on the http client performance.
Because the client I used is implemented in *Ruby* I mostly compare ruby libs and versions. I also implemented a *Java* based http client implementation along with the benchmark to compare the results to a completely different setup. As a round up I added a test with [apache bench](http://httpd.apache.org/docs/2.0/programs/ab.html) and [httperf](http://www.hpl.hp.com/research/linux/httperf/) to see what a normal benchmark would tell us in time and throughput.

The tests consists of two things:

- The first step is to download a 16KB *json* asset from the remote server. The server in my tests is a separate system directly linked with a _cat.6 gigabit ethernet cable_. The remote is a 2.26 GHz Intel Core 2 Duo, 8 GB 1067 MHz DDR3. The main focus is on benchmarking library and stack efficiency.
- The second step is to parse these incoming data and grep a value and check this value for correctness. Sure, this is not possible in the case of *apache bench* and *httperf*.

For the json parsing i used the [yajl-ruby](https://github.com/brianmario/yajl-ruby) gem and included the *json* gem wrapper to leverage the *json* parser of *yajl* when possible and provided though the library. If the library doesn't have direct support for json parsing I implemented it by my self in a competitive manner.

In the *Java* world i used the [json simple](http://code.google.com/p/json-simple/) project from google code. The performance is nearly comparable to what ruby offers. I started the java test both in server and client mode.

I used the [apache](http://httpd.apache.org/) web server that is bundled with *MacOSX* (Version 10.6.7 - Apache/2.2.17).

I use *sessions* if the library provides it. The test will be executed _10.000_ times. 

The results and objective of the benchmark is to measure the user and system time used during the execution of the requests and the throughput that could be generated.

The whole test scenario can be found on [github](https://github.com/threez/test-http-clients). To run the test suite on your mashine simple run:

    make

The results I found out during my observations is pretty impressive. *Ruby* actually doesn't have to hide itself in these benchmarks when using the native libraries that are empowered using [libcurl](http://curl.haxx.se/libcurl/).
Here are the facts:

* using *Java* one can achieve around *7,3 MB/s* and approximately *460 requests/s*
* using *Ruby* around *13,9 MB/s* and approximately *876 requests/s* are possible

Native applications without json parsing:

* *apache bench* produces less throughput (*12,7 MB/s*) than *Ruby* and approximately *766 requests/s*
* the best results can be achieved using *httperf* around *18 MB/s* and approximately *1091 requests/s*

Ruby has a bunch of native http libraries that perform very well when doing a lot of blocking http calls. For me [curb](http://curb.rubyforge.org/) and [patron](https://github.com/taf2/curb) are the best choise. Even if they may miss some functionality (certificate authentication in patron) or convenience (curb sessions). Java has also good numbers but can not compete with the native libraries on the same level.

<img src="/images/posts/2011-04-26-throughput.png" alt="Throughput MB/s"/>
<img src="/images/posts/2011-04-26-real-time.png" alt="Real Time (s)"/>
<img src="/images/posts/2011-04-26-user-system-time.png" alt="User + System Time (s)"/>
<img src="/images/posts/2011-04-26-table.png" alt="All Data"/>

Here are the raw results:

    Execute http performance test using ruby ruby 1.9.2p180 (2011-02-18 revision 30909) [x86_64-darwin10.6.0]
      doing 10000 request in each test...
                              user     system      total        real
    testing curb          4.440000   1.000000   5.440000 ( 11.419920)
    testing curl         10.850000  43.560000 161.490000 (156.847936)
    testing httparty     47.390000   1.990000  49.380000 ( 57.106096)
    testing httpclient  138.780000  12.430000 151.210000 (208.142251)
    testing httprb       13.450000   1.650000  15.100000 ( 26.002268)
    testing net/http     12.400000   1.600000  14.000000 ( 20.534805)
    testing patron        5.670000   1.040000   6.710000 ( 12.757735)
    testing RestClient   15.220000   1.690000  16.910000 ( 23.757431)
    testing righttp       8.610000   2.630000  11.240000 ( 23.175959)
    testing rufus-jig    11.010000   2.080000  13.090000 ( 37.957054)
    Execute http performance test using ruby ruby 1.8.7 (2011-02-18 patchlevel 334) [i686-darwin10.6.0]
      doing 10000 request in each test...
                              user     system      total        real
    testing curb          5.540000   1.100000   6.640000 ( 13.137579)
    testing curl         12.900000  43.720000 158.450000 (172.777551)
    testing httparty     55.140000   3.380000  58.520000 ( 66.566792)
    testing httpclient  142.000000  13.380000 155.380000 (242.390312)
    testing httprb       17.690000   2.030000  19.720000 ( 42.169571)
    testing net/http     15.660000   1.910000  17.570000 ( 24.812459)
    testing patron        6.840000   1.420000   8.260000 ( 14.602333)
    testing RestClient   20.180000   2.120000  22.300000 ( 30.403473)
    testing righttp      12.110000   2.570000  14.680000 ( 28.235797)
    testing rufus-jig    12.540000   2.580000  15.120000 ( 31.141118)
    Time for 10000 requests to http://169.254.41.121/~vincentlandgraf/test.json:
                              user     system      total        real
    testing httpclient    7,832342   4,183709  12,016051 ( 21,852000)
    Time for 10000 requests to http://169.254.41.121/~vincentlandgraf/test.json:
                              user     system      total        real
    testing httpclient    7,978861   4,256037  12,234898 ( 21,662000)
    This is ApacheBench, Version 2.3 <$Revision: 655654 $>
    Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
    Licensed to The Apache Software Foundation, http://www.apache.org/

    Benchmarking 169.254.41.121 (be patient)


    Server Software:        Apache/2.2.17
    Server Hostname:        169.254.41.121
    Server Port:            80

    Document Path:          /~vincentlandgraf/test.json
    Document Length:        16621 bytes

    Concurrency Level:      1
    Time taken for tests:   13.043 seconds
    Complete requests:      10000
    Failed requests:        0
    Write errors:           0
    Total transferred:      169200000 bytes
    HTML transferred:       166210000 bytes
    Requests per second:    766.68 [#/sec] (mean)
    Time per request:       1.304 [ms] (mean)
    Time per request:       1.304 [ms] (mean, across all concurrent requests)
    Transfer rate:          12668.21 [Kbytes/sec] received

    Connection Times (ms)
                  min  mean[+/-sd] median   max
    Connect:        0    0   0.1      0       1
    Processing:     1    1   0.2      1      14
    Waiting:        0    1   0.1      1       9
    Total:          1    1   0.2      1      14

    Percentage of the requests served within a certain time (ms)
      50%      1
      66%      1
      75%      1
      80%      1
      90%      2
      95%      2
      98%      2
      99%      2
     100%     14 (longest request)
    httperf --client=0/1 --server=169.254.41.121 --port=80 --uri=/~vincentlandgraf/test.json --send-buffer=4096 --recv-buffer=16384 --num-conns=10000 --num-calls=1
    Maximum connect burst length: 1

    Total: connections 10000 requests 10000 replies 10000 test-duration 9.166 s

    Connection rate: 1091.0 conn/s (0.9 ms/conn, <=1 concurrent connections)
    Connection time [ms]: min 0.7 avg 0.9 max 3.8 median 0.5 stddev 0.2
    Connection time [ms]: connect 0.2
    Connection length [replies/conn]: 1.000

    Request rate: 1091.0 req/s (0.9 ms/req)
    Request size [B]: 93.0

    Reply rate [replies/s]: min 1086.9 avg 1086.9 max 1086.9 stddev 0.0 (1 samples)
    Reply time [ms]: response 0.4 transfer 0.3
    Reply size [B]: header 280.0 content 16621.0 footer 0.0 (total 16901.0)
    Reply status: 1xx=0 2xx=10000 3xx=0 4xx=0 5xx=0

    CPU time [s]: user 1.75 system 7.39 (user 19.1% system 80.6% total 99.7%)
    Net I/O: 18105.3 KB/s (148.3*10^6 bps)

    Errors: total 0 client-timo 0 socket-timo 0 connrefused 0 connreset 0
    Errors: fd-unavail 0 addrunavail 0 ftab-full 0 other 0
