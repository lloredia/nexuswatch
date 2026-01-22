# NexusWatch

**SIEM Dashboard • SecOps Command Center Component 3**

NexusWatch is a centralized Security Information and Event Management (SIEM) dashboard designed to aggregate, correlate, and visualize security events from across your infrastructure. As the third component of the SecOps Command Center, it integrates seamlessly with HoneyTrap (honeypot network) and SentinelForge (threat intelligence platform) to provide real-time security monitoring and incident visibility.

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                         NEXUSWATCH SIEM                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐               │
│  │   HoneyTrap  │  │ SentinelForge│  │   Firewall   │               │
│  │   SSH/HTTP   │  │  IOC Feed    │  │    Logs      │               │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘               │
│         │                  │                  │                      │
│         └──────────────────┼──────────────────┘                      │
│                            ▼                                         │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    Event Ingestion Layer                     │    │
│  │  • Syslog/JSON/CEF parsing  • Normalization  • Enrichment   │    │
│  └─────────────────────────────┬───────────────────────────────┘    │
│                                ▼                                     │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    Analytics Engine                          │    │
│  │  • Real-time correlation  • Pattern detection  • Scoring    │    │
│  └─────────────────────────────┬───────────────────────────────┘    │
│                                ▼                                     │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    Dashboard Interface                       │    │
│  │  • Metrics  • Event Stream  • Timeline  • Threat Map        │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Features

### Real-Time Event Monitoring
- Live event stream with automatic updates (3-second intervals)
- Severity-based color coding (Critical, High, Medium, Low)
- IOC match indicators integrated from SentinelForge
- Threat scoring for each event

### Metrics Dashboard
- **Total Events (24h)**: Aggregate event count with trend indicators
- **Critical Alerts**: High-priority security incidents requiring immediate attention
- **Active Threats**: Currently unresolved security threats
- **Blocked IPs**: Count of IPs blocked by automated response
- **IOC Matches**: Events matching known indicators of compromise
- **Events/sec**: Real-time throughput measurement
- **Avg Response**: Mean incident response time
- **Data Sources**: Active security data feeds

### Visualization
- **Event Timeline**: 24-hour histogram showing event distribution by severity
- **Threat Map**: Top attack origin IP ranges with frequency analysis
- **Data Source Status**: Real-time health monitoring of all integrated sources

### Event Investigation
- Click any event to view detailed information
- Quick actions: Block IP, Investigate, Resolve
- Full event context including source, destination, threat score

### Data Sources
The dashboard aggregates events from multiple security tools:
- HoneyTrap-SSH (honeypot network)
- HoneyTrap-HTTP (honeypot network)
- SentinelForge (threat intelligence)
- Firewall logs
- IDS/IPS alerts
- EDR telemetry
- WAF events
- DNS monitoring

---

## Installation

### Prerequisites
- Node.js 18+ 
- npm or yarn
- HoneyTrap and SentinelForge deployed (optional, for full integration)

### Quick Start

```bash
# Clone or navigate to the project
cd nexuswatch

# Install dependencies
npm install

# Start development server
npm run dev

# Build for production
npm run build
```

### Docker Deployment

```dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
```

```bash
docker build -t nexuswatch:latest .
docker run -d -p 3000:80 nexuswatch:latest
```

---

## Configuration

### Environment Variables

```env
# API Endpoints
VITE_HONEYTRAP_API=http://localhost:8080
VITE_SENTINELFORGE_API=http://localhost:3001

# Feature Flags
VITE_ENABLE_LIVE_MODE=true
VITE_REFRESH_INTERVAL=3000

# Authentication (for production)
VITE_AUTH_ENABLED=false
```

### Data Source Integration

NexusWatch expects events in the following JSON format:

```json
{
  "id": "EVT-000001",
  "timestamp": "2026-01-22T10:30:00Z",
  "type": "INTRUSION_ATTEMPT",
  "severity": "critical",
  "source": "HoneyTrap-SSH",
  "sourceIp": "185.220.101.45",
  "destIp": "10.0.1.50",
  "destPort": 22,
  "status": "active",
  "details": "Multiple failed SSH authentication attempts detected",
  "iocMatch": true,
  "threatScore": 95
}
```

---

## Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `Ctrl+L` | Toggle live mode on/off |
| `Escape` | Close event detail modal |

---

## Integration with SecOps Command Center

NexusWatch is designed to work as part of the larger SecOps ecosystem:

### HoneyTrap → NexusWatch
```python
# Example: Forward honeypot events to SIEM
import requests

event = {
    "type": "BRUTE_FORCE",
    "severity": "high",
    "source": "HoneyTrap-SSH",
    "sourceIp": attacker_ip,
    "details": f"Brute force attack detected: {attempt_count} attempts"
}

requests.post("http://nexuswatch:3000/api/events", json=event)
```

### SentinelForge → NexusWatch
```python
# Example: IOC enrichment integration
def enrich_event(event):
    # Check against SentinelForge IOC database
    ioc_response = requests.get(
        f"http://sentinelforge:3001/api/iocs/search",
        params={"ip": event["sourceIp"]}
    )
    
    if ioc_response.json().get("matches"):
        event["iocMatch"] = True
        event["threatScore"] = min(100, event["threatScore"] + 20)
    
    return event
```

---

## Roadmap

NexusWatch is Component 3 of the SecOps Command Center. Remaining components:

| # | Component | Status | Description |
|---|-----------|--------|-------------|
| 1 | HoneyTrap | ✅ Complete | Distributed honeypot network |
| 2 | SentinelForge | ✅ Complete | Threat intelligence aggregator |
| 3 | **NexusWatch** | ✅ Complete | SIEM Dashboard |
| 4 | Incident Response Orchestrator | 🔜 Planned | Automated response workflows |
| 5 | Compliance Engine | 🔜 Planned | Regulatory adherence tracking |

---

## API Reference

### Event Ingestion

```
POST /api/events
Content-Type: application/json

{
  "type": "string",
  "severity": "critical|high|medium|low",
  "source": "string",
  "sourceIp": "string",
  "destIp": "string",
  "destPort": "number",
  "details": "string"
}
```

### Query Events

```
GET /api/events?severity=critical&source=HoneyTrap-SSH&limit=100
```

### Metrics

```
GET /api/metrics
```

---

## Security Considerations

- Deploy behind a reverse proxy with TLS
- Enable authentication for production environments
- Restrict network access to internal security team
- Audit log access regularly
- Do not expose to public internet

---

## License

MIT License - Part of the SecOps Command Center project.

---

## Credits

Built as part of the SecOps Command Center cybersecurity infrastructure project.

**Components:**
- 🍯 HoneyTrap - Deception technology
- 🛡️ SentinelForge - Threat intelligence
- 👁️ NexusWatch - SIEM Dashboard
