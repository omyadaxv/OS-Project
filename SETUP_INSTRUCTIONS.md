# Real-Time Process Monitoring System

## Complete Setup Instructions

This document provides comprehensive instructions for setting up and running the Process Monitoring System on Windows and Linux.

---

## System Requirements

- **Node.js**: Version 18.0.0 or higher
- **npm**: Version 8.0.0 or higher
- **Operating System**: Windows 10+, Ubuntu 18.04+, CentOS 7+, or macOS 10.15+

---

## Project Structure

```
/agent                          # Agent (runs on monitored machines)
├── package.json
├── config.js                   # Agent configuration
├── agent.js                    # Main entry point
├── services/
│   ├── processCollector.js     # Collects process information
│   ├── systemStats.js          # Collects system metrics
│   ├── transport.js            # HTTP/WebSocket communication
│   └── commandExecutor.js      # Executes control commands
└── utils/
    └── logger.js               # Logging utility

/backend                        # Backend Server
├── package.json
├── config.js                   # Server configuration
├── server.js                   # Main entry point
├── routes/
│   ├── agents.js               # Agent management routes
│   ├── processes.js            # Process data routes
│   └── control.js              # Control command routes
├── controllers/
│   ├── agentsController.js     # Agent management logic
│   ├── processController.js    # Process data logic
│   └── controlController.js    # Control command logic
├── services/
│   ├── agentRegistry.js        # Manages connected agents
│   ├── dataIngestService.js    # Processes incoming telemetry
│   ├── commandService.js       # Handles control commands
│   └── anomalyService.js       # Detects anomalies
└── utils/
    ├── logger.js               # Logging utility
    └── authMiddleware.js       # Authentication middleware
```

---

## Backend Setup

### 1. Navigate to Backend Directory

```bash
cd backend
```

### 2. Install Dependencies

```bash
npm install
```

### 3. Configure Environment (Optional)

Create a `.env` file in the backend directory:

```env
# Server Configuration
PORT=3001
HOST=0.0.0.0

# Security
API_KEY=your-secure-api-key-here

# CORS (comma-separated origins or * for all)
CORS_ORIGIN=*

# Data Retention
TELEMETRY_TTL=300
MAX_SNAPSHOTS=60

# Anomaly Detection Thresholds
CPU_THRESHOLD=90
MEMORY_THRESHOLD=90
CPU_SPIKE_THRESHOLD=50

# Logging
LOG_LEVEL=info

# MongoDB (Optional)
MONGODB_ENABLED=false
MONGODB_URI=mongodb://localhost:27017/process_monitor

# Agent Timeouts
AGENT_TIMEOUT=10000
AGENT_CLEANUP=60000
```

### 4. Start the Backend Server

**Development mode (with auto-reload):**
```bash
npm run dev
```

**Production mode:**
```bash
npm start
```

The server will start and display:
- HTTP API: `http://0.0.0.0:3001/api`
- WebSocket: `ws://0.0.0.0:3001`
- Health Check: `http://0.0.0.0:3001/health`

---

## Agent Setup

### 1. Navigate to Agent Directory

```bash
cd agent
```

### 2. Install Dependencies

```bash
npm install
```

### 3. Configure Environment

Create a `.env` file in the agent directory or set environment variables:

```env
# Agent Identity (auto-generated if not set)
AGENT_ID=agent-001

# Backend Connection
BACKEND_HTTP_URL=http://your-backend-server:3001
BACKEND_WS_URL=http://your-backend-server:3001

# Security
API_KEY=your-secure-api-key-here

# Collection Settings
COLLECTION_INTERVAL=2000
MAX_PROCESSES=100

# Transport Mode: 'websocket' or 'http'
TRANSPORT_MODE=websocket

# Logging
LOG_LEVEL=info
```

### 4. Start the Agent

**Development mode:**
```bash
npm run dev
```

**Production mode:**
```bash
npm start
```

---

## Platform-Specific Instructions

### Windows Setup

1. **Install Node.js:**
   - Download from https://nodejs.org/
   - Run the installer
   - Verify: `node --version` and `npm --version`

2. **Run as Administrator (for full process access):**
   - Right-click Command Prompt → "Run as administrator"
   - Navigate to agent directory and run

3. **Install as Windows Service (optional):**
   ```bash
   npm install -g node-windows
   ```
   
   Create `install-service.js`:
   ```javascript
   const Service = require('node-windows').Service;
   const svc = new Service({
     name: 'Process Monitor Agent',
     description: 'Real-time process monitoring agent',
     script: require('path').join(__dirname, 'agent.js')
   });
   svc.on('install', () => svc.start());
   svc.install();
   ```

### Linux Setup

1. **Install Node.js:**
   ```bash
   # Ubuntu/Debian
   curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
   sudo apt-get install -y nodejs

   # CentOS/RHEL
   curl -fsSL https://rpm.nodesource.com/setup_18.x | sudo bash -
   sudo yum install -y nodejs
   ```

2. **Run with elevated privileges (for full process access):**
   ```bash
   sudo npm start
   ```

3. **Create systemd service:**
   
   Create `/etc/systemd/system/process-monitor-agent.service`:
   ```ini
   [Unit]
   Description=Process Monitor Agent
   After=network.target

   [Service]
   Type=simple
   User=root
   WorkingDirectory=/opt/process-monitor/agent
   ExecStart=/usr/bin/node agent.js
   Restart=always
   RestartSec=10
   Environment=NODE_ENV=production
   Environment=BACKEND_WS_URL=http://your-backend:3001
   Environment=API_KEY=your-api-key

   [Install]
   WantedBy=multi-user.target
   ```

   Enable and start:
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable process-monitor-agent
   sudo systemctl start process-monitor-agent
   ```

---

## API Reference

### Agent Management

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/agents/register` | Register new agent |
| GET | `/api/agents/list` | List all agents |
| GET | `/api/agents/stats` | Get agent statistics |
| GET | `/api/agents/:agentId` | Get agent details |
| DELETE | `/api/agents/:agentId` | Unregister agent |

### Process Data

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/processes/live` | Get live process data |
| GET | `/api/processes/top` | Get top processes by resource |
| GET | `/api/processes/agent/:agentId` | Get processes for agent |
| GET | `/api/processes/history/:agentId` | Get historical metrics |
| GET | `/api/processes/:agentId/:pid` | Get specific process |

### Control Commands

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/processes/:agentId/:pid/kill` | Kill a process |
| POST | `/api/processes/:agentId/:pid/restart` | Restart a process |
| POST | `/api/processes/:agentId/:pid/renice` | Change process priority |
| GET | `/api/alerts` | Get active alerts |
| POST | `/api/alerts/:alertId/acknowledge` | Acknowledge an alert |

### WebSocket Events

**From Agent to Backend:**
- `agent:register` - Register agent
- `telemetry` - Send telemetry data
- `command:result` - Send command result

**From Backend to Frontend:**
- `telemetry` - Real-time process data
- `alert` - Anomaly alerts

**From Backend to Agent:**
- `command` - Control command

---

## How It All Connects

### Data Flow

```
┌─────────────────┐     WebSocket/HTTP      ┌──────────────────┐     WebSocket      ┌──────────────┐
│     Agent       │ ──────────────────────► │     Backend      │ ─────────────────► │   Frontend   │
│  (on machine)   │                         │    (server)      │                    │  (dashboard) │
└─────────────────┘                         └──────────────────┘                    └──────────────┘
        │                                           │
        │ Collects:                                 │ Provides:
        │ • Process list                            │ • REST API
        │ • CPU/Memory usage                        │ • WebSocket broadcast
        │ • System metrics                          │ • Anomaly detection
        │                                           │ • Command routing
        ▼                                           ▼
   Sends every 2s                             Stores & broadcasts
```

### Connection Process

1. **Agent starts** and connects to backend via WebSocket (or HTTP fallback)
2. **Agent registers** with unique ID, hostname, and platform info
3. **Agent collects** process and system metrics every 2 seconds
4. **Agent sends telemetry** to backend
5. **Backend processes** data, detects anomalies, stores snapshots
6. **Backend broadcasts** to all connected frontend clients
7. **Frontend displays** real-time data
8. **User sends command** (e.g., kill process) via frontend
9. **Backend routes command** to appropriate agent
10. **Agent executes** command and sends result back

---

## Troubleshooting

### Agent Not Connecting

1. Check backend URL in agent config
2. Verify API key matches
3. Check firewall allows port 3001
4. Try HTTP mode: `TRANSPORT_MODE=http`

### Permission Errors

- **Windows**: Run as Administrator
- **Linux**: Run with `sudo` or as root

### High CPU Usage by Agent

- Increase `COLLECTION_INTERVAL` (e.g., 5000ms)
- Reduce `MAX_PROCESSES` limit

### WebSocket Connection Drops

- Check network stability
- Agent will auto-reconnect
- Use HTTP mode as fallback

---

## Security Recommendations

1. **Change default API key** in production
2. **Use HTTPS** with reverse proxy (nginx/Apache)
3. **Restrict CORS** to specific origins
4. **Use firewall** to limit access to backend
5. **Run agent** with minimal required privileges
6. **Rotate API keys** periodically

---

## Monitoring Multiple Machines

Deploy the agent on each machine you want to monitor:

1. Copy agent directory to each machine
2. Configure unique `AGENT_ID` for each
3. Point all agents to the same backend server
4. Start agents on all machines

The backend will aggregate data from all agents and the frontend will display a unified view.
