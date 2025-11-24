# Prometheus Transparent

Easy ready-to-run docker-compose stack to get and parse metrics from Transparent Edge API.

## Prerequisites

- Docker and Docker Compose installed
- A valid `CID` and `CSECRET` pair from Transparent Edge Dashboard
  - Visit: https://dashboard.transparentedge.eu/user/profile > (API Keys)

## Getting Started

1. Clone this repository:
   ```bash
   git clone https://github.com/sistemasTES/prometheus-transparent.git
   cd prometheus-transparent/
   ```

2. Configure your API credentials:
   ```bash
   vim custom/prometheus/prometheus-te.yml
   ```
   Add your `CID` and `CSECRET` to the configuration file.

3. Start the stack:
   ```bash
   docker compose up -d
   ```

## Folder Structure

```
│
├── README.md
├── custom
│   ├── grafana
│   │   ├── hosts
│   │   └── provisioning
│   │       ├── dashboards
│   │       │   ├── CDN_Metrics-Comprehensive_Dashboard-internally_cassic.json
│   │       │   └── dashboards.yml
│   │       └── datasources
│   │           └── datasource.yml
│   └── prometheus
│       ├── hosts-te
│       └── prometheus-te.yml
├── deploy.sh
└── docker-compose.yml
```

## Documentation

For detailed information about available metrics, see [prometheus-metrics.md](prometheus-metrics.md).
