---
layout: post
title: "Application Level Replication in Redis"
date: 2013-05-19 15:19
comments: true
categories:
- Redis
- nginx
- Lua
---
About 6 months ago, at $DAYJOB, we were retooling a site for a high traffic event.
In the past this site had relied primarily on nginx's built in cache. While that worked fine, I had a few reasons that I
wanted to explore a slightly different approach:
<!--more-->

* We were going to be deploying at least partially in the "cloud" and
  we've had various issues with disk IO there.
* We would need to be able to purge based on url, including partial
  url (prefix, extension, etc).
* I also wanted to explore some ideas to replicate the cache between
  nodes as the backend to this particular site had had some
  performance issues with many concurrent requests.

I did the initial protoype in nginx/Lua using redis as the cache. I
could have used ngx_lua's
[shared dictionaries](http://wiki.nginx.org/HttpLuaModule#ngx.shared.DICT)
but it may do disk IO (it's implemented as anonymous shared memory segments). I'd
also had some issues on a different project were the spin locks it
uses became a bottle neck and I knew that network IO did not block the
nginx process. I'd also have to write my own API interface for
selectively prurging items.

My initial implementation was fairly simple, as far as the cache was
concerned, but was suffeciently fast. Various testing showed that, for
this use case, two nginx workers and one redis process on a 2 cpu VM
seemed to give the best performance without "wasting" resources. I configured redis to never write
to disk and did a little minor tuning. I also had nginx talk to redis
over unix sockets - this gave a slight bump in over all
performance. **Note:** I really wished I had taken better notes, so I
apologize for speaking in very general terms about performance.

We decided, because of the time crunch, we'd just use native redis
calls for the selective cache purging. This was admittedly hacky, but
a `knife ssh` command with calls to `redis-cli` worked fairly well.

So, now I needed to tackle the "shared cache" problem. I thought about
doing sharded redis and/or using redis replication.  I was willing to
give up a little performance on each node that I got by using unix
sockets to gain overall performance in the system. Our goal was to
make as few requests as possible to the backend origin. One of the
issues of using shards is that I really didn't want to write and test a
sharding layer for redis, at least not in the time frame I had. I
started investigating various methods for doing redis clustering and
replication. (For a great discussion of redis sentinel see
[Call Me Maybe - Redis](http://aphyr.com/posts/283-call-me-maybe-redis),
though it doesn't really fit this use case.)  Most of the schemes seem
to have single points of failure and/or were fairly complex.  For our
use case, basically an HTTP app cache, they all seemed like
overkill.

So, I had an idea. I probably heard this idea from somewhere else, so
I'm not claiming it's an original idea.  Each node would have a local
redis and do "lazy/lossy" replication to/from other nodes. It's easier
to explain by using a cache miss as an example:

1. the nginx/Lua app checks redis for a given cache key and does not
  find it
2. nginx makes an HTTP request to the upstream (it could be any
  protocol or computation really)
3. nginx stores the object in redis
4. nginx writes the key of the object and a timestamp (ie,
  _timestamp_-_key_) to a redis pubsub channel.

A small app on each of the cache nodes is listening to the pubsub
channel of each of the other nodes for the cache write messages. This
app uses the timestamp and the key to determine if it should replicate
it. If it decides it should, then it fetches from the other node and
writes it locally. Simple and good enough for our cache use case.

(Yes, this whole post is really about that last paragraph.)

Once we had redis, it was fairly simple to replicate nginx's
[`proxy_cache_use_stale updating`](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_use_stale) functionality as well as cache
statistics. Also, redis's built in [Lua scripting](http://redis.io/commands/eval) meshed well with
our nginx/Lua application.

