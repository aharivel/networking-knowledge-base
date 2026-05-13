# Benchmarks & Network Performance Tools

> Measuring, analyzing, and optimizing network performance

---

## 🎯 Purpose

Network benchmarks and performance tools help you:
- **Measure** throughput, latency, packet loss
- **Identify** bottlenecks and performance issues
- **Compare** different configurations and hardware
- **Validate** SLA requirements
- **Optimize** network performance

## 📊 Key Network Metrics

| Metric | Definition | Typical Values | Measurement Tools |
|--------|------------|----------------|-------------------|
| **Throughput** | Data transfer rate (bps) | 1 Gbps - 100+ Gbps | iperf3, nuttcp |
| **Latency** | Time for packet to travel | 0.1ms - 100ms (LAN/WAN) | ping, hping, traceroute |
| **Jitter** | Latency variation | <1ms (good), 1-10ms (acceptable) | mtr, iperf3 |
| **Packet Loss** | % of lost packets | <0.1% (good), >1% (problem) | ping, iperf3, mtr |
| **CPU Usage** | % CPU consumed | Varies | top, vmstat, perf |
| **Memory Usage** | Memory consumed | Varies | free, vmstat |
| **Connections/s** | New connections per second | 10K - 1M+ | wrk, nginx bench |
| **Requests/s** | Requests per second | 1K - 1M+ | ab, wrk, siege |

## 🧪 Network Benchmarking Tools

### iperf3

**Purpose**: Network throughput testing (TCP & UDP)

**Installation:**
```bash
# Ubuntu/Debian
sudo apt install iperf3

# CentOS/RHEL
sudo yum install iperf3

# macOS
brew install iperf3

# From source
wget https://downloads.es.net/pub/iperf/iperf-3.16.tar.gz
tar xzf iperf-3.16.tar.gz
cd iperf-3.16
./configure && make && sudo make install
```

**Basic Usage:**
```bash
# Server (receiver)
iperf3 -s -p 5201

# Client (sender) - TCP
iperf3 -c <server_ip> -p 5201 -t 60 -i 10

# Client - UDP
iperf3 -c <server_ip> -p 5201 -u -b 1G -t 60 -i 10
```

**Common Options:**

| Option | Description |
|--------|-------------|
| `-c <host>` | Client mode, connect to server |
| `-s` | Server mode |
| `-p <port>` | Port (default: 5201) |
| `-t <time>` | Test duration in seconds |
| `-i <interval>` | Reporting interval |
| `-P <parallel>` | Number of parallel streams |
| `-R` | Reverse mode (server sends, client receives) |
| `-b <bandwidth>` | UDP bandwidth target |
| `-u` | UDP mode |
| `-w <window>` | TCP window size |
| `-M <mss>` | TCP MSS |
| `-J` | JSON output |

**Examples:**

**Single TCP Stream:**
```bash
iperf3 -c 192.168.1.100 -p 5201 -t 30 -i 5
```

**Multiple Parallel Streams (simulate multiple connections):**
```bash
iperf3 -c 192.168.1.100 -p 5201 -P 10 -t 30
```

**UDP Throughput Test:**
```bash
iperf3 -c 192.168.1.100 -p 5201 -u -b 1G -t 30 -i 5
```

**Bidirectional Test:**
```bash
# Run on both machines
iperf3 -c 192.168.1.100 -p 5201 -t 30 -R
iperf3 -c 192.168.1.101 -p 5201 -t 30 -R
```

**Multi-threaded Test:**
```bash
iperf3 -c 192.168.1.100 -p 5201 -t 30 -P 20 --bidir
```

### nuttcp

**Purpose**: Advanced network performance measurement (successor to nettest)

**Features:**
- More accurate than iperf3 for some scenarios
- Supports various protocols (TCP, UDP, ICMP)
- Detailed statistics
- Scriptable

**Installation:**
```bash
# Ubuntu/Debian
sudo apt install nuttcp

# From source
wget https://nuttcp.net/nuttcp-8.2.2.tar.gz
tar xzf nuttcp-8.2.2.tar.gz
cd nuttcp-8.2.2
./configure && make && sudo make install
```

**Usage:**
```bash
# Server
nuttcp -S

# Client - TCP
nuttcp -i1 -T30 <server_ip>

# Client - UDP
nuttcp -i1 -u -T30 <server_ip>

# Client - with JSON output
nuttcp -i1 -T30 -j <server_ip>
```

**Common Options:**

| Option | Description |
|--------|-------------|
| `-S` | Server mode |
| `-i <interval>` | Reporting interval |
| `-T <time>` | Test duration |
| `-u` | UDP mode |
| `-n <count>` | Number of iterations |
| `-p <port>` | Port |
| `-j` | JSON output |
| `-v` | Verbose |

### flent (The FLExible Network Tester)

**Purpose**: Advanced network testing with visualization

**Features:**
- Combines ping, iperf, and other tests
- Generates plots and statistics
- Configurable test profiles
- Python-based

**Installation:**
```bash
# Ubuntu/Debian
sudo apt install flent python3-matplotlib python3-numpy

# pip
pip install flent
```

**Usage:**
```bash
# Run a test
flent rrul -H <server_ip> -l 60 -i 1 -o test_result.png

# Test types:
# - rrul: Realtime Response Under Load
# - ping: Simple ping test
# - iperf: iperf-based test
# - tcp_cubic: TCP cubic test
```

**Example - RRUL Test:**
```bash
# Start iperf3 server on remote
ss -tlnp | grep 5201 || iperf3 -s

# Run RRUL test
flent rrul -H <server_ip> -l 120 -i 1 -o rrul_test.png
```

### netperf

**Purpose**: Network performance benchmarking

**Features:**
- Mature, stable tool
- Supports many test types
- Good for detailed analysis

**Installation:**
```bash
# Ubuntu/Debian
sudo apt install netperf

# From source
wget ftp://ftp.netperf.org/netperf/netperf-2.7.0.tar.bz2
tar xjf netperf-2.7.0.tar.bz2
cd netperf-2.7.0
./configure && make && sudo make install
```

**Usage:**
```bash
# Server
netserver

# Client - TCP throughput
netperf -H <server_ip> -p 12865 -t TCP_STREAM -l 60

# Client - UDP throughput
netperf -H <server_ip> -p 12865 -t UDP_STREAM -l 60

# Client - Request/response
netperf -H <server_ip> -p 12865 -t TCP_RR -l 60
```

## 🔍 Latency Measurement Tools

### ping

**Purpose**: Basic latency and packet loss measurement

**Usage:**
```bash
# Basic ping
ping <host>

# Continuous ping
ping -c 100 <host>

# Ping with interval
ping -i 0.1 -c 1000 <host>

# Ping with timestamp
ping -D <host>

# Flood ping (root required)
ping -f <host>
```

**Windows:**
```cmd
ping <host>
ping -t <host>  # Continuous
ping -n 100 <host>  # Count
ping -w 1000 <host>  # Timeout in ms
```

**Statistics:**
```
--- example.com ping statistics ---
100 packets transmitted, 100 received, 0% packet loss, time 99999ms
rtt min/avg/max/mdev = 10.123/15.456/20.789/2.345 ms
```

### hping

**Purpose**: Advanced ping with custom packet crafting

**Features:**
- Custom packet size and content
- TCP/UDP/ICMP modes
- Flood mode
- Port scanning

**Installation:**
```bash
# Ubuntu/Debian
sudo apt install hping3

# From source
wget http://www.hping.org/hping3/hping3-20051105.tar.gz
tar xzf hping3-20051105.tar.gz
cd hping3-20051105
./configure && make && sudo make install
```

**Usage:**
```bash
# TCP ping (SYN packets)
hping3 -S <host>

# UDP ping
hping3 -2 <host>

# ICMP ping
hping3 -1 <host>

# With packet size
hping3 -S -d 1472 <host>  # 1500 byte packets (1472 + 20 IP + 8 ICMP)

# Flood mode (careful!)
hping3 -S --flood <host>

# Port scan
hping3 -S -p 80,443 <host>
```

### mtr (My Traceroute)

**Purpose**: Combines traceroute and ping for path analysis

**Features:**
- Shows path to destination
- Measures latency and packet loss at each hop
- Continuous monitoring
- Interactive and report modes

**Installation:**
```bash
# Ubuntu/Debian
sudo apt install mtr

# CentOS/RHEL
sudo yum install mtr

# macOS
brew install mtr
```

**Usage:**
```bash
# Interactive mode
mtr <host>

# Report mode (text output)
mtr -r <host>

# Report mode with wide output
mtr -rw <host>

# With interval and count
mtr -i 0.5 -c 100 -r <host>

# UDP mode (instead of ICMP)
mtr -u <host>

# TCP mode
mtr -T <host>
```

**Output Example:**
```
My traceroute  [v0.93]
MyNode (0.0.0.0) -> example.com (93.184.216.34)
Keys:  Help   Display mode   Restart statistics   Order of fields   quit
                                     Packets               Pings
 Host                                    Loss%   Snt   Last   Avg  Best  Wrst StDev
 1. router.local                         0.0%   100    1.2   1.3   1.1   1.5   0.1
 2. 10.0.0.1                             0.0%   100    5.6   5.7   5.5   6.1   0.2
 3. 209.85.241.17                        0.0%   100   12.3  12.4  12.2  12.7   0.1
 4. 209.85.255.145                       0.0%   100   15.6  15.7  15.5  16.0   0.1
 5. 93.184.216.34                        0.0%   100   20.1  20.2  20.0  20.5   0.1
```

### traceroute

**Purpose**: Trace the path packets take to a destination

**Installation:**
```bash
# Ubuntu/Debian
sudo apt install traceroute

# CentOS/RHEL
sudo yum install traceroute

# macOS
# Built-in: traceroute <host>
# Or install from homebrew
brew install traceroute
```

**Usage:**
```bash
# Basic traceroute
traceroute <host>

# With specific maximum TTL
traceroute -m 30 <host>

# UDP mode (default)
traceroute -U <host>

# TCP mode
traceroute -T <host>

# ICMP mode
traceroute -I <host>

# With source port
traceroute -s 50000 <host>

# AS path lookup
traceroute -A <host>

# Geographic lookup
traceroute -g <host>
```

**Windows Equivalent:**
```cmd
tracert <host>
```

## 🌐 HTTP Load Testing Tools

### ab (Apache Benchmark)

**Purpose**: HTTP server benchmarking

**Installation:**
```bash
# Ubuntu/Debian
sudo apt install apache2-utils

# CentOS/RHEL
sudo yum install httpd-tools
```

**Usage:**
```bash
# Basic test
ab -n 1000 -c 10 http://example.com/

# With keep-alive
ab -n 1000 -c 10 -k http://example.com/

# POST request
ab -n 1000 -c 10 -p post_data.txt -T "application/json" http://example.com/api

# With timeout
ab -n 1000 -c 10 -s 30 http://example.com/
```

**Options:**

| Option | Description |
|--------|-------------|
| `-n <requests>` | Number of requests |
| `-c <concurrency>` | Number of concurrent requests |
| `-t <timelimit>` | Max seconds for test |
| `-k` | Enable keep-alive |
| `-H <header>` | Add custom header |
| `-p <file>` | POST file content |
| `-T <type>` | Content type |

**Output Example:**
```
Server Software:        nginx/1.18.0
Server Hostname:        example.com
Server Port:            80

Document Path:          /
Document Length:        1256 bytes

Concurrency Level:      10
Time taken for tests:   1.234 seconds
Complete requests:      1000
Failed requests:        0
Total transferred:      1287000 bytes
HTML transferred:       1256000 bytes
Requests per second:    810.37 [#/sec] (mean)
Time per request:       12.345 [ms] (mean, across all concurrent requests)
Time per request:       1.234 [ms] (mean, per concurrency level)
Transfer rate:          1029.45 [Kbytes/sec] received

Connection Times (ms):
              min  mean[+/-sd] median   max
Total:        10   12  0.5     12      15
Wait:          5   10  0.3     10      12
Connect:       1    2  0.1      2       3
Processing:    4    8  0.4      8      10
```

### wrk

**Purpose**: Modern HTTP benchmarking tool

**Features:**
- Multi-threaded
- Scriptable (Lua)
- Supports HTTP/1.1 and HTTP/2
- Real-time statistics

**Installation:**
```bash
# Ubuntu/Debian
sudo apt install wrk

# From source
git clone https://github.com/wg/wrk.git
cd wrk
make && sudo cp wrk /usr/local/bin
```

**Usage:**
```bash
# Basic test
wrk -t12 -c400 -d30s http://example.com

# With multiple threads
wrk -t20 -c2000 -d60s http://example.com

# HTTP/2
wrk -t12 -c400 -d30s --http2 http://example.com

# With Lua script
wrk -t12 -c400 -d30s -s script.lua http://example.com
```

**Options:**

| Option | Description |
|--------|-------------|
| `-t <threads>` | Number of threads |
| `-c <connections>` | Connections per thread |
| `-d <duration>` | Test duration |
| `--latency` | Record latency statistics |
| `--timeout <ms>` | Socket timeout |
| `-s <script>` | Lua script file |

**Lua Scripting Example (`script.lua`):**
```lua
wrk.method = "POST"
wrk.headers["Content-Type"] = "application/json"
wrk.body = '{"key": "value"}'

request = function()
  return wrk.format(nil, nil, wrk.headers, wrk.body)
end
```

**Output Example:**
```
Running 30s test @ http://example.com
  12 threads and 400 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    285.12ms   56.25ms 200.49ms   86.89%
    Req/Sec     1.65k   299.17     2.33k    73.33%
  120075 requests in 30.00s, 17.20MB read
Requests/sec:   4002.50
```

### siege

**Purpose**: HTTP load testing and benchmarking

**Features:**
- Multi-threaded
- Configurable via file
- Supports various protocols
- Simulates real users

**Installation:**
```bash
# Ubuntu/Debian
sudo apt install siege

# From source
wget http://download.joedog.org/siege/siege-latest.tar.gz
tar xzf siege-latest.tar.gz
cd siege-*
./configure && make && sudo make install
```

**Usage:**
```bash
# Basic test
siege -c 50 -r 10 http://example.com

# With configuration file
siege -f urls.txt

# Verbose mode
siege -v -c 100 -r 20 http://example.com

# Random delays
siege -i -c 100 -r 20 http://example.com
```

**`urls.txt` Example:**
```
http://example.com/
http://example.com/about
http://example.com/contact
http://example.com/api/data
```

### nginx bench (ab alternative)

**Purpose**: Simple HTTP benchmarking

**Installation:**
```bash
# Comes with nginx
# Or install nginx
sudo apt install nginx
```

**Usage:**
```bash
nginx bench -c 100 -n 1000 http://example.com/
```

## 📈 Monitoring & Analysis Tools

### vmstat

**Purpose**: Virtual memory statistics

**Usage:**
```bash
# Basic
vmstat 1

# With timestamp
vmstat -t 1

# Slab memory
vmstat -m

# Active/inactive memory
vmstat -a
```

**Output:**
```
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 123456  78901 234567    0    0    10    20   50  100  5  2 93  0  0
```

### sar

**Purpose**: System activity reporter (historical and real-time)

**Installation:**
```bash
# Ubuntu/Debian
sudo apt install sysstat

# CentOS/RHEL
sudo yum install sysstat

# Enable data collection
sudo systemctl enable sysstat
sudo systemctl start sysstat
```

**Usage:**
```bash
# CPU usage
sar -u 1 5

# Memory usage
sar -r 1 5

# Network statistics
sar -n DEV 1 5

# TCP statistics
sar -n TCP 1 5

# Historical data
sar -f /var/log/sa/sa01
```

### ss

**Purpose**: Socket statistics (replacement for netstat)

**Usage:**
```bash
# List all TCP connections
ss -t -a

# List UDP sockets
ss -u -a

# List listening ports
ss -t -l -n -p

# Summary statistics
ss -s

# TCP connections with state
ss -t -a -o state established

# Detailed socket information
ss -t -a -n -i
```

### ip

**Purpose**: IP configuration and statistics

**Usage:**
```bash
# Show interfaces
ip link show

# Show IP addresses
ip addr show

# Show routes
ip route show

# Show ARP table
ip neigh show

# Show statistics
ip -s link show

# Show TCP info
ip -s -s tcp
```

### ethtool

**Purpose**: Ethernet tool for driver and hardware settings

**Installation:**
```bash
# Ubuntu/Debian
sudo apt install ethtool
```

**Usage:**
```bash
# Show interface information
ethtool eth0

# Show speed and duplex
ethtool eth0 | grep -E "Speed|Duplex"

# Show statistics
ethtool -S eth0

# Show driver info
ethtool -i eth0

# Show offload features
ethtool -k eth0

# Show ring buffer sizes
ethtool -g eth0

# Show packet size statistics
ethtool -s eth0
```

### nload

**Purpose**: Real-time network traffic monitor

**Installation:**
```bash
# Ubuntu/Debian
sudo apt install nload

# CentOS/RHEL
sudo yum install nload
```

**Usage:**
```bash
# Monitor all interfaces
nload

# Monitor specific interface
nload eth0

# With refresh interval
nload -t 500 eth0

# Incoming only
nload -i 1000 eth0

# Outgoing only
nload -o 1000 eth0
```

### iftop

**Purpose**: Bandwidth monitoring per connection

**Installation:**
```bash
# Ubuntu/Debian
sudo apt install iftop

# CentOS/RHEL
sudo yum install iftop
```

**Usage:**
```bash
# Monitor all connections
sudo iftop

# Monitor specific interface
sudo iftop -i eth0

# With port display
sudo iftop -P

# Show cumulative totals
sudo iftop -C

# Sort by destination
sudo iftop -o destination
```

### iptraf-ng

**Purpose**: Interactive network traffic monitor

**Installation:**
```bash
# Ubuntu/Debian
sudo apt install iptraf-ng

# CentOS/RHEL
sudo yum install iptraf-ng
```

**Usage:**
```bash
# Start interactive interface
sudo iptraf-ng
```

### tcpdump

**Purpose**: Packet capture and analysis

**Installation:**
```bash
# Ubuntu/Debian
sudo apt install tcpdump

# CentOS/RHEL
sudo yum install tcpdump

# macOS
# Built-in
```

**Usage:**
```bash
# Capture on interface
tcpdump -i eth0

# Capture with count
tcpdump -i eth0 -c 100

# Write to file
tcpdump -i eth0 -w capture.pcap

# Read from file
tcpdump -r capture.pcap

# Filter by host
tcpdump -i eth0 host 192.168.1.100

# Filter by port
tcpdump -i eth0 port 80

# Filter by protocol
tcpdump -i eth0 tcp

# Filter by source/destination
tcpdump -i eth0 src 192.168.1.100
tcpdump -i eth0 dst 192.168.1.100

# Show packet contents
tcpdump -i eth0 -X

# Verbose output
tcpdump -i eth0 -v

# Very verbose
tcpdump -i eth0 -vv

# Print ASCII
tcpdump -i eth0 -A
```

### Wireshark

**Purpose**: Graphical network protocol analyzer

**Installation:**
```bash
# Ubuntu/Debian
sudo apt install wireshark

# CentOS/RHEL
sudo yum install wireshark

# macOS
brew install --cask wireshark
```

**Usage:**
```bash
# Start GUI
wireshark &

# CLI capture (tshark)
tshark -i eth0 -w capture.pcap

# CLI analysis
tshark -r capture.pcap -q -z io,stat,0
```

**Common Filters:**

| Filter | Description |
|--------|-------------|
| `tcp` | TCP packets only |
| `udp` | UDP packets only |
| `http` | HTTP traffic |
| `ip.addr == 192.168.1.100` | Specific IP |
| `tcp.port == 80` | Port 80 traffic |
| `http.request.method == "GET"` | GET requests |
| `tcp.analysis.retransmission` | TCP retransmissions |

## 📊 Creating Custom Benchmarks

### Python Script Example

```python
#!/usr/bin/env python3
import time
import socket
import statistics
import argparse

def measure_latency(host, port=80, count=100):
    """Measure TCP connection latency"""
    latencies = []
    
    for _ in range(count):
        start = time.perf_counter()
        try:
            sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            sock.settimeout(5)
            sock.connect((host, port))
            sock.close()
        except Exception as e:
            print(f"Error: {e}")
            return None
        end = time.perf_counter()
        latencies.append((end - start) * 1000)  # Convert to ms
        time.sleep(0.01)  # Small delay between tests
    
    return latencies

def measure_throughput(host, port=80, size=1024, count=100):
    """Measure TCP throughput"""
    total_bytes = 0
    start = time.perf_counter()
    
    for _ in range(count):
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.connect((host, port))
        data = b'X' * size
        sock.sendall(data)
        sock.recv(size)  # Wait for response
        sock.close()
        total_bytes += size * 2  # Send + receive
    
    end = time.perf_counter()
    elapsed = end - start
    throughput = (total_bytes * 8) / elapsed / 1000 / 1000  # Mbps
    
    return throughput, elapsed

if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="Network Benchmark Tool")
    parser.add_argument("host", help="Target host")
    parser.add_argument("-p", "--port", type=int, default=80, help="Port")
    parser.add_argument("-c", "--count", type=int, default=100, help="Test count")
    parser.add_argument("--latency", action="store_true", help="Test latency")
    parser.add_argument("--throughput", action="store_true", help="Test throughput")
    args = parser.parse_args()
    
    if args.latency:
        latencies = measure_latency(args.host, args.port, args.count)
        if latencies:
            print(f"Latency to {args.host}:{args.port}")
            print(f"  Min: {min(latencies):.3f} ms")
            print(f"  Max: {max(latencies):.3f} ms")
            print(f"  Avg: {statistics.mean(latencies):.3f} ms")
            print(f"  Median: {statistics.median(latencies):.3f} ms")
            print(f"  Std Dev: {statistics.stdev(latencies):.3f} ms")
    
    if args.throughput:
        throughput, elapsed = measure_throughput(args.host, args.port, 1024, args.count)
        print(f"Throughput to {args.host}:{args.port}")
        print(f"  Total: {throughput:.2f} Mbps")
        print(f"  Time: {elapsed:.2f} seconds")
```

## 🎯 Key Takeaways

1. **iperf3** is the gold standard for throughput testing
2. **ping/hping** for basic latency measurement
3. **mtr/traceroute** for path analysis
4. **wrk/ab** for HTTP load testing
5. **vmstat/sar** for system-level monitoring
6. **tcpdump/Wireshark** for packet-level analysis
7. **ethtool** for NIC-level statistics
8. **Always test both ways** (bidirectional)
9. **Multiple runs** for consistent results
10. **Baseline before optimizing**

## 📚 Best Practices

### Before Benchmarking
1. **Check system health** (CPU, memory, disk)
2. **Disable unnecessary services**
3. **Close background applications**
4. **Use dedicated hardware** if possible
5. **Document baseline** before changes

### During Benchmarking
1. **Run multiple iterations**
2. **Test at different times** (network conditions vary)
3. **Test both directions** (client ↔ server)
4. **Use appropriate packet sizes**
5. **Monitor system resources** during test

### After Benchmarking
1. **Analyze results** (not just numbers)
2. **Compare with baselines**
3. **Identify bottlenecks**
4. **Document findings**
5. **Share with team**

## 🔗 Further Reading

- [iperf3 Documentation](https://iperf.fr/iperf-doc.php)
- [nuttcp Documentation](https://nuttcp.net/nuttcpman.html)
- [flent Documentation](https://flent.org/)
- [wrk GitHub](https://github.com/wg/wrk)
- [siege Documentation](https://www.joedog.org/siege-home/)
- [Linux Networking Tools](https://www.linuxfoundation.org/en/blog/linux-networking-commands/)
- [Brendan Gregg's Tools](http://www.brendangregg.com/linuxperf.html)
