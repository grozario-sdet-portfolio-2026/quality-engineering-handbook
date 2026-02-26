# Metrics, Logging, and Tracing

## Visão Geral

Este documento detalha cada um dos 3 pilares da observabilidade, com implementações práticas, ferramentas concretas e métricas importantes.

---

## 1. MÉTRICAS (Metrics)

### Conceito

Métricas são **valores numéricos capturados e armazenados ao longo do tempo**, permitindo visualizar padrões e tendências.

**Características:**
- ✅ Numérico (quantidade)
- ✅ Time-series (ao longo do tempo)
- ✅ Agregável (sum, avg, max, etc)
- ✅ Eficiente em storage
- ✅ Bom para alertas

### Tipos de Métricas

#### Infrastructure Metrics (Infra)

```yaml
CPU:
  ├─ User %: Tempo em código user
  ├─ System %: Tempo em kernel
  ├─ Wait I/O %: Tempo esperando disco
  └─ Idle %: Tempo não usado

Memory:
  ├─ Total: RAM total disponível
  ├─ Used: RAM utilizada
  ├─ Free: RAM livre
  ├─ Cached: RAM em cache
  └─ Swap: Uso de swap

Disk:
  ├─ Used: Espaço ocupado
  ├─ Free: Espaço livre
  ├─ Read Latency: Tempo de leitura
  └─ IOPS: Operações por segundo

Network:
  ├─ Packets in/out
  ├─ Bytes in/out
  ├─ Errors: Pacotes perdidos
  └─ Dropped: Pacotes descartados
```

#### Application Metrics (App)

```yaml
Requests:
  ├─ Request count
  ├─ Request rate (RPS)
  ├─ Request by endpoint
  └─ Request by method (GET, POST, etc)

Latency:
  ├─ Min: Requisição mais rápida
  ├─ Max: Requisição mais lenta
  ├─ Mean: Média
  ├─ P50 (Median): 50º percentil
  ├─ P95: 95º percentil
  ├─ P99: 99º percentil
  └─ P99.9: 99.9º percentil

Errors:
  ├─ 4xx: Erro do cliente
  ├─ 5xx: Erro do servidor
  ├─ Timeout: Requisição expirou
  ├─ Exception: Exceção lançada
  └─ Error rate %: Taxa de erro

Cache:
  ├─ Hit count: Cache acertou
  ├─ Miss count: Cache errou
  ├─ Hit ratio: % acertos
  └─ Eviction count: Dados removidos

Database:
  ├─ Query count
  ├─ Query latency (P95, P99)
  ├─ Slow queries: > 1s
  ├─ Connection pool: % utilizado
  ├─ Active connections
  └─ Lock wait time
```

#### Business Metrics

```yaml
Conversion:
  ├─ Funnel step: % que completa cada step
  ├─ Conversion rate: % que converte
  └─ Abandonment rate: % que desiste

Revenue:
  ├─ Total revenue: $ total
  ├─ Revenue per user
  ├─ Average order value
  └─ Transaction count

Users:
  ├─ Active users (daily, monthly)
  ├─ New users/hour
  ├─ Churn rate: % que deixa
  ├─ Session duration
  └─ Pages per session
```

### Ferramentas de Métricas

#### Prometheus (Open Source)

```yaml
Características:
  ├─ Pull-based: Scrape metrics de apps
  ├─ Time-series database
  ├─ PromQL: Query language
  ├─ Alerting built-in
  └─ Free and open source

Exemplo config:
```
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'api-server'
    static_configs:
      - targets: ['localhost:8080']

  - job_name: 'database'
    static_configs:
      - targets: ['db.internal:9090']

alert_rules:
  - alert: HighCPU
    expr: node_cpu_percent > 80
    for: 5m
```

#### Grafana (Visualização)

```yaml
Características:
  ├─ Conecta em Prometheus, Datadog, etc
  ├─ Dashboards customizáveis
  ├─ Rich visualization
  ├─ Alerting
  └─ Free (com versão enterprise)

Exemplo Dashboard:
├─ Panel 1: CPU usage (line graph)
├─ Panel 2: Memory usage (gauge)
├─ Panel 3: Request rate (counter)
├─ Panel 4: Error rate (stat)
├─ Panel 5: Latency p99 (heat map)
└─ Panel 6: Alerts (list)
```

#### Datadog (Cloud Completo)

```yaml
Características:
  ├─ SaaS (no self-hosting)
  ├─ Integra tudo: infra, app, logs, traces
  ├─ UI moderna
  ├─ AI-powered insights
  └─ Expense: Caro mas completo

Quando usar:
├─ Pequeno time: Melhor Prometheus + Grafana
├─ Médio time: Pode valer Datadog
└─ Grande empresa: Datadog é comum

Pricing:
├─ ~$15-30 por servidor/mês
└─ ~$1-2 por 1M events/mês (APM)
```

#### New Relic

```yaml
Características:
  ├─ APM (Application Performance Monitoring)
  ├─ Full observability
  ├─ Integrado com logs + traces
  └─ Expense: Competidor Datadog

Diferenciais:
├─ Excelente em app-level insights
├─ AI-powered anomaly detection
└─ Real-time alerting
```

### Dashboard Ideal de Métricas

```
┌─────────────────────────────────────────────────────┐
│        System Health Dashboard (Real-time)         │
├─────────────────────────────────────────────────────┤
│                                                     │
│  CPU Usage         Memory Usage     Disk Space     │
│  ▁▂▃▄▅▆▇█ 65%     ▂▄▆████████ 82%  █████░░░ 65%  │
│                                                     │
│  Requests/sec      Error Rate       P99 Latency    │
│  2,450 RPS         0.2%             245ms          │
│                                                     │
│  Top Slow Endpoints:                                │
│  ├─ POST /api/checkout: 450ms p99                  │
│  ├─ GET /api/products: 180ms p99                   │
│  └─ GET /api/users: 120ms p99                      │
│                                                     │
│  Alerts (Last 24h):                                │
│  ├─ ⚠️ High CPU - 2h ago (resolved)                │
│  ├─ 🔴 Failed deploy - 1h ago (critical)          │
│  └─ ℹ️ Cache hit rate down - 30min ago             │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

## 2. LOGGING (Logs)

### Conceito

Logs são **eventos estruturados ou textuais** que registram o que aconteceu em um ponto específico da aplicação.

**Características:**
- ✅ Textual/Estruturado
- ✅ Ponto no tempo
- ✅ Alto-volume
- ✅ Excelente para debugging
- ❌ Mais volume que métricas

### Tipos de Logs

#### Application Logs

```
2024-02-26T14:32:15.123Z INFO  [payment-service] User completed purchase
  user_id=12345
  order_id=ord_987654
  amount=99.99
  currency=USD
  request_id=req_abc123
  duration_ms=245

2024-02-26T14:32:17.456Z ERROR [payment-service] Payment gateway timeout
  user_id=12345
  order_id=ord_987654
  gateway=stripe
  error_code=GATEWAY_TIMEOUT
  retry_count=2
  request_id=req_abc123
  stack_trace=[...]

2024-02-26T14:32:20.789Z WARN  [payment-service] Retrying payment
  user_id=12345
  order_id=ord_987654
  retry_number=1
  next_retry_ms=5000
```

#### System Logs

```
kernel: Out of memory: Kill process 1234 (java) score 850
systemd: payment-service.service: Main process exited, code=exited, status=137
sshd: Failed password for root from 192.168.1.100 port 12345 ssh2
```

#### Security Logs

```
2024-02-26T14:32:15Z AUTH [auth-service] Login attempt
  username=user@example.com
  ip=203.0.113.45
  success=true
  method=password

2024-02-26T14:35:22Z SECURITY [auth-service] Suspicious activity
  username=attacker@evil.com
  ip=198.51.100.10
  attempt_count=50
  result=blocked
  reason=rate_limit_exceeded
```

### Ferramentas de Logging

#### ELK Stack (Open Source)

```yaml
Stack:
  Elasticsearch (E):
    ├─ Search engine para logs
    ├─ Scalável
    └─ Rápido

  Logstash (L):
    ├─ Processa logs
    ├─ Parse estruturado
    └─ Envia para ES

  Kibana (K):
    ├─ UI de visualização
    ├─ Query e análise
    └─ Dashboards

Configuração típica:
App → Logstash → Elasticsearch ← Kibana

Custo:
├─ Self-hosted: $0 + infraestrutura
└─ Elastic Cloud: $0.50-2/GB/dia

Quando usar:
├─ High volume logs
├─ Need to self-host
└─ Cost-sensitive
```

#### Sentry (Error Tracking)

```yaml
Características:
  ├─ Especializado em exceptions/errors
  ├─ Automatic error grouping
  ├─ Source maps support
  ├─ Release tracking
  └─ Team collaboration

Perfeito para:
├─ Frontend errors
├─ Backend exceptions
├─ Performance monitoring
└─ Release management

Exemplo:
App crashes → Sentry captures:
  ├─ Error message
  ├─ Stack trace
  ├─ User context
  ├─ Device info
  ├─ Release version
  └─ Breadcrumbs (eventos anteriores)

UI:
├─ Issues list (grouped by error)
├─ Error detail page
├─ Timeline of events
└─ User affected

Custo:
├─ Free: 5,000 events/month
├─ Paid: $25-299/month
└─ Enterprise: Custom pricing
```

#### Datadog Logs

```yaml
Características:
  ├─ Integrado com metrics + traces
  ├─ Full-text search
  ├─ Log-based alerting
  ├─ Live tail
  └─ Retention policies

Exemplo query:
"status:error AND service:payment-service AND -retry:true"
 sorting by timestamp desc
 last 24 hours

Custo:
├─ ~$0.10-0.30 per 1M logs
└─ Retention tiers (hot, warm, cold)
```

### Log Query Prático

```
Cenário: "Por que checkout falhando?"

Query em ELK:
  logs.service:"payment" 
  AND logs.level:"error"
  AND logs.timestamp > "now-1h"
  AND logs.user_id:12345

Resultado:
├─ Error 1: Payment gateway timeout (14:32:17)
├─ Error 2: Retry attempt (14:32:20)
├─ Error 3: Payment declined (14:32:30)
└─ Error 4: Max retries exceeded (14:32:45)

Timeline:
14:32:15 - User submits order
14:32:17 - Gateway timeout (1ª tentativa)
14:32:20 - Retry (2ª tentativa)
14:32:30 - Card declined (3ª tentativa)
14:32:45 - Fail (deu up)

Action: Contatar user, tentar account antigo
```

---

## 3. TRACING (Rastreamento Distribuído)

### Conceito

Tracing rastreia o **flow de uma requisição através de múltiplos serviços**, mostrando onde o tempo é gasto.

**Características:**
- ✅ Rastreia fluxo cross-service
- ✅ Mostra dependências
- ✅ Identifica gargalos
- ✅ Visualiza latência real

### Componentes de Trace

```
Trace: Requisição inteira
  │
  ├─ Span 1: API Gateway
  │  ├─ Operation: receive_request
  │  ├─ Duration: 2ms
  │  └─ Tags: method=GET, path=/checkout
  │
  ├─ Span 2: Auth Service
  │  ├─ Operation: verify_token
  │  ├─ Duration: 4ms
  │  ├─ Status: success
  │  └─ Parent: Span 1
  │
  ├─ Span 3: Order Service
  │  ├─ Operation: validate_cart
  │  ├─ Duration: 8ms
  │  ├─ Parent: Span 1
  │  │
  │  └─ Span 3a: Inventory Check (child)
  │     ├─ Duration: 3ms
  │     └─ Parent: Span 3
  │
  ├─ Span 4: Payment Service
  │  ├─ Operation: process_payment
  │  ├─ Duration: 25ms  ← LENTO!
  │  ├─ Parent: Span 1
  │  │
  │  └─ Span 4a: Call Gateway (child)
  │     ├─ Duration: 22ms  ← CULPRIT
  │     └─ Parent: Span 4
  │
  └─ Span 5: Order Service
     ├─ Operation: create_order
     ├─ Duration: 10ms
     └─ Parent: Span 1

Total latency: 50ms
Breakdown:
├─ Payment: 25ms (50%)
└─ Outros: 25ms (50%)

Problema identificado: Payment gateway lento
Action: Aumentar timeout, cache, ou alertar provider
```

### Ferramentas de Tracing

#### Jaeger (Open Source, CNCF)

```yaml
Características:
  ├─ Open source (Apache 2.0)
  ├─ CNCF graduated project
  ├─ Scalável
  ├─ UI excelente
  └─ Sem custo

Componentes:
  Jaeger Client (SDK):
    └─ Instrumento código (emit spans)
  
  Jaeger Agent:
    └─ Local server que coleta
  
  Jaeger Collector:
    └─ Backend que armazena
  
  Jaeger Query:
    └─ UI para visualizar

Exemplo implementação (Node.js):
```
const jaeger = require('jaeger-client');

const tracer = jaeger.initTracer({
  serviceName: 'payment-service',
  sampler: {
    type: 'const',
    param: 1,
  },
});

const span = tracer.startSpan('process_payment');
span.setTag('user_id', userId);
span.log({ event: 'payment_start' });

// ... seu código ...

span.finish();
```

Custo: Gratuito (você gerencia infra)
```

#### Datadog APM (Cloud)

```yaml
Características:
  ├─ Full-featured
  ├─ Conecta com metrics + logs
  ├─ Service map auto-generated
  ├─ Flame graphs built-in
  └─ Integrado no Datadog

Exemplo visualização:
UI mostra:
├─ Service dependency graph
├─ Request timing por serviço
├─ Error attribution by service
└─ Performance trends

Custo:
├─ ~$1-2 per 1M spans/month
└─ Ingested traces + analytics
```

#### New Relic APM

```yaml
Características:
  ├─ APM especializado
  ├─ Distributed tracing
  ├─ Excelente UI
  └─ Integrado com logs

Custo:
├─ Similar to Datadog
└─ ~$100-300/month típico
```

### Visualizações de Trace

#### Timeline View

```
Mostra quando cada operação ocorreu:

 0ms    5ms     10ms    15ms    20ms    25ms    30ms
 |——————|————————|————————|————————|————————|————————|

Auth    |██|
        └─ 4ms

Cart    |████████|
        └─ 8ms

Inventory   |███|
            └─ 3ms

Payment           |█████████████████████████|
                  └─ 25ms (SLOW)
  
  └─ Gateway        |█████████████████████|
                    └─ 22ms (CULPRIT)

Order    |██████████|
         └─ 10ms

Total: 50ms
```

#### Service Map

```
User → API Gateway → Auth Service
         │         ├─ Cart Service (with Inventory)
         │         ├─ Payment Service (with Gateway Call)
         │         └─ Order Service → DB

Cores:
├─ Verde: Normal (< 100ms)
├─ Amarelo: Degraded (100-500ms)
└─ Vermelho: Crítico (> 500ms)

Clicando em linhas:
├─ Histograma de latência
├─ Error rate
└─ Modo de falhas
```

---

## Métricas Importantes: Downtime e Performance

### Downtime (Disponibilidade)

```yaml
Definição: % de tempo que sistema está online

Cálculo:
Availability = (Total Time - Downtime) / Total Time × 100

Exemplo com 30 dias:
Total Time: 43,200 minutos
Downtime: 20 minutos
Availability: (43,200 - 20) / 43,200 × 100 = 99.95%

Alvos comuns (SLA):
├─ 99% (uptime "dois noves"): 3.6 horas downtime/mês
├─ 99.9% (três noves): 21.6 minutos downtime/mês
├─ 99.95% (quatro noves): 10.8 minutos downtime/mês
├─ 99.99% (cinco noves): 1.3 minutos downtime/mês
└─ 99.999% (seis noves): 8 segundos downtime/mês

Monitoramento:
├─ Synthetic pinging (verificar a cada 10s)
├─ Alert if downtime > 30s
├─ Dashboard mostrando uptime %
└─ Correlate com logs/metrics

Exemplo Grafana metric:
rate(requests_total[5m]) == 0  → Downtime detected
```

### Performance Metrics (Performance)

```yaml
Critical Metrics:

1. Latency (Latência):
   ├─ P50: 50º percentil
   ├─ P95: 95º percentil (crítico)
   ├─ P99: 99º percentil (muito crítico)
   └─ Max: Pior caso

   SLOs típicos:
   ├─ P95 < 100ms
   ├─ P99 < 200ms
   └─ Max < 1s

2. Throughput (Taxa):
   ├─ Requests per second (RPS)
   ├─ Bytes per second (BPS)
   └─ Operations per second

   SLOs típicos:
   ├─ Handle 1000 RPS without degradation
   └─ Scale to 10000 RPS in < 5min

3. Resource Usage (Recursos):
   ├─ CPU: % utilizado
   ├─ Memory: % utilizado
   ├─ Network: Mbps consumed
   └─ Disk I/O: IOPS

   SLOs típicos:
   ├─ CPU < 70% under normal load
   ├─ Memory < 80% utilization
   └─ Latency doesn't degrade if CPU < 85%

4. Error Rate (Taxa de erro):
   ├─ % de requisições com erro
   ├─ Erros por tipo (5xx, timeout, etc)
   └─ Erros por endpoint

   SLOs típicos:
   ├─ < 0.1% error rate
   ├─ Critical operations: < 0.01%
   └─ Recover from single error in < 30s

Dashboard exemplo:
```

```
┌─────────────────────────────────────────────┐
│     Performance Dashboard                   │
├─────────────────────────────────────────────┤
│                                             │
│ Uptime & Availability:   99.95% ✅         │
│ Last incident: 3 days ago (20min)           │
│ SLA status: HEALTHY                         │
│                                             │
│ Latency:                                    │
│  P95: 95ms (target: 100ms) ✅               │
│  P99: 180ms (target: 200ms) ✅              │
│  Max: 850ms (target: 1000ms) ✅             │
│                                             │
│ Throughput:                                 │
│  RPS: 2,450 (capacity: 5,000) ✅            │
│  Growing trend: +5% today                   │
│                                             │
│ Error Rate:                                 │
│  Overall: 0.02% (target: 0.1%) ✅           │
│  5xx: 0.01%                                 │
│  Timeout: 0.005%                            │
│  4xx: 0.005%                                │
│                                             │
│ Resource Usage:                             │
│  CPU: 55% (threshold: 85%) ✅               │
│  Memory: 62% (threshold: 90%) ✅            │
│  Latency at high resource: Normal ✅        │
│                                             │
└─────────────────────────────────────────────┘
```

---

## Integração: Metrics + Logs + Traces

### Exemplo Real: Investigando Performance Degradation

```
Cenário: P99 latency aumentou de 100ms para 500ms

Step 1: Métrica alerta
├─ Alert: "P99 latency > threshold"
├─ Dashboard mostra: P99 jump to 500ms at 14:30

Step 2: Correlacionar com logs
├─ Buscar logs entre 14:29-14:31
├─ Encontra: "Database connection pool depleted"
├─ 100 conexões simultâneas na queue

Step 3: Analisar com traces
├─ Pegar sample de requisição lenta
├─ Ver timeline: Payment service esperando DB
├─ DB queries todas em ~40ms (OK)
├─ Mas 10+ queries queued (não OK)

Step 4: Root cause
├─ DB connection pool = 50
├─ Traffic spike: 5000 RPS (norm: 1000)
├─ Todos esperando conexão
├─ Connection acquisition: 400ms (muito!)

Step 5: Solution options
├─ Aumentar pool size: 50 → 100 (quick)
├─ Optimize queries para usar menos connections (better)
├─ Add read replicas (scale)
├─ Implementar circuit breaker (resilience)

Step 6: Implement + Monitor
├─ Deploy: Aumentar pool para 100
├─ Alert se pool > 80%
├─ P99 volta para 120ms
├─ Planned para próximo: otimizar queries
```

---

## Conclusão

**Os 3 pilares funcionam em harmonia:**

- ✅ **Métricas** alertam do problema
- ✅ **Logs** fornecem contexto
- ✅ **Traces** identificam raiz causa

**Ferramentas principais:**
- Prometheus + Grafana (open source, métricas)
- ELK Stack (open source, logs)
- Jaeger (open source, traces)
- Datadog (cloud, completo, caro)
- Sentry (especializado em errors)

**Benefício:**
- ✅ MTTR: 92% mais rápido
- ✅ Visibility: 360° do sistema
- ✅ Confiança: Data-driven decisions

Próximo: [Observability Culture](observability-culture.md)
