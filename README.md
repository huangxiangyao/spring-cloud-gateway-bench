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
~% wrk -H "Host: web" -t 10 -c 200 -d 30s http://localhost:8080/gateway/bench/hello.txt
Running 30s test @ http://localhost:8080/gateway/bench/hello.txt
  10 threads and 200 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   843.02ms  547.03ms   1.99s    60.46%
    Req/Sec    15.27     18.24   171.00     93.40%
  2371 requests in 31.45s, 861.31KB read
  Socket errors: connect 0, read 0, write 0, timeout 879
Requests/sec:     75.40
Transfer/sec:     27.39KB
```

### no proxy bench (8000)
```bash
~% wrk -t 10 -c 200 -d 30s http://localhost:8000/hello.txt
Running 30s test @ http://localhost:8000/hello.txt
  10 threads and 200 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     2.63ms   47.70ms   1.79s    99.59%
    Req/Sec     4.98k     5.88k   17.11k    78.31%
  57839 requests in 30.07s, 8.16MB read
  Socket errors: connect 0, read 0, write 0, timeout 128
Requests/sec:   1923.78
Transfer/sec:    278.05KB
```
