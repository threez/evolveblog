---
layout: post
title: Nonblocking IO
tags: io nonblocking blocking nodejs ruby thin eventmachine javascript
---

I'm working on a little project, where i get many requests from a separate server system. These requests are like events, they contain event data. The application server that processes these events must be able to scale horizontally. Therefore i need some kind of database. I decided to use [memcached](http://memcached.org/), because the data are less than 1 MB and don't need to be persistent but fast.

I was wondering which solution would bring me best performance results. I don't care the memory usage but for cpu utilization. I searched the web, but there was not suitable test. So i did it on my own. I wanted to compare *Ruby 1.8.7*, *Ruby 1.9.2* and [NodeJS](nodejs) with blocking and nonblocking calls.

These are the applications, frameworks, libraries and versions I used:

    ruby 1.9.2p0 (2010-08-18 revision 29036) [x86_64-darwin10.5.0]
    Thin 1.2.7
    Sinatra 1.1.0
    memcache-client 1.8.5
    memcached VERSION 1.2.8
    
    ruby 1.8.7 (2009-06-12 patchlevel 174) [universal-darwin10.0]
    Thin 1.2.7
    Sinatra 1.1.0
    memcache-client 1.8.5
    memcached VERSION 1.2.8
    
    Node 0.2.5
    Memcache LIB: https://github.com/elbart/node-memcache at 07bb1d4eed1d7e083d54

I'm running a normal MacBook with 2GHz Intel Core 2 Duo and 2GB 1067 MHz DDR3 Ram.

For benchmarking i used [Apache Bechmark (ab)](http://httpd.apache.org/docs/2.0/programs/ab.html) with 170 concurrent requests and a total count of 1000.

    ab -n 1000 -c 170 http://127.0.0.1:4567/test
    
The *memcached* was just started with default values. I filled the *memcache* with one big (nearly 1MB) data record called test.

{% highlight ruby %}
require "memcache"
m = MemCache.new "localhost"
m["test"] = File.read("test.b64")[0..(1024 * 1000)];
{% endhighlight %}

The nonblocking implementation in *Ruby* looked like this:

{% highlight ruby %}
AsyncResponse = [-1, {}, []].freeze

def memcache
  @@memcache ||= EM::P::Memcache.connect 'localhost', 11211
end

get "/:name" do
  name = params[:name].to_sym
  EventMachine.next_tick do
    memcache.get(name) do |value|
      env['async.callback'].call [200, {}, [value]]
    end
  end
  AsyncResponse
end
{% endhighlight %}

The blocking implementation in *Ruby*:

{% highlight ruby %}
CACHE = MemCache.new 'localhost'

get "/:name" do
  name = params[:name]
  CACHE.get(name)
end
{% endhighlight %}

The *NodeJS* implementation, and obviously there is no blocking:

{% highlight javascript %}
var memcache = require('./lib/memcache');

mcClient = new memcache.Client();
mcClient.connect();

http.createServer(function (request, response) {
  response.writeHead(200, {'Content-Type': 'text/plain'});
  var pathname = url.parse(request.url).pathname;
  
  mcClient.get(pathname.substr(1, pathname.length), function(data) {
    response.end(data);
  });
}).listen(8124);
{% endhighlight %}

After running the benchmark i get the following results:

<img src="/images/posts/2010-12-11-summary.png" alt="the graphs">

The results are pretty impressive and didn't meet my expectations. I expected *Ruby 1.9.2* and *NodeJS* to be at eye level and the nonblocking calls to be faster. But in fact the blocking calls are faster and one will get more throughput! The non-blocking calls alternatives are up to 5.5x times slower that the nonblocking calls (*Ruby 1.9.2*). The *NodeJS* is extremely slow at the moment a only 3 requests/s. I expect better results in te future. The *Ruby 1.8.7* beats the *Ruby 1.9.2* when using nonblocking calls. I was wondering why that happens - I have no idea, but i guess that EventMachine is not tuned for it. 

**As a conclusion i wouldn't advice to use *EM-memcache* or *NodeJS* in this situation.**

Notes:

- The nonblocking calls in ruby use a memcache pool, so that they can handle multiple resquests. The blocking alternative is just using one connection per process.
- Programming with the blocking is harder and more complex that with normal blocking calls
- In ruby one can benefit from the memcache implementationsin *C*. This would be an option for *NodeJS* too.
- *Ruby 1.9.2* and *EventMachine* maybe don't play well together at the moment.
- [Thin](http://code.macournoyer.com/thin/) is an awesome webserver
- The nonblocking calls utilize all the cpu power available, the blocking not necessarily.
- Ruby grows extremely in memory (up to 700MB in my tests), where *NodeJS* is consuming nearly nothing.
- Is the pooling of *memcached* connections optimal for many requests that do nothing? For example in a comet server situation.
- Because memcached is so fast and there are no long running calls, one won't get the benefit of the nonblocking model.

I also tried mongrel, but the results where so bad, that i did't mention them here. I guess [passenger](http://www.modrails.com/) and [unicorn](http://unicorn.bogomips.org/) may be also good alternatives.

I think if the clients are permanently connected the *EventMaschine* approach is much better, even if it is slower in high load situations. Just because one call handle thousands of users with a single box.

You can find the full source and the results [here](https://github.com/threez/memcache-broker).

These Post is inspired by the blog [ post](http://macournoyer.com/blog/2009/06/04/pusher-and-async-with-thin/) of the *Thin* creator and a talk from [Mike Perham](http://www.mikeperham.com/2010/01/27/scalable-ruby-processing-with-eventmachine/).

<iframe src="http://player.vimeo.com/video/10849958?portrait=0" height="300" frameborder="0">.</iframe>
