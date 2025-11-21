# Network Connectivity Monitor - Project Plan

## Overview
A Python-based network monitoring solution to diagnose intermittent connectivity issues in a Deco mesh network environment.

## Problem Statement
- Home network with Deco mesh routers experiencing intermittent disconnections
- Machines temporarily lose connection and regain it later
- Troubleshooting is difficult due to intermittent nature of errors
- Need automated monitoring and analysis to identify patterns

## Solution Architecture

### Component 1: Network Monitor (`network_monitor.py`)
**Purpose**: Continuously test network connectivity and log results

**Test Layers**:
1. **Layer 1 - Gateway/Router** (Local Network)
   - Ping default gateway (Deco router)
   - Detects mesh network issues

2. **Layer 2 - DNS Resolution** (Name Resolution)
   - Test multiple DNS servers (8.8.8.8, 1.1.1.1, gateway DNS)
   - Detects DNS service failures

3. **Layer 3 - External Connectivity** (Internet)
   - HTTP(S) requests to reliable endpoints
   - Detects ISP/internet connectivity issues

4. **Layer 4 - Latency & Quality**
   - Measure response times
   - Track packet loss
   - Identify degraded performance

**Execution**: Run via cron every 1-5 minutes

### Component 2: Log Analyzer (`analyze_logs.py`)
**Purpose**: Parse logs and generate actionable insights

**Analysis Features**:
1. **Disconnection Events**
   - Identify outage periods
   - Calculate duration
   - Determine affected layers

2. **Pattern Recognition**
   - Time-of-day analysis
   - Day-of-week patterns
   - Frequency trends

3. **Statistics & Metrics**
   - Uptime percentage by layer
   - Mean Time Between Failures (MTBF)
   - Mean Time To Recovery (MTTR)
   - Latency statistics (min/max/avg/p95)

4. **Root Cause Identification**
   - Local mesh vs ISP issues
   - DNS-specific problems
   - Gradual degradation patterns

## Technology Stack

### Python Standard Library
- `subprocess` - Execute ping commands
- `socket` - Low-level network testing, DNS resolution
- `urllib` / `requests` - HTTP connectivity tests
- `json` - Structured logging format
- `argparse` - Command-line argument parsing
- `datetime` - Timestamp handling
- `statistics` - Calculate metrics
- `platform` - Cross-platform ping commands
- `logging` - Structured logging framework

### External Libraries (Optional)
- `requests` - More robust HTTP client (if available)
- `pandas` - Advanced log analysis (if available)
- `matplotlib` / `plotly` - Visualization (future enhancement)

### File Structure
```
network-check/
├── README.md                  # Setup and usage instructions
├── PROJECT_PLAN.md           # This file
├── requirements.txt          # Python dependencies
├── config.json               # Configuration file
├── network_monitor.py        # Main monitoring script
├── analyze_logs.py           # Log analysis script
├── logs/                     # Log directory (created at runtime)
│   ├── network_YYYYMMDD.json # Daily log files
│   └── network_monitor.log   # Script execution log
└── reports/                  # Analysis reports (created at runtime)
    └── summary_YYYYMMDD.txt  # Daily summaries
```

## Data Structure

### Log Entry Format (JSON)
```json
{
  "timestamp": "2025-11-21T10:30:45.123456",
  "test_type": "ping_gateway|dns_lookup|http_check|latency",
  "target": "192.168.1.1|8.8.8.8|https://www.google.com",
  "status": "success|failure|timeout|degraded",
  "latency_ms": 12.5,
  "details": {
    "error_message": "Network unreachable",
    "packets_sent": 3,
    "packets_received": 3,
    "packet_loss_pct": 0.0,
    "dns_server": "8.8.8.8",
    "resolved_ip": "142.250.185.78",
    "http_status": 200
  }
}
```

### Configuration Format (JSON)
```json
{
  "monitoring": {
    "gateway": "auto",
    "dns_servers": ["8.8.8.8", "1.1.1.1", "gateway"],
    "http_endpoints": [
      "https://www.google.com",
      "https://www.cloudflare.com",
      "https://www.amazon.com"
    ],
    "ping_count": 3,
    "timeout_seconds": 5,
    "log_directory": "./logs",
    "log_retention_days": 30
  },
  "thresholds": {
    "latency_warning_ms": 100,
    "latency_critical_ms": 500,
    "packet_loss_warning_pct": 5,
    "packet_loss_critical_pct": 20
  }
}
```

## Test Targets

### Gateway Detection
- Auto-detect default gateway using `ip route` or `netstat`
- Fallback to common Deco IPs: 192.168.68.1, 192.168.1.1

### DNS Servers
- Google DNS: 8.8.8.8, 8.8.4.4
- Cloudflare DNS: 1.1.1.1, 1.0.0.1
- Gateway DNS (from DHCP)

### HTTP Endpoints
- Google: https://www.google.com
- Cloudflare: https://www.cloudflare.com
- Amazon: https://www.amazon.com
- Criteria: High availability, geographic distribution

## Implementation Phases

### Phase 1: Core Monitoring ✓ (To Implement)
- Gateway ping test
- Basic DNS resolution
- HTTP connectivity check
- JSON logging

### Phase 2: Enhanced Testing ✓ (To Implement)
- Multiple DNS servers
- Latency measurements
- Packet loss tracking
- Configuration file support

### Phase 3: Analysis Tools ✓ (To Implement)
- Log parsing
- Disconnection detection
- Basic statistics
- Summary reports

### Phase 4: Advanced Analysis (Future)
- Pattern recognition
- Visualization
- Alerting (email/SMS)
- Web dashboard

## Success Metrics

1. **Monitoring Reliability**
   - Script runs without errors for 7+ days
   - Logs capture all connectivity events
   - Minimal false positives

2. **Diagnostic Value**
   - Can identify root cause (mesh vs ISP)
   - Provides actionable insights
   - Reduces troubleshooting time

3. **Performance**
   - Test execution < 30 seconds
   - Minimal resource usage
   - Cron-friendly (no hanging processes)

## Deployment

### Cron Schedule Examples
```bash
# Every 5 minutes
*/5 * * * * /usr/bin/python3 /path/to/network_monitor.py

# Every minute (aggressive monitoring)
* * * * * /usr/bin/python3 /path/to/network_monitor.py

# Daily analysis at 7 AM
0 7 * * * /usr/bin/python3 /path/to/analyze_logs.py --summary
```

### System Requirements
- Debian/Linux server (always-on)
- Python 3.7+
- Network access
- Cron/systemd timer
- ~10MB storage per week of logs

## Risk Mitigation

### Potential Issues & Solutions
1. **Gateway IP changes**: Auto-detection + configuration override
2. **Network timeouts hang**: All operations have timeouts
3. **Disk space**: Log rotation and retention policies
4. **Permissions**: Script validation checks
5. **DNS caching**: Test multiple DNS servers

## Next Steps
1. ✓ Complete project plan
2. Implement network_monitor.py
3. Test monitoring on actual network
4. Implement analyze_logs.py
5. Create documentation
6. Deploy to production
