# OTEL-08: Troubleshooting OTel Pipelines

> **Series:** OTEL — OpenTelemetry Integration | **Notebook:** 8 of 8 | **Created:** January 2026 | **Last Updated:** 08/04/2026

## Debugging OpenTelemetry Data Flows
When telemetry data doesn't arrive as expected, systematic troubleshooting is essential. This notebook covers common issues, diagnostic techniques, and resolution steps for OpenTelemetry pipelines.

---

## Table of Contents

1. [Troubleshooting Methodology](#troubleshooting-methodology)
2. [Collector Issues](#collector-issues)
3. [SDK Issues](#sdk-issues)
4. [Connectivity Issues](#connectivity-issues)
5. [Data Quality Issues](#data-quality-issues)
6. [Performance Issues](#performance-issues)
7. [Diagnostic Tools](#diagnostic-tools)
8. [Common Fixes](#common-fixes)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Knowledge** | OTEL-01 through OTEL-07 |
| **Access** | Collector logs, application logs |

<a id="troubleshooting-methodology"></a>
## 1. Troubleshooting Methodology
### Systematic Approach

```
1. Identify the symptom
   ↓
2. Check component by component
   SDK → Collector → Backend
   ↓
3. Verify configuration
   ↓
4. Test connectivity
   ↓
5. Apply fix
   ↓
6. Verify resolution
```

### Symptom Categories

| Symptom | Likely Issue | Start Here |
|---------|--------------|------------|
| No data at all | Connectivity, auth | Section 4 |
| Partial data | Filtering, sampling | Section 5 |
| Delayed data | Performance | Section 6 |
| Wrong data | Configuration | Section 5 |
| Collector crashes | Resources | Section 6 |

<a id="collector-issues"></a>
## 2. Collector Issues
### Collector Won't Start

**Check configuration syntax:**
```bash
# Validate config
otelcol-contrib validate --config=config.yaml
```

**Common configuration errors:**

| Error | Cause | Fix |
|-------|-------|-----|
| `unknown type` | Invalid component name | Check spelling, verify contrib has component |
| `duplicate key` | YAML syntax error | Check indentation |
| `no pipelines defined` | Missing service section | Add service.pipelines |

### Enable Debug Logging

```yaml
service:
  telemetry:
    logs:
      level: debug
```

### Collector Dropping Data

**Check queue status:**
```bash
# If using zpages extension
curl http://localhost:55679/debug/tracez
```

**Common causes:**

| Cause | Indicator | Fix |
|-------|-----------|-----|
| Memory pressure | OOM, restarts | Increase memory_limiter limits |
| Export failure | Retry logs | Check backend connectivity |
| Queue full | Queue overflow logs | Increase queue size or consumers |

<a id="sdk-issues"></a>
## 3. SDK Issues
### No Spans Generated

**Python:**
```python
# Enable SDK debug logging
import logging
logging.basicConfig(level=logging.DEBUG)

# Verify tracer provider is set
from opentelemetry import trace
print(trace.get_tracer_provider())
# Should NOT be NoOpTracerProvider
```

**Java:**
```bash
# Enable agent debug
java -javaagent:opentelemetry-javaagent.jar \
  -Dotel.javaagent.debug=true \
  -jar app.jar
```

### Common SDK Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| NoOpTracer | Provider not set | Call set_tracer_provider() |
| No export | Processor not added | Add BatchSpanProcessor |
| Missing spans | Not in context | Use start_as_current_span |
| Lost context | Async not handled | Use context propagation |

### Async Context Loss

```python
# Wrong - loses context
async def bad_example():
    with tracer.start_as_current_span("parent"):
        await asyncio.gather(
            child_task(),  # No context
            child_task()
        )

# Right - preserves context
async def good_example():
    with tracer.start_as_current_span("parent"):
        token = context.attach(context.get_current())
        try:
            await asyncio.gather(
                child_task(),
                child_task()
            )
        finally:
            context.detach(token)
```

<a id="connectivity-issues"></a>
## 4. Connectivity Issues
### Test Collector Connectivity

```bash
# Test OTLP endpoint
curl -v http://collector:4318/v1/traces \
  -X POST \
  -H "Content-Type: application/json" \
  -d '{}'

# Test gRPC (with grpcurl)
grpcurl -plaintext collector:4317 list
```

### Test Dynatrace Connectivity

```bash
# Test OTLP endpoint (set DT_API_TOKEN env var first)
curl -v https://${DT_ENV_ID}.live.dynatrace.com/api/v2/otlp/v1/traces \
  -X POST \
  -H "Authorization: Api-Token ${DT_API_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{}'

# Expected: 200 OK (empty body is fine)
```

### Common Connectivity Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `connection refused` | Wrong port/host | Verify endpoint |
| `401 Unauthorized` | Invalid token | Check token, regenerate |
| `403 Forbidden` | Missing scope | Add required scope to token |
| `certificate error` | TLS issue | Check certs, disable verify for test |
| `timeout` | Network/firewall | Check firewall rules |

### Proxy Issues

```yaml
# Collector proxy configuration
exporters:
  otlphttp:
    endpoint: https://tenant.live.dynatrace.com/api/v2/otlp
    proxy_url: http://proxy.example.com:8080
```

```bash
# Or via environment
export HTTPS_PROXY=http://proxy.example.com:8080
```

<a id="data-quality-issues"></a>
## 5. Data Quality Issues
### Missing Attributes

**Debug exporter to see data:**
```yaml
exporters:
  debug:
    verbosity: detailed

service:
  pipelines:
    traces:
      exporters: [debug, otlphttp]
```

### service.name Missing

Without `service.name`, Dynatrace can't create service entities:

```yaml
# Add default service.name in collector
processors:
  resource:
    attributes:
      - key: service.name
        value: "unknown-service"
        action: insert  # Only adds if missing
```

### Wrong Time Zone

```yaml
# Ensure collector uses UTC
# In Kubernetes:
env:
  - name: TZ
    value: UTC
```

### Data Being Filtered

Check filter processor configuration:
```yaml
processors:
  filter:
    traces:
      span:
        # This filters OUT matching spans
        - 'attributes["http.url"] == "/health"'
```

<a id="performance-issues"></a>
## 6. Performance Issues
### Collector Memory Issues

**Symptoms:**
- OOMKilled restarts
- Slow processing
- Data drops

**Fixes:**
```yaml
processors:
  memory_limiter:
    check_interval: 1s
    limit_mib: 1500      # Increase limit
    spike_limit_mib: 500 # Allow spikes

  batch:
    timeout: 5s           # Faster flush
    send_batch_size: 500  # Smaller batches
```

### High Latency

**Causes and fixes:**

| Cause | Indicator | Fix |
|-------|-----------|-----|
| Large batches | Delayed export | Reduce batch size |
| Slow export | Backend latency | Increase parallelism |
| Too many processors | Processing time | Remove unnecessary processors |

```yaml
exporters:
  otlphttp:
    sending_queue:
      num_consumers: 10  # More parallel exports
```

### SDK Overhead

```python
# Use sampling to reduce volume
from opentelemetry.sdk.trace.sampling import TraceIdRatioBased

provider = TracerProvider(
    sampler=TraceIdRatioBased(0.1),  # 10% sampling
    resource=resource
)
```

<a id="diagnostic-tools"></a>
## 7. Diagnostic Tools
### Collector zpages

```yaml
extensions:
  zpages:
    endpoint: 0.0.0.0:55679

service:
  extensions: [zpages]
```

Access:
- `http://collector:55679/debug/servicez` - Service info
- `http://collector:55679/debug/pipelinez` - Pipeline status
- `http://collector:55679/debug/extensionz` - Extensions
- `http://collector:55679/debug/tracez` - Trace samples

### Health Check

```yaml
extensions:
  health_check:
    endpoint: 0.0.0.0:13133
```

```bash
curl http://collector:13133/
# {"status":"Server available"}
```

### pprof Profiling

```yaml
extensions:
  pprof:
    endpoint: 0.0.0.0:1777
```

```bash
go tool pprof http://collector:1777/debug/pprof/heap
```

```dql
// Check OTel data arrival
fetch spans, from: now() - 1h
| filter isNotNull(otel.library.name)
| summarize count = count(), by:{time_bucket = bin(timestamp, 5m)}
| sort time_bucket asc
```

```dql
// Find services without service.name
fetch spans, from: now() - 1h
| filter isNull(service.name) or service.name == ""
| summarize count = count(), by:{otel.library.name}
| sort count desc
```

<a id="common-fixes"></a>
## 8. Common Fixes
### Quick Reference

| Problem | Fix |
|---------|-----|
| No data at all | Check endpoint, auth token, network |
| No service entity | Add service.name resource attribute |
| Collector OOM | Increase memory_limiter, reduce batch size |
| High cardinality | Remove dynamic attributes |
| Context lost | Use context propagation in async |
| Wrong time | Set TZ=UTC |
| SSL errors | Check certs, add CA |

### Collector Config Template

A production-ready template:

```yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318

processors:
  memory_limiter:
    check_interval: 1s
    limit_mib: 1500
    spike_limit_mib: 500
  batch:
    timeout: 10s
    send_batch_size: 1000
  resource:
    attributes:
      - key: service.name
        value: unknown
        action: insert

exporters:
  otlphttp:
    endpoint: https://${DT_ENV}.live.dynatrace.com/api/v2/otlp
    headers:
      Authorization: Api-Token ${DT_TOKEN}
    retry_on_failure:
      enabled: true
      initial_interval: 5s
      max_interval: 30s

extensions:
  health_check:
  zpages:

service:
  extensions: [health_check, zpages]
  telemetry:
    logs:
      level: info
  pipelines:
    traces:
      receivers: [otlp]
      processors: [memory_limiter, resource, batch]
      exporters: [otlphttp]
    metrics:
      receivers: [otlp]
      processors: [memory_limiter, resource, batch]
      exporters: [otlphttp]
    logs:
      receivers: [otlp]
      processors: [memory_limiter, resource, batch]
      exporters: [otlphttp]
```

---

## Summary

In this notebook, you learned:

- Systematic troubleshooting methodology
- Collector debugging and configuration validation
- SDK issue diagnosis and async context handling
- Connectivity testing and common errors
- Data quality issue resolution
- Performance tuning for memory and latency
- Diagnostic tools: zpages, health check, pprof
- Quick fixes for common problems

---

## References

- [Collector Troubleshooting](https://opentelemetry.io/docs/collector/troubleshooting/)
- [Python instrumentation (opentelemetry.io)](https://opentelemetry.io/docs/languages/python/)
- [Dynatrace OTel Troubleshooting](https://docs.dynatrace.com/docs/ingest-from/opentelemetry/troubleshooting)

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
