---
layout: post
title: "Simple Consul Example"
date: 2014-05-03 11:16
comments: false
categories:
---
On both:

* `sudo apt-get install -y curl unzip`
* `curl -OL https://dl.bintray.com/mitchellh/consul/0.2.0_linux_amd64.zip`
* `unzip 0.2.0_linux_amd64.zip`
* `chmod +x consul`
* `sudo cp consul /usr/local/bin/`

On Server:

* `sudo apt-get install -y apt-cacher-ng`
* `consul agent -server -bootstrap -data-dir /tmp/consul`

On Client:

* `consul agent -data-dir /tmp/consul`

Then in another terminal on client:

* `consul members`
* `consul join <ip-of-server>`
* `consul members`

In another terminal on server:

* `sudo mkdir /etc/consul.d`
* `sudo chmod 777 /etc/consul.d`
* `echo '{"service": {"name": "apt-cacher-ng", "port": 3142,  "check": { "script": "/usr/bin/curl -sS -o /dev/null http://127.0.0.1:3142/acng-report.html", "interval": "10s"} } }' > /etc/consul.d/apt-cacher-ng.json`

Then in the terminal where the consul server is running, `Ctrl+C` tos top it then run:

* `consul agent -server -bootstrap -data-dir /tmp/consul -config-dir /etc/consul.d`

# you will probably have to rejoin in the client: `consul join <ip-of-server>`

make sure you can see the service from both client and server:

* `curl http://localhost:8500/v1/catalog/service/apt-cacher-ng`
* `curl http://localhost:8500/v1/health/service/apt-cacher-ng`
* `curl http://localhost:8500/v1/health/service/apt-cacher-ng?passing`

On the client install jq:

* `curl -LO http://stedolan.github.io/jq/download/linux64/jq`
* `chmod +x jq`
* `sudo cp jq /usr/local/bin/`

Grab the service info:

* `curl -s http://localhost:8500/v1/health/service/apt-cacher-ng?passing | jq -r '.[] | .Node.Address + ":" + (.Service.Port | tostring)'`

Now do the discovery config into `/etc/apt/apt.conf.d/30autoproxy`:
`Acquire::http::ProxyAutoDetect "/usr/local/bin/deb-proxy-discover";`

Then write the script into `/usr/local/bin/deb-proxy-discover` and `chmod +x` it

    #!/bin/sh
    PATH=$PATH:/usr/local/bin
    curl -s http://localhost:8500/v1/health/service/apt-cacher-ng?passing | jq -r '.[] | "http://" + .Node.Address + ":" + (.Service.Port | tostring)' | sort -R | head -n1


Test:

on the server:
`tail -F /var/log/apt-cacher-ng/apt-cacher.log`

and on the client:

* `sudo apt-get update`
* `sudo apt-get install -y build-essential`

You should see accesses in the apt-cacher.log

