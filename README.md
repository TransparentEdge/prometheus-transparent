# Prometheus Transparent

Easy ready-to-run docker-compose stack to get and parse metrics from Transparent Edge API.

## Getting started

- You need a valid `CID` and `CSECRET` pair. You can get them from T.E. Dashboard
  - Visit: https://dashboard.transparentedge.eu/user/profile  > (API Keys)
- Clone this repo and edit `custom/prometheus/prometheus-te.yml` file with your own secret data.
  - `git clone https://github.com/sistemasTES/prometheus-transparent.git`
  - `cd prometheus-transparent/ && vim custom/prometheus/prometheus-te.yml`
- Run it! 
  - `docker compose up -d`

## Folder structure
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

