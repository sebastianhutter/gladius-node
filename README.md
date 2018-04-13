# Gladius Node (Golang version)

The full suite of binaries for running a Gladius Node

### Build yourself
Install [go](https://golang.org/doc/install)

Build executables for all platforms with `./build-all` and move the appropriate
executables to your preferred install path, and add them to your path. An
install folder structure could look like this
```
your/install/path/gladius/
│   gladius-cli
│   gladius-networkd
│   gladius-control-daemon (not yet included in this repo)
```

##### Some untested stuff with services
Setup the networking daemon (or control-daemon when implemented) service on your
 machine with:
`gladius-networkd install`
Start with: `gladius-networkd start`
Stop with: `gladius-networkd stop`


### Run
Run the executable created by the above step with `gladius-<executable-name>`
(Or use the steps above and make it a service)

## CLI
TODO

## Network Daemon

### Test the RPC server (Only Start and Stop work now)
```bash
$ HDR1='Content-type: application/json'
$ HDR2='Accept: application/json'

$ MSG='{"jsonrpc": "2.0", "method": "GladiusEdge.Start", "id": 1}'
$ curl -H $HDR1 -H $HDR2 -d $MSG http://localhost:5000/rpc
{"jsonrpc":"2.0","result":"Started server","id":1}

$ MSG='{"jsonrpc": "2.0", "method": "GladiusEdge.Stop", "id": 1}'
$ curl -H $HDR1 -H $HDR2 -d $MSG http://localhost:5000/rpc
{"jsonrpc":"2.0","result":"Stopped server","id":1}

$ MSG='{"jsonrpc": "2.0", "method": "GladiusEdge.Status", "id": 1}'
$ curl -H $HDR1 -H $HDR2 -d $MSG http://localhost:5000/rpc
{"jsonrpc":"2.0","result":"Not implemented","id":1}
```

### Some benchmarks compared to the previous version
Done over a gigabit link between two machines with the same bundle file being
served.

#### Node version (with express routing)
```
ab -n 5000 -c 1000 http://<remote IP>:8080/content_bundle

This is ApacheBench, Version 2.3 <$Revision: 1796539 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking <remote IP> (be patient)
Finished 5000 requests


Server Software:        
Server Hostname:        <remote IP>
Server Port:            8080

Document Path:          /content_bundle
Document Length:        452460 bytes

Concurrency Level:      1000
Time taken for tests:   32.079 seconds
Complete requests:      5000
Failed requests:        0
Total transferred:      2263530000 bytes
HTML transferred:       2262300000 bytes
Requests per second:    155.87 [#/sec] (mean)
Time per request:       6415.760 [ms] (mean)
Time per request:       6.416 [ms] (mean, across all concurrent requests)
Transfer rate:          68907.77 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        4  268 611.6     39    3066
Processing:  1095 3654 3755.5   2974   31027
Waiting:       18 1029 3766.7     92   30872
Total:       1112 3922 3957.5   3072   32070

Percentage of the requests served within a certain time (ms)
  50%   3072
  66%   3402
  75%   3549
  80%   3596
  90%   5293
  95%   8859
  98%  18118
  99%  31110
 100%  32070 (longest request)
```
#### Go version
```
ab -n 5000 -c 1000 http://<remote IP>:8080/content\?website\=test.com

This is ApacheBench, Version 2.3 <$Revision: 1796539 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking <remote IP> (be patient)

Finished 5000 requests


Server Software:        fasthttp
Server Hostname:        <remote IP>
Server Port:            8080

Document Path:          /content?website=test.com
Document Length:        452461 bytes

Concurrency Level:      1000
Time taken for tests:   19.265 seconds
Complete requests:      5000
Failed requests:        0
Total transferred:      2263050000 bytes
HTML transferred:       2262305000 bytes
Requests per second:    259.54 [#/sec] (mean)
Time per request:       3853.006 [ms] (mean)
Time per request:       3.853 [ms] (mean, across all concurrent requests)
Transfer rate:          114716.15 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        9  185 365.8     47    3089
Processing:    14 3534 458.6   3622    6552
Waiting:        3   76  99.7     48    2276
Total:         32 3719 637.0   3677    7604

Percentage of the requests served within a certain time (ms)
  50%   3677
  66%   3706
  75%   3749
  80%   3919
  90%   4662
  95%   4834
  98%   5144
  99%   5535
 100%   7604 (longest request)
```

### Set up content delivery

Right now files are loaded from `~/.config/gladius/gladius-networkd/` and take
the format of `example.com.json`. This functionality only works on linux right
now, and serving is not backwards compatible with the previous release. Content
can then be accessed at `http://<host>:8080/content?website=example.com`
