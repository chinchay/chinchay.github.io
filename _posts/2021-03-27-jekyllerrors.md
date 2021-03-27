---
title: Jekyll errors fix
author: Carlos Leon
date: 2021-03-27
categories: [Blogging, fix]
tags: []
<!---pin: true--->
---

## `--trace` error

When trying to run `bundle exec jekyll s`, the following error might arise

```console
$ bundle exec jekyll s
...
                    ------------------------------------------------
      Jekyll 4.2.0   Please append `--trace` to the `serve` command
                     for any additional information or backtrace.
                    ------------------------------------------------
...
```
[This solution](https://stackoverflow.com/questions/14973082/bundler-could-not-find-compatible-versions-for-gem-nokogiri) might work:

```console
$ gem "webrick", "~> 1.7"
```

## Close correctly with Ctrl-C

The error might arise also when not closing the previous job correctly, so it is busy to open a new job. Here is a [solution](https://stackoverflow.com/questions/31039998/rails-address-already-in-use-bind2-errnoeaddrinuse) that worked for me:

```console
$ lsof -wni tcp:4000
$ kill -9 <thePID>
```