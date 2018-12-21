Spring Cloud Gateway Benchmark
=======

TL;DR

Proxy | Avg Latency | Avg Req/Sec/Thread
-- | -- | -- 
none | 2.09ms | 11.77k
Spring Cloud Gateway | 6.61ms | 3.24k
linkerd | 7.62ms | 2.82k
zuul | 12.56ms | 2.09k
RestCloud Gateway | xx.xxms | x.xxk

## Initial (debian 8)
1.git clone
```
git clone https://github.com/myvas/spring-cloud-gateway-bench.git
```
2.java
```
apt-get install default-jdk
```
3.mvnw
```
apt-get install maven
```
4.https.protocols
```
echo 'export JAVA_TOOL_OPTIONS="-Dhttps.protocols=TLSv1.2"' >> ~/.bashrc
source ~/.bashrc
```

## Terminal 1 (simple webserver)

```bash
cd static
./webserver # or ./webserver.darwin-amd64 on a mac
```

## Terminal 2 (zuul)
```bash
cd zuul
./mvnw clean package
java -jar target/zuul-0.0.1-SNAPSHOT.jar 
```

## Terminal 3 (gateway)
```bash
cd gateway
./mvnw clean package
java -jar target/gateway-0.0.1-SNAPSHOT.jar 
```

## Terminal 4 (linkerd)
```bash
cd linkerd
java -jar linkerd-1.3.4.jar linkerd.yaml
```

## Terminal N (wrk)

### install `wrk`
Ubuntu: `sudo apt-get install wrk`

Mac: `brew install wrk`

NOTE: run each one multiple times to warm up jvm

### Gateway bench (8082)
```bash
$ wrk -t 10 -c 200 -d 30s http://localhost:8082/hello.txt
Running 30s test @ http://localhost:8082/hello.txt
  10 threads and 200 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     6.61ms    4.71ms  49.59ms   69.36%
    Req/Sec     3.24k   278.42     9.02k    75.89%
  969489 requests in 30.10s, 175.67MB read
Requests/sec:  32213.38
Transfer/sec:      5.84MB

```

### zuul bench (8081)
```bash
~% wrk -t 10 -c 200 -d 30s http://localhost:8081/hello.txt
Running 30s test @ http://localhost:8081/hello.txt
  10 threads and 200 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    12.56ms   13.35ms 195.11ms   86.33%
    Req/Sec     2.09k   215.10     4.28k    71.81%
  625781 requests in 30.09s, 123.05MB read
Requests/sec:  20800.13
Transfer/sec:      4.09MB
```

### linkerd bench (9990)
```bash
~% wrk -H "Host: web" -t 10 -c 200 -d 30s http://localhost:9990/hello.txt
Running 30s test @ http://localhost:9990/hello.txt
  10 threads and 200 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     7.62ms    5.45ms  53.51ms   69.82%
    Req/Sec     2.82k   184.58     4.11k    72.17%
  843418 requests in 30.07s, 186.61MB read
Requests/sec:  28050.76
Transfer/sec:      6.21MB
```

### RestCloud Gateway bench (8080)
```bash
~% wrk -H "Host: web" -t 10 -c 200 -d 30s http://localhost:8080/restcloud/rest/gateway/hello
Running 30s test @ http://localhost:8080/restcloud/rest/gateway/hello
  10 threads and 200 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     316ms   477.89ms   1.99s    61.79%
    Req/Sec   312.67     14.62   121.00     86.68%
  1730 requests in 30.05s, 707.88KB read
  Socket errors: connect 0, read 0, write 0, timeout 1327
  Non-2xx or 3xx responses: 1730
Requests/sec:     57.58
Transfer/sec:     23.56KB
```

### no proxy bench (8000)
```bash
~% wrk -t 10 -c 200 -d 30s http://localhost:8000/hello.txt
Running 30s test @ http://localhost:8000/hello.txt
  10 threads and 200 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     2.09ms    2.07ms  28.37ms   85.89%
    Req/Sec    11.77k     2.07k   45.46k    70.97%
  3516807 requests in 30.10s, 637.24MB read
Requests/sec: 116841.15
Transfer/sec:     21.17MB
```
