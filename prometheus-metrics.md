# Prometheus Metrics Exporter

## Overview

Our Prometheus exporter provides comprehensive metrics about CDN services including delivery, security (WAF/DDoS), and bot mitigation. All metrics are collected from Elasticsearch and exposed in standard Prometheus format.

**Endpoint:** `/v1/statistics/metrics/<company_id>/prometheus`  
**Authentication:** Required  
**Service Requirement:** Company must have `PrometheusMetrics` service enabled  
**Format:** Prometheus text exposition format (version 0.0.4)  
**Metric Type:** All metrics are exposed as Gauges (snapshots of current state)

## Quick Start

### 1. Configure Prometheus

Add this to your `prometheus.yml`:

```yaml
global:
  scrape_interval: 30s
  scrape_timeout: 10s

scrape_configs:
  - job_name: "cdn_metrics"
    metrics_path: "/v1/statistics/metrics/<company_id>/prometheus"  # Replace with company_id
    static_configs:
      - targets: ["api.transparentcdn.com"]
    scheme: https

    # Add oauth2 authentication
    oauth2:
      client_id: "MY_CLIENT_ID"
      client_secret: "MY_CLIENT_SECRET"
      token_url: "https://api.transparentcdn.com/v1/oauth2/access_token/"
      scopes:
        - "read"
```
> **Note:** Each company needs a separate scrape job with its own company_id in the path.

### Multi-Company Setup

For monitoring multiple companies, add multiple scrape jobs:

```yaml
scrape_configs:
  - job_name: "cdn_metrics_company_<company_id>"
    scrape_interval: 30s
    metrics_path: "/v1/statistics/metrics/<company_id>/prometheus"
    static_configs:
      - targets: ["api.transparentcdn.com"]

    # Add oauth2 authentication
    oauth2:
      client_id: "MY_CLIENT_ID"
      client_secret: "MY_CLIENT_SECRET"
      token_url: "https://api.transparentcdn.com/v1/oauth2/access_token/"
      scopes:
        - "read"

  - job_name: "cdn_metrics_company_<company_id>"
    scrape_interval: 30s
    metrics_path: "/v1/statistics/metrics/<company_id>/prometheus"
    static_configs:
      - targets: ["api.transparentcdn.com"]

    # Add oauth2 authentication
    oauth2:
      client_id: "MY_CLIENT_ID"
      client_secret: "MY_CLIENT_SECRET"
      token_url: "https://api.transparentcdn.com/v1/oauth2/access_token/"
      scopes:
        - "read"
```

Or use Prometheus service discovery for dynamic company management.

### 2. Verify Metrics

```bash
# Test the endpoint (replace with actual company_id)
curl -H "Authorization: Bearer YOUR_TOKEN" \
  https://api.transparentcdn.com/v1/statistics/metrics/<company_id>/prometheus

# Check Prometheus targets
# Open http://prometheus:9090/targets
```

### 3. Query Metrics

```promql
# Average response time for a specific company
delivery_avg_response_time_seconds{company_id="<company_id>"}

# Total bandwidth by site for a company
sum by (vhost) (delivery_bandwidth_bytes_total{company_id="<company_id>"})

# Cache hit ratio for a company
sum(delivery_cache_requests_total{company_id="<company_id>",cache_status="hit"}) / 
sum(delivery_requests_total{company_id="<company_id>"}) * 100
```

---

## Available Metrics

### Delivery Service (30+ base metrics)

All delivery metrics use these standard labels: `company_id`, `company_name`, `vhost`

#### Core Metrics
| Metric | Type | Description | Labels |
|--------|------|-------------|--------|
| `delivery_requests_total` | gauge | Total delivery requests in time window | company_id, company_name, vhost |
| `delivery_requests_rate_total` | gauge | Delivery requests per second | company_id, company_name, vhost |
| `delivery_bandwidth_bytes_total` | gauge | Total bandwidth served in bytes | company_id, company_name, vhost |
| `delivery_bandwidth_bytes_rate_total` | gauge | Bandwidth served in bytes per second | company_id, company_name, vhost |
| `delivery_avg_response_time_seconds` | gauge | Average response time in seconds | company_id, company_name, vhost |
| `delivery_response_time_seconds_bucket` | histogram | Response time distribution buckets | company_id, company_name, vhost, le |
| `delivery_response_time_seconds_sum` | histogram | Sum of all response times | company_id, company_name, vhost |
| `delivery_response_time_seconds_count` | histogram | Count of response time observations | company_id, company_name, vhost |

#### Cache Performance
| Metric | Type | Description | Additional Labels |
|--------|------|-------------|-------------------|
| `delivery_cache_requests_total` | gauge | Requests by cache status | cache_status |
| `delivery_cache_bytes_total` | gauge | Bytes served by cache status | cache_status |

**Cache Status Values:** `hit`, `miss`, `pass`

#### HTTP Status Codes
| Metric | Type | Description | Additional Labels |
|--------|------|-------------|-------------------|
| `delivery_status_code_total` | gauge | Requests by HTTP status code | code |
| `delivery_status_code_rate_total` | gauge | Requests per second by status code | code |

**Note:** Status codes are dynamic based on actual traffic (e.g., 200, 301, 404, 500, etc.)

#### HTTP Methods
| Metric | Type | Description | Additional Labels |
|--------|------|-------------|-------------------|
| `delivery_method_total` | gauge | Requests by HTTP method | method |

**Method Values:** `GET`, `POST`, `HEAD`, `PUT`, `DELETE`, `OPTIONS`, etc.

#### Protocols
| Metric | Type | Description | Additional Labels |
|--------|------|-------------|-------------------|
| `delivery_proto_total` | gauge | Requests by protocol | protocol |

**Protocol Values:** `http`, `https`

#### HTTP Versions
| Metric | Type | Description | Additional Labels |
|--------|------|-------------|-------------------|
| `delivery_http_version_total` | gauge | Requests by HTTP version | version |

**Version Values:** `HTTP/1.0`, `HTTP/1.1`, `HTTP/2`, `HTTP/3`

#### Content Types
| Metric | Type | Description | Additional Labels |
|--------|------|-------------|-------------------|
| `delivery_content_type_html_total` | gauge | HTML content requests | content_type |
| `delivery_content_type_json_total` | gauge | JSON content requests | content_type |
| `delivery_content_type_image_total` | gauge | Image content requests | content_type |
| `delivery_content_type_video_total` | gauge | Video content requests | content_type |
| `delivery_content_type_css_total` | gauge | CSS content requests | content_type |
| `delivery_content_type_javascript_total` | gauge | JavaScript content requests | content_type |
| `delivery_content_type_other_total` | gauge | Other content requests | content_type |

**Note:** Content type classification is based on substring matching in the actual Content-Type header

#### Object Sizes (by Average)
| Metric | Type | Description |
|--------|------|-------------|
| `delivery_object_size_1kb_total` | gauge | Requests with average object size ≤ 1KB |
| `delivery_object_size_10kb_total` | gauge | Requests with average object size ≤ 10KB |
| `delivery_object_size_100kb_total` | gauge | Requests with average object size ≤ 100KB |
| `delivery_object_size_1mb_total` | gauge | Requests with average object size ≤ 1MB |
| `delivery_object_size_10mb_total` | gauge | Requests with average object size ≤ 10MB |
| `delivery_object_size_100mb_total` | gauge | Requests with average object size ≤ 100MB |
| `delivery_object_size_larger_total` | gauge | Requests with average object size > 100MB |

**Note:** Object size metrics are calculated based on average object size per site (total_bytes / total_requests)

### WAF Metrics (3 metrics)

All WAF metrics use these standard labels: `company_id`, `company_name`, `vhost`

| Metric | Type | Description | Labels |
|--------|------|-------------|--------|
| `waf_requests_total` | gauge | Total WAF evaluated requests | company_id, company_name, vhost |
| `waf_requests_rate_total` | gauge | WAF requests per second | company_id, company_name, vhost |
| `waf_requests_by_country_total` | gauge | WAF requests by country | company_id, company_name, vhost, country_code |

**Country Code:** ISO 3166-1 alpha-2 format (e.g., `US`, `ES`, `FR`)

### DDoS Metrics (5 metrics)

All DDoS metrics use these standard labels: `company_id`, `company_name`, `vhost`

| Metric | Type | Description | Labels |
|--------|------|-------------|--------|
| `ddos_requests_total` | gauge | Total DDoS evaluated requests | company_id, company_name, vhost |
| `ddos_requests_rate_total` | gauge | DDoS requests per second | company_id, company_name, vhost |
| `ddos_action_total` | gauge | DDoS requests by mitigation action | company_id, company_name, vhost, action |
| `ddos_mitigated_bytes_total` | gauge | Bytes from mitigated attacks | company_id, company_name, vhost |
| `ddos_mitigated_bytes_rate_total` | gauge | Mitigated bytes per second | company_id, company_name, vhost |

### Bot Mitigation Metrics (5 metrics)

All bot mitigation metrics use these standard labels: `company_id`, `company_name`, `vhost`  
**Note:** The vhost label corresponds to the `host` field in bot mitigation logs

| Metric | Type | Description | Labels |
|--------|------|-------------|--------|
| `bot_requests_total` | gauge | Total bot evaluated requests | company_id, company_name, vhost |
| `bot_requests_rate_total` | gauge | Bot requests per second | company_id, company_name, vhost |
| `bot_requests_by_action_total` | gauge | Bot requests by action taken | company_id, company_name, vhost, action |
| `bot_requests_by_reason_total` | gauge | Bot requests by detection reason | company_id, company_name, vhost, reason |
| `bot_requests_by_country_total` | gauge | Bot requests by country | company_id, company_name, vhost, country_code |

**Action Values:** Dynamic based on bot mitigation actions (e.g., `challenge`, `block`, `allow`)  
**Reason Values:** Dynamic based on detection reasons  
**Country Code:** ISO 3166-1 alpha-2 format (e.g., `US`, `ES`, `CN`)

---

**Total Metrics:** 40+ base metrics with dynamic labels that expand based on actual traffic patterns

---

## Understanding Metric Types and Time Windows

### Why Gauges Instead of Counters?

Unlike typical Prometheus exporters that expose counters (ever-increasing values), our metrics use **Gauges** because:

1. **Data Source**: We aggregate data from Elasticsearch over time windows
2. **Window-based Collection**: Each collection represents a snapshot of the last N seconds
3. **Reset Behavior**: Values can go up or down between collections based on actual traffic
4. **Pre-aggregated Data**: Elasticsearch provides aggregated counts, not raw events

**Example:**
```
# Traditional counter approach (typical exporter)
requests_total 1000  # Ever increasing
requests_total 1050  # +50 requests since last scrape

# Our gauge approach (aggregated from Elasticsearch)
delivery_requests_total{vhost="example.com"} 1523  # Requests in last 60s window
delivery_requests_total{vhost="example.com"} 1489  # Requests in next 60s window (can vary)
```

### Rate Metrics

Since our base metrics are gauges (not counters), we provide **pre-calculated rate metrics**:

```
delivery_requests_total = 1523          # Total requests in time window
delivery_requests_rate_total = 25.38    # Requests per second (1523 / 60)
```

**Using rate metrics in PromQL:**
```promql
# Use rate metrics directly (no rate() function needed)
delivery_requests_rate_total

# For base metrics, you can still use rate() but results may differ
rate(delivery_requests_total[5m])  # Rate of change of the gauge
```

> **Recommendation:** Use the `_rate_total` metrics for per-second rates, as they're accurately calculated from the source data.

---

## Common Label Definitions

| Label | Description | Example Values |
|-------|-------------|----------------|
| `company_id` | Unique company identifier | `123` |
| `company_name` | Company display name | `Example Corp` |
| `vhost` | Site host / domain | `example.com`, `cdn.example.com` |
| `code` | HTTP status code (dynamic) | `200`, `404`, `500` |
| `method` | HTTP method (dynamic) | `GET`, `POST`, `HEAD` |
| `protocol` | Protocol type | `http`, `https` |
| `version` | HTTP version | `HTTP/1.1`, `HTTP/2`, `HTTP/3` |
| `content_type` | Content type from HTTP header (dynamic) | `text/html`, `application/json`, `image/jpeg` |
| `cache_status` | Cache hit/miss status | `hit`, `miss`, `pass` |
| `action` | Security/mitigation action (dynamic) | varies by service |
| `reason` | Bot detection reason (dynamic) | varies by detection logic |
| `country_code` | ISO 3166-1 alpha-2 country code | `US`, `ES`, `FR`, `CN` |
| `le` | Histogram bucket (less-or-equal) | `0.1`, `0.5`, `1.0`, `+Inf` |

---

## Example PromQL Queries

### Delivery Performance

```promql
# Average response time across all companies
delivery_avg_response_time_seconds

# 95th percentile response time (using histogram)
histogram_quantile(0.95, sum by (company_name, le) (rate(delivery_response_time_seconds_bucket[1m])))

# 99th percentile response time
histogram_quantile(0.99, sum by (company_name, le) (rate(delivery_response_time_seconds_bucket[1m])))

# Total bandwidth per company (bytes/sec) - using rate metric
sum by (company_name) (delivery_bandwidth_bytes_rate_total)

# Requests per second by vhost - using rate metric
sum by (vhost) (delivery_requests_rate_total)

# Cache hit ratio (using cache_status label)
sum(delivery_cache_requests_total{cache_status="hit"}) / 
sum(delivery_requests_total) * 100
```

### Error Monitoring

```promql
# 5xx error rate (sum all 5xx status codes)
sum(delivery_status_code_rate_total{code=~"5.."})

# 4xx error rate
sum(delivery_status_code_rate_total{code=~"4.."})

# 404 errors by vhost
sum by (vhost) (delivery_status_code_total{code="404"})

# Error percentage (4xx + 5xx)
(sum(delivery_status_code_total{code=~"[45].."}) / sum(delivery_requests_total)) * 100

# Status code distribution
sum by (code) (delivery_status_code_total)
```

### Security Metrics

```promql
# WAF requests rate
sum(waf_requests_rate_total)

# DDoS mitigation by action type
sum by (action) (ddos_action_total)

# DDoS mitigated bandwidth
ddos_mitigated_bytes_rate_total

# Bot requests by action
sum by (action) (bot_requests_by_action_total)

# Bot requests by country (top 10)
topk(10, sum by (country_code) (bot_requests_by_country_total))

# Total security events rate
sum(waf_requests_rate_total) + sum(ddos_requests_rate_total) + sum(bot_requests_rate_total)
```

### Traffic Analysis

```promql
# HTTP/HTTPS distribution
sum by (protocol) (delivery_proto_total)

# HTTPS adoption rate
sum(delivery_proto_total{protocol="https"}) / 
sum(delivery_requests_total) * 100

# HTTP/2 vs HTTP/1.1 usage
sum by (version) (delivery_http_version_total)

# HTTP/2 adoption rate
sum(delivery_http_version_total{version="HTTP/2"}) / 
sum(delivery_requests_total) * 100

# Content type distribution
sum by (content_type) (
  delivery_content_type_html_total or
  delivery_content_type_json_total or
  delivery_content_type_image_total or
  delivery_content_type_video_total or
  delivery_content_type_css_total or
  delivery_content_type_javascript_total or
  delivery_content_type_other_total
)

# HTTP method distribution
sum by (method) (delivery_method_total)
```

### Business Metrics

```promql
# Top 10 companies by traffic (requests)
topk(10, sum by (company_name) (delivery_requests_total))

# Top 10 companies by bandwidth
topk(10, sum by (company_name) (delivery_bandwidth_bytes_total))

# Top 10 vhosts by traffic
topk(10, sum by (vhost) (delivery_requests_total))

# Average object size per vhost (bytes)
delivery_bandwidth_bytes_total / delivery_requests_total

# Object size distribution (percentage)
(delivery_object_size_1kb_total / delivery_requests_total) * 100
```

### Geographic Distribution

```promql
# WAF requests by country (top 20)
topk(20, sum by (country_code) (waf_requests_by_country_total))

# Bot traffic by country
topk(10, sum by (country_code) (bot_requests_by_country_total))
```

---

## Histogram Metrics Explained

### Response Time Histogram

We provide a histogram metric for delivery response times, allowing accurate percentile calculations.

**Metric Name:** `delivery_response_time_seconds`

**Buckets (seconds):**
- `le="0.05"` - ≤ 50ms
- `le="0.1"` - ≤ 100ms
- `le="0.25"` - ≤ 250ms
- `le="0.5"` - ≤ 500ms
- `le="0.75"` - ≤ 750ms
- `le="1.0"` - ≤ 1 second
- `le="2.5"` - ≤ 2.5 seconds
- `le="5.0"` - ≤ 5 seconds
- `le="7.5"` - ≤ 7.5 seconds
- `le="10.0"` - ≤ 10 seconds
- `le="+Inf"` - All requests

### Using Histograms for Percentiles

```promql
# Calculate any percentile (0.0 to 1.0)
histogram_quantile(0.50, sum by (le) (rate(delivery_response_time_seconds_bucket[1m])))  # p50 (median)
histogram_quantile(0.95, sum by (le) (rate(delivery_response_time_seconds_bucket[1m])))  # p95
histogram_quantile(0.99, sum by (le) (rate(delivery_response_time_seconds_bucket[1m])))  # p99

# Percentile by company
histogram_quantile(0.95, 
  sum by (company_name, le) (rate(delivery_response_time_seconds_bucket[5m]))
)

# Percentile by vhost
histogram_quantile(0.95, 
  sum by (vhost, le) (rate(delivery_response_time_seconds_bucket[5m]))
)
```

### Apdex Score Calculation

```promql
# Apdex with T=100ms (satisfied), 4T=400ms (tolerated)
(
  sum(rate(delivery_response_time_seconds_bucket{le="0.1"}[5m])) +
  sum(rate(delivery_response_time_seconds_bucket{le="0.4"}[5m])) / 2
) / sum(rate(delivery_response_time_seconds_count[5m]))
```

### Understanding Histogram Components

Each histogram metric generates three time series:

1. **`_bucket{le="X"}`** - Cumulative count of observations ≤ X
2. **`_sum`** - Sum of all observed values
3. **`_count`** - Total number of observations

```promql
# Average response time (alternative calculation)
rate(delivery_response_time_seconds_sum[5m]) / 
rate(delivery_response_time_seconds_count[5m])
```

**Important Notes:**
- The histogram is populated by observing individual bucket values from Elasticsearch
- Elasticsearch returns pre-aggregated histogram data (50ms buckets = 50,000 microseconds)
- Each observation from Elasticsearch is recorded `count` times in the Prometheus histogram
- This approach allows accurate percentile calculations despite aggregated source data

---

## Grafana Dashboards

A pre-built Grafana dashboard is available at:
[custom/grafana/provisioning/dashboards/](./custom/grafana/provisioning/dashboards/CDN_Metrics-Comprehensive_Dashboard-internally_cassic.json)

**Dashboard Includes:**
- Real-time request rates and bandwidth
- Response time percentiles (p50, p95, p99)
- Cache performance metrics
- HTTP status code distribution
- Security event monitoring (WAF/DDoS/Bot)
- Top companies/vhosts by traffic

**Import Instructions:**
1. Open Grafana → Dashboards → Import
2. Upload `CDN_Metrics-Comprehensive_Dashboard-internally_cassic.json`
3. Select your Prometheus data source
4. Click Import

---

## Troubleshooting

### No Data in Prometheus

**Check endpoint is accessible:**
```bash
curl https://api.transparentcdn.com/v1/statistics/metrics/<company_id>/prometheus
```

**Verify Prometheus configuration:**
- Check `prometheus.yml` has correct target
- Verify scrape_interval is set
- Check Prometheus logs for scrape errors

### Missing Metrics

If specific metrics are missing:

**Verify data exists:**
- Check if company has recent data in your [Dashboard Analytics](https://dashboard.transparentcdn.com/)
- Verify service is active for company
- Check time window is appropriate

### High Cardinality Warnings

If Prometheus shows cardinality warnings:

**Label cardinality is controlled** in our implementation:
- `company_id` / `company_name`: One per active company with metrics service enabled
- `vhost`: Limited to configured sites per company (can be high for large customers)
- `code`: HTTP status codes (dynamic, typically 10-20 different values)
- `method`: HTTP methods (typically 6-8 values: GET, POST, HEAD, PUT, DELETE, OPTIONS, etc.)
- `protocol`: 2 values (http, https)
- `version`: Typically 3-4 values (HTTP/1.1, HTTP/2, HTTP/3)
- `cache_status`: 3 values (hit, miss, pass)
- `content_type`: Dynamic based on actual content types (potentially high cardinality)
- `action`: Dynamic based on security actions (varies by service)
- `reason`: Dynamic based on bot detection reasons (potentially high cardinality)
- `country_code`: Up to ~250 country codes

**High cardinality sources:**
- **vhosts**: Companies with many domains can generate many time series
- **content_type**: Full content-type strings can create many unique values
- **action/reason**: Security and bot mitigation can have varied action types

**Mitigation strategies:**
1. Use metric relabeling in Prometheus to drop high-cardinality labels
2. Aggregate by higher-level labels (company, not vhost)
3. Use recording rules to pre-aggregate commonly used queries
4. Consider filtering less important label combinations

If cardinality becomes an operational issue, contact the development team for optimization options.

---

## Summary

### Key Points to Remember

1. **Per-Company Endpoint**: Each company has its own metrics endpoint at `/v1/statistics/metrics/<company_id>/prometheus`
2. **Authentication Required**: Use Bearer token authentication for all requests
3. **Service Requirement**: Companies must have `PrometheusMetrics` service enabled
4. **Gauge Metrics**: All metrics are Gauges (not Counters) representing time window snapshots
5. **Rate Metrics Included**: Pre-calculated `_rate_total` metrics for convenience
6. **Dynamic Labels**: Many label values (status codes, actions, etc.) are dynamic based on actual traffic
7. **Histogram Available**: Response time histogram for accurate percentile calculations

### Common Pitfalls

❌ **Don't** use `rate()` function on `_rate_total` metrics (they're already per-second)  
❌ **Don't** expect ever-increasing counters (these are gauge snapshots)  
❌ **Don't** forget to configure authentication in Prometheus  
✅ **Do** use histogram quantile functions for percentiles  
✅ **Do** filter by company_id when querying multi-company setups  
✅ **Do** monitor cache hits to understand actual metric freshness  
✅ **Do** use recording rules for complex queries
