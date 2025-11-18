# Prometheus Transparent

Easy ready-to-run docker-compose stack to get and parse metrics from Transparent Edge API.

## Getting started

- You need a valid `CID` and `CSECRET` pair. You can get them from T.E. Dashboard
- Clone this repo and edit the `datasource.yml` file with your own secret data.
- Run it! with `docker compose up -d`

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

