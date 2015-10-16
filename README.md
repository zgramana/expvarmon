# Installation

For OSX, just [download the binary](https://github.com/zgramana/expvarmon/releases/download/0.1/expvarmon).

# Sync Gateway Usage Guide

Monitoring memory stats:
```sh
./expvarmon -ports="4985" -vars="mem:memstats.Alloc,mem:memstats.Sys,mem:memstats.HeapAlloc,mem:memstats.HeapInuse,memstats.NumGC,memstats.PauseTotalNs,memstats.PauseNs"
```
Monitoring CBS get ops:
```sh
./expvarmon -ports="4985" -vars="cb.ops.GetsRaw.count,cb.ops.GetsRaw.p99,cb.ops.GetsRaw.p95,cb.ops.GetsRaw.p90,cb.ops.GetsRaw.p75,cb.ops.GetsRaw.p50,cb.ops.GetsRaw.p25"
```

Monitoring CBS incr ops:
```sh
./expvarmon -ports="4985" -vars="cb.ops.Incr.count,cb.ops.Incr.p99,cb.ops.Incr.p95,cb.ops.Incr.p90,cb.ops.Incr.p75,cb.ops.Incr.p50,cb.ops.Incr.p25"
```

Monitoring CBS write ops:
```sh
./expvarmon -ports="4985" -vars="cb.ops.Write(raw).count,cb.ops.Write(raw).p99,cb.ops.Write(raw).p95,cb.ops.Write(raw).p90,cb.ops.Write(raw).p75,cb.ops.Write(raw).p50,cb.ops.Write(raw).p25"
```

Monitoring CBS CAS ops:
```sh
./expvarmon -ports="4985" -vars="cb.ops.casNext.count,cb.ops.casNext.p99,cb.ops.casNext.p95,cb.ops.casNext.p90,cb.ops.casNext.p75,cb.ops.casNext.p50,cb.ops.casNext.p25"
```

Monitoring CBS pools stats [Doesn't currently work due to bug]:
```sh
#./expvarmon -ports="4985" -vars="cb.ops.pools.count,cb.ops.pools.p99,cb.ops.pools.p95,cb.ops.pools.p90,cb.ops.pools.p75,cb.ops.pools.p50,cb.ops.pools.p25"
```

Monitoring Memcached stats:
```sh
./expvarmon -ports="4985" -vars="mc.recv.errs,mem:mc.recv.bytes.GET,mem:mc.recv.bytes.SET,mem:mc.recv.bytes.INCREMENT,mem:mc.recv.bytes.ADD,mem:mc.recv.bytes.total"
```

Monitoring SG change cache stats:
```sh
./expvarmon -ports="4985" -vars="syncGateway_changeCache.view_queries,syncGateway_changeCache.lag-total-0000ms"
```

Monitoring SG db stats:
```sh
./expvarmon -ports="4985" -vars="syncGateway_db.channelChangesFeeds,syncGateway_db.document_gets,syncGateway_db.revisionCache_adds,syncGateway_db.revs_added,syncGateway_db.sequence_gets,syncGateway_db.sequence_reserves"
```

Monitoring SG listener stats:
```sh
./expvarmon -ports="4985" -vars="syncGateway_httpListener.max_active,syncGateway_httpListener.max_wait"
```

Monitoring SG REST API stats:
```sh
./expvarmon -ports="4985" -vars="syncGateway_rest.changesFeeds_active,syncGateway_rest.changesFeeds_total,syncGateway_rest.requests_active,syncGateway_rest.requests_total,syncGateway_rest.requests_0000ms,syncGateway_rest.requests_0100ms,syncGateway_rest.requests_0600ms"
```


##Things you want to do, but currently cant:
**Cannot group expvars into panels (or especially named panels)**
  Would be nice to be able to group panels like this:
   ```sh
    ./expvarmon -vars="foo,bar"  -vars="fizz,buzz"
   ```
   or even
   ```sh
    ./expvarmon -vars="These:foo,bar"  -vars="Others:fizz,buzz"
   ```
**Cannot support paths like `cb.ops.pools.127.0.0.1:11210.count`**

**Cannot use globs like `cb.ops.*`**


# ExpvarMon

TermUI based Go apps monitor using [expvars](http://golang.org/pkg/expvar/) variables (/debug/vars). Quickest way to monitor your Go app.

## Introduction

Go apps console monitoring tool. Minimal configuration efforts. Quick and easy monitoring solution for one or multiple services.

## Features

* Single- and multi-apps mode
* Local and remote apps support
* HTTP and HTTPS endpoints, including Basic Auth support
* Arbitrary number of apps and vars to monitor (from 1 to 30+, depends on size of your terminal)
* Track restarted/failed apps
* Show maximum value
* Supports: Integer, float, duration, memory, string, bool, array variables
* Sparkline charts for integer, duration and memory data
* Auto-resize on font-size change or window resize
* Uses amazing [TermUI](https://github.com/gizak/termui) library by [gizak](https://github.com/gizak)

## Demo

### Multiple apps mode
<img src="./demo/demo_multi.png" alt="Multi mode" width="800">

### Single app mode
<img src="./demo/demo_single.png" alt="Single mode" width="800">

You can monitor arbitrary number of services and variables:

<a href="./demo/demo_1var.png" target="_blank"><img src="./demo/demo_1var.png" alt="1 var" width="350"></a> <a href="./demo/demo_small.png" target="_blank"><img src="./demo/demo_small.png" alt="25 apps" width="350"></a>

## Purpose

This app targets debug/develop sessions when you need an instant way to monitor you app(s). It's not intended to monitor apps in production.
Also it doesn't use any storage engines and doesn't send notifications.

## Install

Just run go get:

    go get github.com/divan/expvarmon

## Usage

### Prepare your app

First, you have to add [expvars](http://golang.org/pkg/expvar/) support into your Go program. It's as simple as:

    import _ "expvar"

and note the port your app is listening on. If it's not, just add two lines:

    import "net/http"
    ...
    http.ListenAndServe(":1234", nil)

and expvar will add handler for "localhost:1234/debug/vars" to your app.

By default, expvars adds two variables: *memstats* and *cmdline*. It's enough to monitor memory and garbage collector status in your app.

### Run expvarmon

Just run expvarmon with -ports="1234" flag:

    expvarmon -ports="1234"
    
That's it.

More examples:

    ./expvarmon -ports="80"
    ./expvarmon -ports="23000-23010,http://example.com:80-81" -i=1m
    ./expvarmon -ports="80,remoteapp:80" -vars="mem:memstats.Alloc,duration:Response.Mean,Counter"
    ./expvarmon -ports="1234-1236" -vars="Goroutines" -self
    ./expvarmon -ports="https://user:pass@my.remote.app.com:443" -vars="Goroutines" -self

## Advanced usage

If you need to monitor more (or less) vars, you can specify them with -vars command line flag.

    $ expvarmon -help
	Usage of ./expvarmon:
	  -dummy=false: Use dummy (console) output
	  -i=5s: Polling interval
	  -ports="": Ports/URLs for accessing services expvars (start-end,port2,port3,https://host:port)
	  -self=false: Monitor itself
	  -vars="mem:memstats.Alloc,mem:memstats.Sys,mem:memstats.HeapAlloc,mem:memstats.HeapInuse,duration:memstats.PauseNs,duration:memstats.PauseTotalNs": Vars to monitor (comma-separated)

	Examples:
		./expvarmon -ports="80"
		./expvarmon -ports="23000-23010,http://example.com:80-81" -i=1m
		./expvarmon -ports="80,remoteapp:80" -vars="mem:memstats.Alloc,duration:Response.Mean,Counter"
		./expvarmon -ports="1234-1236" -vars="Goroutines" -self

	For more details and docs, see README: http://github.com/divan/expvarmon

So, yes, you can specify multiple ports, using '-' for ranges, and specify fully-qualified URLs for remote apps.

You can also monitor expvarmon itself, using -self flag.

### Basic Auth

If your expvar endpoint is protected by Basic Auth, you have two options:

 - Set environmental variables *HTTP_USER* and *HTTP_PASSWORD* accordingly. These values will be applied to each endpoint.
 - Embed your credentials to URL via command line flag: `-ports="http://user:pass@myapp:1234"`

### Vars

Expvarmon doesn't restrict you to monitor only memstats. You can publish your own counters and variables using [expvar.Publish()](http://golang.org/pkg/expvar/#Publish) method or using expvar wrappers libraries. Just pass your variables names as they appear in JSON to -var command line flag.

Notation is dot-separated, for example: **memstats.Alloc** for .MemStats.Alloc field. Quick link to runtime.MemStats documentation: http://golang.org/pkg/runtime/#MemStats

Expvar allows to export only basic types - structs, ints, floats, arrays (int or float), bools and strings. For arrays, average will be calculated. Ints are used for sparklines, and displayed as is. But you can specify modifier to make sure it will be rendered properly.

Vars are specified as a comma-separated list of var identifiers with (optional) modifiers.

| Modifier | Description |
| --------- | ----------- |
| mem:      | renders int64 as memory string (KB, MB, etc) |
| duration: | renders int64 as time.Duration (1s, 2ms, 12h23h) |
| str:      | doesn't display sparklines chart for this value, just display as string |
