---
layout: post
title: "SmartOs Playing at Home"
date: 2013-05-18 14:57
comments: true
categories:
- smartos
- virtualization
- project-fifo
---

About a month ago, I decided I wanted to rebuild my Linux server that
I'd used for backups and general playing.  The power supply had died
in it and it was a Frankenstein collection of hardware - particularly
harddrives.  After doing some reading and talking with friends, I
decided that this would be a good opportunity to try [SmartOS](http://smartos.org/).

<!--more-->

I won't go into much detail about what SmartOS is. You can read about
that on your own.  I had a decent amount of Solaris experience, but it
was quite dated - like Solaris 9.

I did a good bit of reading before buying any hardware and decided to
just buy an already built server. I wound up buying a Dell [PowerEdge T110 II Tower Server](http://www.dell.com/us/business/p/poweredge-t110-2/fs).  I got the smallest SATA drive they sold and 2
8GB DIMMs of RAM.
I seperately ordered 3 [3TB SATA drives](http://www.amazon.com/gp/product/B005T3GRLY/ref=oh_details_o02_s00_i01?ie=UTF8&psc=1)
and a
[128GB SSD](http://www.amazon.com/gp/product/B009NB8WR0/ref=oh_details_o02_s00_i03?ie=UTF8&psc=1)
from Amazon.  Shortly after receiving it, I went ahead and order 2
more
[8GB DIMMs](http://www.crucial.com/store/mpartspecs.aspx?mtbpoid=FFCD7DA4A5CA7304).

I had previously burned a copy of SmartOS onto a DVD in preperation. I
booted it and noticed that it only saw one hard drive. Digging through
the Dell bios, I found that I had to specifically enable each hard
drive.  Having done that, I rebooted into SmartOS and only added the 3
SATA drives to the ZFS pool. I finished the "setup wizard" - 2 minutes
tops - and rebooted. I manually added the SSD as cache to the pool:
`zpool add zones cache c0t3d0` and I had the following:

    root@resurrection ~]# zpool status
      pool: zones
     state: ONLINE
     scan: none requested
    config:

        NAME        STATE     READ WRITE CKSUM
        zones       ONLINE       0     0     0
          raidz1-0  ONLINE       0     0     0
            c0t0d0  ONLINE       0     0     0
            c0t1d0  ONLINE       0     0     0
            c0t2d0  ONLINE       0     0     0
        cache
          c0t3d0    ONLINE       0     0     0

    errors: No known data errors

Note: all of my electronics at home are named after Battlestar
characters. I named this box "resurrection" after the [resurrection ships](http://en.battlestarwiki.org/wiki/Resurrection_Ship) as I'd be playing with clones, etc.

I played with manually creating a few zones as well as some kvm Linux
boxes using `vmadm` directly. I'd done some research and decided to
try [Project-Fifo](http://project-fifo.net/). I followed the install
guide and had it running.

Note at this point, only an hour had passed since I unboxed the
server. I'd installed the new hard drives, setup SmartOS, and setup
Project-Fifo all in un under an hour. I'll just say I have not had
this painless of an experience with an open source project in a long
time, if ever.

(Note: I installed the other memory about a week later.)

I played with various things in ZFS. I had played with Solaris 10 a
bit at $DAYJOB, but I wouldn't say I was exprienced with it. ZFS was a
breeze to play with.  In a few short days, I was a ZFS convert.  I had
done some dtrace work on OS X, so I already knew a bit  bout it.

Over the next couple of weeks I did a little bit more in depth playing
with Project-Fifo and wrote some simple utilities for it:

* [project-fifo-ruby](https://github.com/bakins/project-fifo-ruby) -
  basic ruby client for the Project-Fifo HTTP API.
* [knife-fifo](https://github.com/bakins/knife-fifo) - [Chef knife](http://docs.opscode.com/plugin_knife.html)
  plugin for Project-Fifo
* https://github.com/bakins/kitchen-fifo -
  [test-kitchen](https://github.com/opscode/test-kitchen) driver for
  Project-Fifo

These are also on [rubygems](http://rubygems.org).

These are very much works in progress and were written more for my own
education that anything.

I'm so impressed that I am actually doing more official testing at
$DAYJOB.

