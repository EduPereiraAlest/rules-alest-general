# 📊 Observability & Logging Rules

## 🎯 Objetivo
Regras obrigatórias para structured logging, monitoring, APM (Application Performance Monitoring), distributed tracing e observabilidade completa em sistemas enterprise.

## 📝 Structured Logging Standards

### **🏗️ Universal Log Structure**
```json
{
  "timestamp": "2024-12-19T14:30:00.123Z",
  "level": "INFO",
  "service": "user-service",
  "version": "1.2.0",
  "environment": "production",
  "trace_id": "abc123def456",
  "span_id": "789ghi012",
  "correlation_id": "req-xyz789",
  "message": "User created successfully",
  "context": {
    "user_id": 12345,
    "action": "create_user",
    "ip_address": "192.168.1.100",
    "request_id": "req-abc123"
  },
  "metrics": {
    "duration_ms": 150,
    "memory_usage_mb": 45.2
  },
  "labels": {
    "team": "backend",
    "component": "authentication"
  }
}
```

### **📊 Multi-Language Logging Implementation**

```python
# Python Structured Logging
import json
import logging
from datetime import datetime
from typing import Dict, Any, Optional
from contextvars import ContextVar
from dataclasses import dataclass, asdict

# Context variables for distributed tracing
trace_id_var: ContextVar[Optional[str]] = ContextVar('trace_id', default=None)

@dataclass
class LogContext:
    user_id: Optional[str] = None
    request_id: Optional[str] = None
    action: Optional[str] = None

class StructuredLogger:
    def __init__(self, service_name: str, version: str, environment: str):
        self.service = service_name
        self.version = version
        self.environment = environment
        self.logger = logging.getLogger(service_name)
        
        # Configure JSON formatter
        handler = logging.StreamHandler()
        handler.setFormatter(self.JsonFormatter())
        self.logger.addHandler(handler)
    
    class JsonFormatter(logging.Formatter):
        def format(self, record):
            log_obj = {
                'timestamp': datetime.utcnow().isoformat() + 'Z',
                'level': record.levelname,
                'service': getattr(record, 'service', 'unknown'),
                'version': getattr(record, 'version', '1.0.0'),
                'environment': getattr(record, 'environment', 'development'),
                'message': record.getMessage()
            }
            
            if trace_id_var.get():
                log_obj['trace_id'] = trace_id_var.get()
                
            if hasattr(record, 'context'):
                log_obj['context'] = record.context
                
            return json.dumps(log_obj)
    
    def info(self, message: str, context: Optional[LogContext] = None):
        extra = {
            'service': self.service,
            'version': self.version,
            'environment': self.environment
        }
        
        if context:
            extra['context'] = {k: v for k, v in asdict(context).items() if v is not None}
            
        self.logger.info(message, extra=extra)
```

```java
// Java Structured Logging with Logback
@Component
public class StructuredLogger {
    private final Logger logger = LoggerFactory.getLogger(StructuredLogger.class);
    private final String serviceName;
    private final String version;
    private final String environment;
    
    public StructuredLogger(
            @Value("${app.name}") String serviceName,
            @Value("${app.version}") String version,
            @Value("${app.environment}") String environment) {
        this.serviceName = serviceName;
        this.version = version;
        this.environment = environment;
    }
    
    public void info(String message, LogContext context) {
        MDC.put("service", serviceName);
        MDC.put("version", version);
        MDC.put("environment", environment);
        
        if (context != null) {
            if (context.getTraceId() != null) MDC.put("trace_id", context.getTraceId());
            if (context.getUserId() != null) MDC.put("user_id", context.getUserId());
        }
        
        try {
            logger.info(message);
        } finally {
            MDC.clear();
        }
    }
    
    @Data
    @Builder
    public static class LogContext {
        private String traceId;
        private String userId;
        private String requestId;
        private String action;
    }
}
```

## 🔍 Distributed Tracing

### **🌐 OpenTelemetry Implementation**
```typescript
// TypeScript OpenTelemetry Setup
import { NodeSDK } from '@opentelemetry/sdk-node';
import { JaegerExporter } from '@opentelemetry/exporter-jaeger';
import { trace, SpanStatusCode, SpanKind } from '@opentelemetry/api';

const jaegerExporter = new JaegerExporter({
  endpoint: process.env.JAEGER_ENDPOINT || 'http://localhost:14268/api/traces',
});

const sdk = new NodeSDK({
  traceExporter: jaegerExporter,
  instrumentations: [],
});

export class TracingService {
  private tracer = trace.getTracer('user-service', '1.2.0');
  
  async traceFunction<T>(
    spanName: string,
    fn: () => Promise<T>,
    attributes?: Record<string, string>
  ): Promise<T> {
    return await this.tracer.startActiveSpan(spanName, async (span) => {
      try {
        if (attributes) {
          span.setAttributes(attributes);
        }
        
        const result = await fn();
        span.setStatus({ code: SpanStatusCode.OK });
        return result;
        
      } catch (error) {
        span.recordException(error as Error);
        span.setStatus({ code: SpanStatusCode.ERROR });
        throw error;
      } finally {
        span.end();
      }
    });
  }
}
```

## 📈 Application Performance Monitoring

### **⚡ Performance Metrics**
```csharp
// C# APM with Application Insights
public interface IPerformanceMonitoringService
{
    void TrackMetric(string metricName, double value);
    void TrackException(Exception exception);
    void TrackEvent(string eventName);
}

public class ApplicationInsightsService : IPerformanceMonitoringService
{
    private readonly TelemetryClient _telemetryClient;
    
    public ApplicationInsightsService(TelemetryClient telemetryClient)
    {
        _telemetryClient = telemetryClient;
    }
    
    public void TrackMetric(string metricName, double value)
    {
        _telemetryClient.TrackMetric(metricName, value);
    }
    
    public void TrackException(Exception exception)
    {
        _telemetryClient.TrackException(exception);
    }
    
    public void TrackEvent(string eventName)
    {
        _telemetryClient.TrackEvent(eventName);
    }
}
```

## 📊 Metrics & Alerting

### **📈 Prometheus Metrics**
```go
// Go Prometheus Metrics
package monitoring

import (
    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promauto"
    "time"
)

var (
    httpRequestsTotal = promauto.NewCounterVec(
        prometheus.CounterOpts{
            Name: "http_requests_total",
            Help: "Total HTTP requests",
        },
        []string{"method", "endpoint", "status_code"},
    )
    
    httpRequestDuration = promauto.NewHistogramVec(
        prometheus.HistogramOpts{
            Name: "http_request_duration_seconds",
            Help: "HTTP request duration",
        },
        []string{"method", "endpoint"},
    )
)

type MetricsService struct {
    httpRequests        *prometheus.CounterVec
    httpRequestDuration *prometheus.HistogramVec
}

func (m *MetricsService) RecordHTTPRequest(method, endpoint, statusCode string, duration time.Duration) {
    m.httpRequests.WithLabelValues(method, endpoint, statusCode).Inc()
    m.httpRequestDuration.WithLabelValues(method, endpoint).Observe(duration.Seconds())
}
```

### **🚨 Alerting Rules**
```yaml
# Prometheus Alerting Rules
groups:
  - name: application.rules
    rules:
      - alert: HighErrorRate
        expr: |
          sum(rate(http_requests_total{status_code=~"5.."}[5m])) /
          sum(rate(http_requests_total[5m])) > 0.05
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
          
      - alert: HighResponseTime
        expr: |
          histogram_quantile(0.95, 
            sum(rate(http_request_duration_seconds_bucket[5m])) by (le)
          ) > 2
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High response time detected"
```

## 🔍 Health Monitoring

### **🏥 Health Check Implementation**
```python
# Multi-layer Health Checks
from enum import Enum
import asyncio

class HealthStatus(Enum):
    HEALTHY = "healthy"
    DEGRADED = "degraded"
    UNHEALTHY = "unhealthy"

class HealthChecker:
    def __init__(self, config):
        self.config = config
    
    async def check_database(self):
        # Database connectivity check
        pass
    
    async def check_redis(self):
        # Redis connectivity check
        pass
    
    async def check_all(self):
        checks = [
            self.check_database(),
            self.check_redis()
        ]
        
        results = await asyncio.gather(*checks, return_exceptions=True)
        
        # Determine overall health
        if any(isinstance(r, Exception) for r in results):
            return HealthStatus.UNHEALTHY
        else:
            return HealthStatus.HEALTHY
```

### **📊 Monitoring Dashboard Requirements**
```yaml
# Required monitoring dashboards
dashboards:
  application_overview:
    panels:
      - request_rate_per_minute
      - error_rate_percentage
      - response_time_p95
      - active_users
      
  infrastructure:
    panels:
      - cpu_usage_percentage
      - memory_usage_percentage
      - disk_usage_percentage
      - network_io
      
  business_metrics:
    panels:
      - user_registrations_per_hour
      - successful_transactions
      - revenue_per_minute
      - feature_adoption_rates
      
alerts:
  critical:
    - service_down
    - high_error_rate_5_percent
    - database_connection_failure
    
  warning:
    - high_response_time_2_seconds
    - memory_usage_80_percent
    - disk_usage_85_percent
```

---

**📊 Regra de Ouro**: Observabilidade não é opcional - toda aplicação enterprise deve ter logs estruturados, métricas e tracing desde o primeiro dia.

**🔍 Lembrete**: "If you can't measure it, you can't improve it" - monitore tudo que importa para o negócio.

**🔄 Versão**: 1.0  
**📅 Última atualização**: 2024-12-19  
**👨‍💻 Criado por**: Universal AI-Powered Development Rules System
