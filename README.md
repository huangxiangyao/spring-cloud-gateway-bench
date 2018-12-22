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
    Latency   280.59ms  186.65ms   1.78s    85.30%
    Req/Sec    85.83     45.56   210.00     61.72%
  22515 requests in 30.06s, 3.18MB read
Requests/sec:    748.88
Transfer/sec:    108.24KB
```

### zuul bench (8081)
```bash
~% wrk -t 10 -c 200 -d 30s http://localhost:8081/hello.txt
Running 30s test @ http://localhost:8081/hello.txt
  10 threads and 200 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   471.78ms  424.48ms   2.00s    79.15%
    Req/Sec    50.03     30.50   190.00     73.07%
  12747 requests in 30.10s, 2.00MB read
  Socket errors: connect 0, read 0, write 0, timeout 234
Requests/sec:    423.54
Transfer/sec:     67.98KB
```

### linkerd bench (9990)
```bash
~% wrk -H "Host: web" -t 10 -c 200 -d 30s http://localhost:9990/hello.txt
Running 30s test @ http://localhost:9990/hello.txt
  10 threads and 200 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    45.32ms   61.21ms 654.76ms   91.49%
    Req/Sec   670.41    270.20     1.22k    70.04%
  185546 requests in 30.09s, 7.96MB read
  Socket errors: connect 0, read 0, write 0, timeout 50
  Non-2xx or 3xx responses: 185546
Requests/sec:   6166.46
Transfer/sec:    270.99KB
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

# FAQ
1.kswapd0 is taking a lot of cpu
In `/etc/sysctl.conf`, try add this line:
```
vm.swappiness = 0
```
See: [https://askubuntu.com/questions/259739/kswapd0-is-taking-a-lot-of-cpu](https://askubuntu.com/questions/259739/kswapd0-is-taking-a-lot-of-cpu)

2.wrk results "timeout xxx"

Use `--timeout xx` to spec a larger timeout (in seconds), eg.
```
wrk -H "Host: web" -t 10 -c 200 -d 30s --timeout 10 http://localhost:8080/gateway/bench/hello.txt
```
