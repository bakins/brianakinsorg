---
layout: post
title: "Jekyll Gzip Hack"
date: 2013-05-18 15:56
comments: true
categories: []
---

Here's a quick hack I did to my install of Jekyll that will gzip most files.  Here's my modified `generate` rake task:

``` ruby
desc "Generate jekyll site"
task :generate do
  raise "### You haven't set anything up yet. First run `rake install` to set up an Octopress theme." unless File.directory?(source_dir)
  puts "## Generating Site with Jekyll"
  system "bundle exec compass compile --css-dir #{source_dir}/stylesheets"
  system "bundle exec jekyll"

  Find.find "public" do |path|
    if FileTest.file?(path) and path.match(/\.(css|js|html|xml|txt)$/)
      system "gzip -c #{path} > #{path}.gz"
    end
  end

end
```

I use this in conjunction with nginx's [gzip static module](http://nginx.org/en/docs/http/ngx_http_gzip_static_module.html). Here's that config:

```
gzip_static       on;
gzip_http_version 1.1;
gzip_disable      "MSIE [1-6]\.";
gzip_vary         on;
gzip_types        text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;
```