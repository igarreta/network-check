# Technical Resources and Dependencies

## Python Standard Library Modules

### Network Operations
- **`socket`** - Low-level networking interface
  - `socket.getdefaulttimeout()` - Set connection timeouts
  - `socket.gethostbyname()` - DNS resolution
  - `socket.create_connection()` - TCP connection testing
  - `socket.AF_INET`, `socket.SOCK_DGRAM` - UDP operations

- **`subprocess`** - Execute system commands
  - `subprocess.run()` - Execute ping, traceroute
  - `subprocess.PIPE` - Capture command output
  - `subprocess.TimeoutExpired` - Handle command timeouts

- **`urllib`** - HTTP client (stdlib alternative)
  - `urllib.request.urlopen()` - HTTP GET requests
  - `urllib.error.URLError` - Exception handling
  - `urllib.request.Request()` - Custom headers

### Data Handling
- **`json`** - JSON encoding/decoding
  - `json.dump()` - Write structured logs
  - `json.load()` - Read configuration and logs
  - `json.dumps()` - Pretty printing

- **`datetime`** - Date and time operations
  - `datetime.datetime.now()` - Timestamps
  - `datetime.timedelta()` - Duration calculations
  - `datetime.strptime()` - Parse timestamps
  - ISO 8601 formatting

- **`statistics`** - Statistical functions
  - `statistics.mean()` - Average latency
  - `statistics.median()` - Median latency
  - `statistics.stdev()` - Standard deviation
  - Custom percentile calculations (P95, P99)

### File Operations
- **`pathlib`** - Object-oriented filesystem paths
  - `Path.exists()` - Check file existence
  - `Path.mkdir()` - Create directories
  - `Path.glob()` - Pattern matching for log files

- **`os`** - Operating system interface
  - `os.path.join()` - Path construction
  - `os.makedirs()` - Recursive directory creation
  - `os.access()` - Permission checking

- **`logging`** - Logging framework
  - `logging.basicConfig()` - Configure logging
  - `logging.info()`, `logging.error()` - Log levels
  - `logging.FileHandler()` - File output
  - `logging.Formatter()` - Custom formats

### System Information
- **`platform`** - Platform identification
  - `platform.system()` - OS detection (Linux/Darwin/Windows)
  - Cross-platform ping command differences

- **`sys`** - System-specific parameters
  - `sys.argv` - Command-line arguments
  - `sys.exit()` - Exit codes
  - `sys.stderr` - Error output

### Command-Line Interface
- **`argparse`** - Argument parser
  - `ArgumentParser()` - CLI interface
  - `add_argument()` - Define options
  - `parse_args()` - Process arguments

### Other Utilities
- **`time`** - Time operations
  - `time.time()` - High-resolution timestamps
  - `time.sleep()` - Delays for retries

- **`collections`** - Specialized containers
  - `defaultdict()` - Automatic initialization
  - `Counter()` - Counting occurrences

- **`re`** - Regular expressions
  - Parse ping output
  - Extract IP addresses
  - Pattern matching in logs

## External Libraries (Optional)

### HTTP Requests
- **`requests`** (optional, recommended)
  - More robust than urllib
  - Better error handling
  - Session management
  - Automatic retries with `urllib3.Retry`
  ```python
  pip install requests
  ```

### Data Analysis (Future Enhancement)
- **`pandas`** (optional)
  - Advanced log analysis
  - Time series operations
  - Data aggregation
  ```python
  pip install pandas
  ```

### Visualization (Future Enhancement)
- **`matplotlib`** (optional)
  - Line plots of latency over time
  - Uptime/downtime visualization
  ```python
  pip install matplotlib
  ```

- **`plotly`** (optional)
  - Interactive dashboards
  - Real-time monitoring
  ```python
  pip install plotly
  ```

## System Commands and Utilities

### Network Diagnostics

#### Ping
```bash
# Linux/macOS
ping -c 3 -W 5 8.8.8.8

# Windows
ping -n 3 -w 5000 8.8.8.8

# Output parsing:
# - RTT (Round Trip Time)
# - Packet loss percentage
# - Min/Avg/Max latency
```

#### IP Route
```bash
# Get default gateway (Linux)
ip route show default | awk '{print $3}'

# Alternative (older systems)
netstat -rn | grep default | awk '{print $2}'

# macOS
route -n get default | grep gateway | awk '{print $2}'
```

#### DNS Tools
```bash
# Test DNS resolution
nslookup google.com 8.8.8.8
dig @8.8.8.8 google.com

# System DNS configuration
cat /etc/resolv.conf
```

#### Network Interfaces
```bash
# List interfaces
ip addr show
ifconfig

# Check connectivity
ip route get 8.8.8.8
```

## Testing Targets

### DNS Servers
| Provider | Primary | Secondary | Notes |
|----------|---------|-----------|-------|
| Google | 8.8.8.8 | 8.8.4.4 | Most reliable, global anycast |
| Cloudflare | 1.1.1.1 | 1.0.0.1 | Privacy-focused, fast |
| Quad9 | 9.9.9.9 | 149.112.112.112 | Security filtering |
| OpenDNS | 208.67.222.222 | 208.67.220.220 | Content filtering |

### HTTP Test Endpoints
| Endpoint | Reliability | Latency | Geographic Coverage |
|----------|-------------|---------|---------------------|
| https://www.google.com | Very High | Low | Global CDN |
| https://www.cloudflare.com | Very High | Low | Global Anycast |
| https://www.amazon.com | High | Medium | Global CDN |
| https://www.microsoft.com | High | Medium | Global CDN |
| http://detectportal.firefox.com/success.txt | High | Low | Captive portal detection |

### Common Gateway IPs (Deco Routers)
- `192.168.68.1` - Default Deco IP
- `192.168.1.1` - Common alternative
- `192.168.0.1` - Another common default
- `10.0.0.1` - ISP router common IP

## File Formats

### Log Format (JSON Lines)
```json
{"timestamp": "2025-11-21T10:30:45.123456", "test_type": "ping_gateway", "status": "success", "latency_ms": 12.5}
```
- One JSON object per line
- Easily parsable
- Streamable
- Compatible with log aggregation tools (ELK, Splunk)

### Configuration Format (JSON)
```json
{
  "monitoring": {
    "gateway": "auto",
    "timeout_seconds": 5
  }
}
```
- Human-readable
- Easy to edit
- Well-documented structure

## Error Handling Strategies

### Network Timeouts
```python
# Set timeout for all operations
socket.setdefaulttimeout(5)

# Per-operation timeout
response = urllib.request.urlopen(url, timeout=5)

# Subprocess timeout
subprocess.run(cmd, timeout=5)
```

### Retry Logic
```python
# Exponential backoff
for attempt in range(max_retries):
    try:
        result = test_connectivity()
        break
    except Exception:
        time.sleep(2 ** attempt)  # 1s, 2s, 4s, 8s
```

### Graceful Degradation
- If requests library unavailable → fallback to urllib
- If ping fails → try alternative tests
- If DNS fails → use IP addresses directly
- If config missing → use sensible defaults

## Performance Considerations

### Execution Time Targets
- Single test run: < 30 seconds
- Gateway ping: < 5 seconds
- DNS resolution: < 3 seconds
- HTTP check: < 10 seconds
- Log write: < 100ms

### Resource Usage
- CPU: Minimal (mostly I/O wait)
- Memory: < 50 MB
- Disk: ~1-2 MB per day of logs
- Network: ~10-20 KB per test cycle

### Optimization Techniques
- Parallel testing (threading/async)
- Connection pooling (HTTP sessions)
- Caching (gateway IP, DNS)
- Efficient log rotation
- Batch writes to reduce I/O

## Security Considerations

### Permissions
- No root/sudo required
- Read/write access to logs directory
- Network access required
- No sensitive data storage

### Data Privacy
- Logs contain IP addresses and timestamps
- No personal information
- No passwords or credentials
- Consider log retention policies

### Network Security
- Outbound connections only
- No listening sockets
- No data collection/telemetry
- Respects firewall rules

## Cron Integration

### Systemd Timer (Alternative to Cron)
```ini
# /etc/systemd/system/network-monitor.timer
[Unit]
Description=Network Monitor Timer

[Timer]
OnBootSec=1min
OnUnitActiveSec=5min

[Install]
WantedBy=timers.target
```

### Log Rotation (logrotate)
```
/home/user/network-check/logs/*.json {
    daily
    rotate 30
    compress
    delaycompress
    notifempty
    missingok
}
```

## References and Documentation

### Python Documentation
- https://docs.python.org/3/library/socket.html
- https://docs.python.org/3/library/subprocess.html
- https://docs.python.org/3/library/json.html
- https://docs.python.org/3/library/logging.html

### Network Protocols
- ICMP (ping): RFC 792
- DNS: RFC 1035
- HTTP: RFC 2616, RFC 7540 (HTTP/2)

### Best Practices
- Network monitoring guidelines
- Logging best practices
- Python code style (PEP 8)
- Error handling patterns

## Development Tools

### Testing
```bash
# Manual test
python3 network_monitor.py

# With verbose output
python3 network_monitor.py --verbose

# Test configuration
python3 -m json.tool config.json

# Syntax check
python3 -m py_compile network_monitor.py
```

### Debugging
```bash
# Trace network calls
strace -e trace=network python3 network_monitor.py

# Profile performance
python3 -m cProfile network_monitor.py

# Check log output
tail -f logs/network_monitor.log
```

## Version Requirements

### Minimum Versions
- Python: 3.7+ (f-strings, datetime.fromisoformat)
- Debian: 10 (Buster) or later
- Linux Kernel: 3.0+

### Tested Platforms
- Debian 10, 11, 12
- Ubuntu 20.04, 22.04, 24.04
- Raspberry Pi OS (Bullseye)

## Future Enhancements

### Planned Features
- Traceroute on failure
- MTR (My Traceroute) integration
- Email/SMS alerts
- Web dashboard
- Machine learning anomaly detection
- Integration with monitoring systems (Prometheus, Grafana)

### Potential Integrations
- Slack/Discord notifications
- MQTT publishing for home automation
- InfluxDB time-series storage
- Elasticsearch log aggregation
