# Observability Fundamentals

## Origem e Conceito

Observability (Observabilidade) é um conceito originário da teoria de controle que refere-se à capacidade de entender o estado interno de um sistema analisando apenas suas saídas.

No contexto de software moderno, observabilidade evoluiu para significar a **capacidade de medir e entender comportamento, performance e saúde de um sistema em produção**, em tempo real, sem necessidade de adicionar código específico.

**Definição:** Observability é a prática de instrumentar, coletar, correlacionar e analisar dados (métricas, logs, traces) para entender o que está acontecendo em um sistema em produção, permitindo diagnóstico rápido de problemas.

---

## Por Que Observabilidade Importa?

### O Problema Sem Observabilidade

```
Cenário: Sistema em produção fica lento

Usuários reclamam: "App está lento"

Sem observabilidade:
├─ Dev pergunta: "Como sabe que está lento?"
├─ User: "Demora mais tempo"
├─ Dev: "Qual página? Qual operação?"
├─ User: "Não sei, só está lento"
│
├─ Dev abre JIRA ticket
├─ Dev: "Preciso reproduzir"
├─ Passa horas debugando localmente
├─ Não consegue reproduzir (problem localizado)
├─ Ticket fica aberto
├─ Mais usuários reclamam
├─ Mais horas investigando
└─ 6+ horas depois descobre: DB query N+1

MTTR: 6+ horas 😞
```

### Com Observabilidade

```
Cenário: Sistema em produção fica lento

Alert: "P95 Latência acima de 5s"

Dashboard mostra:
├─ ✅ Request latency spike
├─ ✅ Query latency spike
├─ ✅ DB CPU 95%
├─ ✅ Specific query N+1 detected
└─ ✅ Timestamp exato

Dev ação:
├─ Log detalhado mostra query
├─ Trace mostra execução
├─ Métricas mostram timeline
├─ Fix: Adicionar index/cache
├─ Deploy solução
└─ Latência volta ao normal

MTTR: 15 minutos ✅
```

### Impacto em Números

```
Métrica                    Sem Observabilidade    Com Observabilidade    Melhoria
─────────────────────────────────────────────────────────────────────────────
MTTR (Mean Time to Repair)  6-8 horas             15-30 min              90% ↓
Bug detection (production)  Reactive              Proactive              -
False positive alerts       N/A                   < 5%                   -
Data-driven decisions       Nenhuma               Real-time              -
User experience issues      Descovered by users   Descovered by system   -
Team confidence            LOW                    HIGH                   -
```

---

## Monitoramento vs Observabilidade

### Monitoramento Tradicional

```
Monitoramento = Verificar métricas conhecidas

Exemplo:
├─ "CPU < 80%?"
├─ "Memory < 90%?"
├─ "Disk space OK?"
├─ "App respondendo?"

Características:
├─ Métricas pré-definidas
├─ Alertas baseados em thresholds
├─ Limitado a métrica conhecidas
├─ Não responde a problemas inesperados
└─ Reativo (espera problema aparecer)

Problema:
├─ CPU está 75% (OK para sistema)
├─ Mas latência explodiu (problema real!)
├─ Métrica pré-definida não captura
└─ Usuários descobrem primeiro que sistema
```

### Observabilidade Moderna

```
Observabilidade = Entender qualquer coisa sobre sistema

Exemplo:
├─ "Por que latência aumentou?"
├─ "Qual query está lento?"
├─ "De onde vêm as requisições lentas?"
├─ "Este user path está afetado?"

Características:
├─ Métricas do produto/negócio
├─ Correlação de dados
├─ Pode respondor perguntas não antecipadas
├─ Proativo e reativo
├─ Cultua de instrumentação

Vantagem:
├─ Dados correlacionados
├─ Raiz causa rápido
├─ Nenhuma pergunta inesperada
└─ Sistema alerta antes de usuário
```

---

## Os 3 Pilares da Observabilidade

### 1. Métricas (Metrics)

**O que:** Valores numéricos capturados ao longo do tempo.

**Exemplos:**
```yaml
Infraestrutura:
  ├─ CPU: 45%
  ├─ Memory: 62%
  ├─ Disk: 78%
  ├─ Network: 250 Mbps
  ├─ Connections DB: 45/100

Aplicação:
  ├─ Request count: 1000 req/s
  ├─ Error rate: 0.5%
  ├─ Latency (p99): 250ms
  ├─ Cache hit rate: 95%
  └─ Queue size: 150 messages

Negócio:
  ├─ Conversão: 2.5%
  ├─ Revenue: $10k/hr
  ├─ Active users: 50k
  └─ Sign-ups: 200/min
```

**Ferramentas:**
```
├─ Prometheus: Coleta e armazena
├─ Grafana: Dashboard de visualização
├─ Datadog: Cloud completo (tudo)
└─ New Relic: Observabilidade completa
```

**Uso:**
```
Alertas baseados em métricas:
├─ IF CPU > 90% → Alert
├─ IF Error rate > 5% → Alert
├─ IF Latency p99 > 1s → Alert
└─ IF Availability < 99% → Alert

Dashboards:
├─ System health
├─ Application performance
├─ Business metrics
└─ Capacity planning
```

---

### 2. Logs (Registros)

**O que:** Eventos textuais registrados durante execução.

**Exemplos:**
```
Timestamp: 2024-02-26 14:32:15.123
Level: ERROR
Service: Payment Service
Message: "Payment processing failed"
User ID: 12345
Transaction ID: txn_987654
Error: "Timeout calling external gateway after 30s"
Stack Trace: [stack trace]
Metadata:
  ├─ Amount: $99.99
  ├─ Currency: USD
  ├─ Retry count: 2
  ├─ Server: api-02.us-east-1
  └─ Request ID: req_abc123
```

**Ferramentas:**
```
├─ ELK (Elasticsearch, Logstash, Kibana): Open source
├─ Splunk: Enterprise
├─ Datadog: Cloud
├─ Sentry: Errors/exceptions
└─ CloudWatch: AWS native
```

**Uso:**
```
Análise de logs:
├─ Procurar mensagens específicas
├─ Correlacionar por request ID
├─ Timeline de eventos
├─ Root cause analysis

Exemplo query:
"message contains 'Payment' AND error AND timestamp > 14:30"
```

---

### 3. Traces (Rastreamento)

**O que:** Fluxo de uma requisição através de múltiplos serviços.

**Exemplo:**
```
Requisição: GET /api/checkout

Timeline:
├─ T+0ms: Client sends request
├─ T+5ms: API Gateway receives
│   ├─ T+6ms: Auth service (verify token) - 4ms
│   ├─ T+11ms: Cart service (calculate total) - 8ms
│   │   ├─ T+12ms: Inventory check - 2ms
│   │   └─ T+14ms: Price aggregation - 6ms
│   ├─ T+20ms: Payment service (process payment) - 15ms
│   │   ├─ T+21ms: Call gateway - 12ms
│   │   └─ T+33ms: Confirm transaction - 2ms
│   └─ T+36ms: Order service (create order) - 8ms
│       └─ T+37ms: DB write - 7ms
└─ T+45ms: Response sent

Visualização (como árvore):
```
GET /api/checkout (45ms)
├─ Auth Service (4ms) ✅
├─ Cart Service (8ms)
│  ├─ Inventory (2ms) ✅
│  └─ Price Agg (6ms) ✅
├─ Payment Service (15ms) ⚠️ LENTO
│  └─ Gateway Call (12ms) 🔴 CULPRIT
└─ Order Service (8ms)
   └─ DB Write (7ms) ✅
```

**Ferramentas:**
```
├─ Jaeger: Open source (CNCF)
├─ Zipkin: Open source
├─ Datadog APM: Cloud
├─ New Relic: Completo
└─ AWS X-Ray: AWS native
```

**Uso:**
```
Análise de performance:
├─ Identificar serviço lento
├─ Visualizar dependências
├─ Latência por serviço
├─ Falhas em qual ponto

Exemplo: P99 latência 1s
├─ Trace mostra: 800ms em Payment gateway
├─ Root cause identificado
├─ Fix: Aumentar timeout ou cache
```

---

## Onde Os 3 Pilares Conectam

```
Problema: "Checkout está lento e com erro"

Métrica alerta:
✅ Latency p99 went from 200ms to 1000ms
✅ Error rate from 0.1% to 5%

Log fornece contexto:
✅ Error: "External gateway timeout"
✅ Service: Payment

Trace mostra raiz causa:
✅ Request spend 800ms em payment-gateway
✅ 90% do tempo em chamada externa
✅ 2 tentativas de retry mostradas

Ação:
✅ Payment gateway estavaDown
✅ Configurar circuit breaker
✅ Add fallback strategy

Resultado:
✅ Latency back to 200ms
✅ Error rate 0%, com graceful degradation
```

---

## Observabilidade vs Testes

### Testes (Prevenção)

```
Objetivo: Evitar bugs antes de produção

Quando: Antes do deploy
Como: Unit, integration, E2E tests
Foco: Cenários conhecidos

Limitação: Não pode prever todos os problemas
```

### Observabilidade (Detecção e Resposta)

```
Objetivo: Entender e fixar problemas em produção

Quando: Em tempo de execução
Como: Métricas, logs, traces
Foco: Comportamento real

Diferença: Encontra problemas inesperados
```

### Complementares!

```
Testes: 95% de bugs não chegam produção ✅
Observabilidade: Aqueles 5% são detectados e fixados rápido ✅

Ambos necessários:
├─ Sem testes: Muitos bugs em produção
├─ Sem observabilidade: Bugs em produção levam horas para descobrir
└─ Com ambos: Confiança total
```

---

## Golden Signals (Sinais de Ouro)

Segundo Google Cloud, os 4 sinais principais de saúde:

### 1. Latência

**O que:** Tempo que leva para servir uma requisição.

```
Métricas importantes:
├─ P50: Mediana (50% das requisições)
├─ P95: 95º percentil (95% das requisições)
├─ P99: 99º percentil (pior 1%)
└─ Max: Pior caso

Exemplo:
P50: 100ms
P95: 250ms
P99: 500ms
Max: 2s

Significado:
├─ 50% dos usuários: < 100ms ✅
├─ 95% dos usuários: < 250ms ✅
├─ 99% dos usuários: < 500ms ⚠️
└─ Pior usuário: 2s ❌

Alerta:
IF P99 latency > 1s → Alert "Latency degradation"
```

### 2. Traffic

**O que:** Quantidade de requisições ao sistema.

```
Métricas:
├─ Requests per second (RPS)
├─ Bytes per second
├─ Concurrent connections
└─ Queue depth

Exemplo:
├─ Normal: 1000 RPS
├─ Peak hour: 5000 RPS
├─ Spike (viral): 20k RPS
└─ Capacity limit: 25k RPS

Alerta:
IF RPS > 24k → Alert "Near capacity"
```

### 3. Errors

**O que:** Taxa de requisições que falharam.

```
Métricas:
├─ Error rate: % de requisições com erro
├─ Error types: 5xx, timeouts, exceptions
├─ Error by service: Qual serviço falhou
└─ Error by endpoint: Qual endpoint problema

Exemplo:
├─ Normal: 0.01% (1 em 10,000)
├─ Degraded: 0.5% (50 em 10,000)
├─ Critical: 5%+ (500+ em 10,000)

Alerta:
IF error rate > 1% → Alert "Error rate high"
```

### 4. Saturation

**O que:** Como full os recursos estão.

```
Métricas:
├─ CPU: % de utilização
├─ Memory: % de utilização
├─ Disk: % de utilização
├─ DB connections: % usadas
├─ Thread pool: % saturado

Exemplo:
├─ Healthy: CPU 30%, Memory 50%
├─ Warning: CPU 70%, Memory 75%
├─ Critical: CPU 90%, Memory 90%
└─ Overload: Tudo 95%+

Alerta:
IF CPU > 80% for > 5min → Alert "High CPU"
```

---

## Referências com Outras Pastas

```
Observabilidade conecta com:

01 - Foundations:
└─ Shift-Left: Prevenir é melhor, mas observar quando escapar

02 - Unit Testing:
└─ Testes detecam cedo, observabilidade detecta em produção

04 - E2E Testing:
└─ E2E testa fluxo, tracing mostra execução real

06 - CI/CD & Automation:
├─ Feature flags: Observar para rollback
├─ Test Pipeline: Métricas de pipeline próprio
└─ Production deployment: POST-DEPLOY OBSERVABILITY

07 - Metrics & Maturity:
├─ DORA Metrics: Use observability para medir
├─ Quality Metrics: Cobertura + observability
└─ Maturity Model: Evolução depende de observability

08 - Culture & Organization:
├─ Ownership: Owner responsável por observar seu componente
└─ Quality Culture: Data-driven decisions
```

---

## Conclusão

**Observability é a base de um sistema de software confiável.**

3 Pilares funcionam juntos:
- ✅ **Métricas:** Verifique problemas
- ✅ **Logs:** Entenda contexto
- ✅ **Traces:** Encontre raiz causa

Golden Signals guiam:
- ✅ **Latency:** Performance
- ✅ **Traffic:** Carga
- ✅ **Errors:** Confiabilidade
- ✅ **Saturation:** Capacidade

Resultado:
- ✅ Problemas detectados cedo
- ✅ MTTR reduzido 90%
- ✅ Confiança em produção
- ✅ Data-driven decisions

Próximo passo: [Implementar observabilidade](metrics-logging-tracing.md)
