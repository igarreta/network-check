# Network Connectivity Monitor

A Python-based network monitoring solution designed to diagnose intermittent connectivity issues in mesh network environments (specifically Deco mesh routers).

## Problem

Home networks with mesh routers sometimes experience intermittent disconnections where machines temporarily lose connection and regain it later. This makes troubleshooting very difficult because errors are sporadic and unpredictable.

## Solution

This project provides two Python scripts:

1. **`network_monitor.py`** - Continuously tests network connectivity at multiple layers and logs results
2. **`analyze_logs.py`** - Analyzes logs to identify patterns, summarize errors, and facilitate troubleshooting

## Features

### Multi-Layer Testing
- **Gateway/Router** - Ping tests to detect local mesh network issues
- **DNS Resolution** - Tests multiple DNS servers to identify name resolution failures
- **HTTP Connectivity** - Validates internet access to external services
- **Latency & Quality** - Measures response times and packet loss

### Comprehensive Logging
- Structured JSON format for easy parsing
- Timestamped entries with detailed metrics
- Daily log rotation with configurable retention
- Separate execution logs for debugging

### Intelligent Analysis
- Identifies disconnection events and duration
- Detects patterns (time-of-day, day-of-week)
- Calculates uptime statistics by layer
- Distinguishes between local mesh vs ISP issues
- Generates summary reports

## Quick Start

### Prerequisites
- Debian/Linux server (or any system with Python 3.7+)
- Network connectivity
- Cron or systemd timer

### Installation

```bash
# Clone the repository
git clone <repository-url>
cd network-check

# Optional: Install enhanced dependencies
pip3 install -r requirements.txt

# Make scripts executable
chmod +x network_monitor.py analyze_logs.py
```

### Configuration

Edit `config.json` to customize:
- DNS servers to test
- HTTP endpoints to monitor
- Ping count and timeouts
- Log retention policies
- Alert thresholds

### Running Manually

```bash
# Run a single connectivity test
python3 network_monitor.py

# Analyze logs and generate report
python3 analyze_logs.py

# Get detailed statistics
python3 analyze_logs.py --detailed

# Analyze specific date range
python3 analyze_logs.py --from 2025-11-01 --to 2025-11-21
```

### Automated Monitoring (Cron)

Add to your crontab (`crontab -e`):

```bash
# Monitor every 5 minutes
*/5 * * * * /usr/bin/python3 /home/user/network-check/network_monitor.py

# Generate daily report at 7 AM
0 7 * * * /usr/bin/python3 /home/user/network-check/analyze_logs.py --summary
```

## Project Structure

```
network-check/
├── README.md                  # This file
├── PROJECT_PLAN.md           # Detailed project plan
├── LICENSE                   # License file
├── requirements.txt          # Python dependencies (optional)
├── config.json               # Configuration file
├── network_monitor.py        # Main monitoring script
├── analyze_logs.py           # Log analysis script
├── logs/                     # Log directory (auto-created)
│   ├── network_YYYYMMDD.json # Daily connectivity logs
│   └── network_monitor.log   # Script execution log
└── reports/                  # Analysis reports (auto-created)
    └── summary_YYYYMMDD.txt  # Daily summaries
```

## Output Examples

### Monitoring Log Entry
```json
{
  "timestamp": "2025-11-21T10:30:45.123456",
  "test_type": "ping_gateway",
  "target": "192.168.68.1",
  "status": "success",
  "latency_ms": 12.5,
  "details": {
    "packets_sent": 3,
    "packets_received": 3,
    "packet_loss_pct": 0.0
  }
}
```

### Analysis Report
```
Network Connectivity Summary (2025-11-21)
==========================================

Overall Uptime: 98.5%
Total Tests: 288
Failed Tests: 4

Disconnection Events: 2
  Event 1: 10:30 - 10:35 (5 min) - Gateway unreachable
  Event 2: 15:15 - 15:18 (3 min) - DNS resolution failed

Root Cause Analysis:
  - Local Mesh Issues: 1 event (5 min downtime)
  - DNS Problems: 1 event (3 min downtime)
  - ISP Issues: 0 events

Recommendations:
  - Check Deco unit placement (gateway timeouts detected)
  - Consider alternative DNS servers
```

## Troubleshooting

### Gateway Not Detected
If auto-detection fails, manually set the gateway IP in `config.json`:
```json
{
  "monitoring": {
    "gateway": "192.168.68.1"
  }
}
```

### Permission Errors
Ensure the script has permissions to create directories and write logs:
```bash
chmod +x network_monitor.py
chmod 755 logs/ reports/
```

### Script Hangs
All network operations have timeouts. If issues persist, reduce `timeout_seconds` in `config.json`.

## Common Deco Mesh Issues

Based on research, common causes of intermittent disconnections include:
- Firmware problems (check for updates in Deco app)
- Poor node placement (units too far apart)
- Network overload (too many devices)
- Weak wireless backhaul (consider wired backhaul)
- Interference from other devices

## Contributing

Contributions welcome! Please see PROJECT_PLAN.md for architecture details.

## License

See LICENSE file for details.

## Support

For issues specific to:
- **This software**: Open an issue in the repository
- **Deco hardware**: Contact TP-Link support or visit community forums
- **Network configuration**: Consult your ISP or network administrator
