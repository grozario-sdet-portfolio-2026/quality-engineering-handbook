# DORA Metrics

## Origem e Conceito

DORA Metrics (DevOps Research and Assessment) são quatro indicadores de performance de delivery identificados por uma pesquisa de 6 anos do Google Cloud DevOps Research Team.

Originada em 2014, a pesquisa DORA analisou dezenas de milhares de empresas e identificou que apenas **4 métricas** correlacionam-se fortemente com sucesso organizacional de software.

**Definição:** DORA Metrics são 4 indicadores quantificáveis que medem a velocidade, eficiência e confiabilidade de um pipeline de software:

1. **Deployment Frequency** - Com que frequência código vai para produção
2. **Lead Time for Changes** - Tempo de desenvolvimento até produção
3. **Mean Time to Recovery** - Tempo para recuperar de incidente
4. **Change Failure Rate** - Porcentagem de alterações que causam problemas

---

## Por Que DORA Metrics?

### Problemas que resolvem:

1. **Métricas Vagas** - "Estamos indo bem?" sem dados objetivos
2. **Foco em Atividades** - Medir trabalho (hours), não impacto
3. **Conflito Teams** - Dev quer velocidade, Ops quer estabilidade
4. **Falta de Direção** - Sem saber em que melhorar
5. **Burnout Silencioso** - Sem métricas, trabalho aumenta indefinidamente
6. **Decisões Subjetivas** - Baseadas em opinions, não dados
7. **Invisibilidade** - Sem um dashboard, ninguém sabe o status

### Benefícios:

- ✅ Métricas objetivas e mensuráveis
- ✅ Correlação com sucesso real (não vanity metrics)
- ✅ Identifica gargalos claramente
- ✅ Alinha dev e ops em objetivos comuns
- ✅ Orientação clara para melhorias
- ✅ Visibilidade executiva

### Impacto em Números

```
Organizações de Alto Desempenho vs Baixo Desempenho:

Métrica                    Baixo Desempenho    Alto Desempenho    Diferença
──────────────────────────────────────────────────────────────────────────
Deployment Frequency       1 vez/mês           500x/dia            15,000x ↑
Lead Time                  6 meses             1 hora              5,300x ↓
Mean Time to Recovery      24 horas            1 hora              24x ↓
Change Failure Rate        45%                 15%                 66% ↓
```

---

## 1. Deployment Frequency

**O que mede:** Com que frequência código é deployado em produção.

**Definição:** Número de vezes que uma alteração é deployada em produção, normalmente medido por dia.

### Categorias de Desempenho

```
ELITE:        > 1 deploy por dia
HIGH:         1 deploy por semana a 1 deploy por mês
MEDIUM:       1 deploy por mês a 1 deploy a cada 3 meses
LOW:          < 1 deploy a cada 3 meses (ou nem sabe medir)
```

### Exemplos Reais

```
Exemplo 1: Google, Amazon, Netflix
├─ Deployment Frequency: 100-500+ deploys/dia
├─ Como: Feature flags, automação, confiança alta
└─ Resultado: Rápido feedback, iteração contínua

Exemplo 2: Banco Tradicional
├─ Deployment Frequency: 1 deploy/ano
├─ Como: Mudanças grandes, processo manual, risco alto
└─ Resultado: Lento feedback, mudanças grandes

Exemplo 3: SaaS em crescimento
├─ Deployment Frequency: 5-10 deploys/dia
├─ Como: CI/CD, feature flags, automação crescente
└─ Resultado: Equilíbrio velocidade/estabilidade
```

### Como Melhorar

```
Nível 1 → Nível 2 (De 1/ano para 1/trimestre):
├─ ✅ Implementar CI/CD básico
├─ ✅ Automação de testes
├─ ✅ Documentação de deploy
└─ Lead time: 6 meses

Nível 2 → Nível 3 (De 1/trimestre para 1/semana):
├─ ✅ Pipeline completo (linting, testes, análise)
├─ ✅ Feature flags
├─ ✅ Blue-green deployment
└─ Lead time: 2 meses

Nível 3 → Nível 4 (De 1/semana para 1/dia+):
├─ ✅ Test Pipelines robusto (ref: 06 - CI/CD)
├─ ✅ Pull Request Testing (ref: 06 - CI/CD)
├─ ✅ Observabilidade em produção
├─ ✅ Confiança em testes (cobertura 80%+)
└─ Lead time: 2 semanas
```

### Cálculo

```
Deployment Frequency = Total Deploys em Produção / Período
                     = 50 deploys / 30 dias
                     = 1.67 deploys por dia

Rastreamento:
├─ Manual: Contar tags de release
├─ Automático: GitHub Actions, GitLab CI logs
├─ Dashboard: Grafana/Datadog
```

---

## 2. Lead Time for Changes

**O que mede:** Tempo do commit até código estar em produção.

**Definição:** Tempo decorrido desde que um commit é feito até estar rodando em produção.

### Categorias de Desempenho

```
ELITE:        < 1 hora
HIGH:         1 dia
MEDIUM:       1 semana
LOW:          > 1 mês
```

### Breakdown de Lead Time

```
Ciclo total de desenvolvimento:

Dev escreve código ────────────────────┐
                                       ▼
                            [PLANNING - opcional]
                            Backlog refinement
                            Estimativa
                            Arquitetura
                            ├─ média: 2 dias
                            │
                            ▼
Dev faz commit ─────────────┐
                            ▼
                   [DEVELOPMENT TIME]
                   ├─ Tempo local: 4 horas
                   │
                   ▼
PR criada ──────────────────┐
                            ▼
                    [REVIEW TIME]
                   ├─ Review: 30 min
                   │
                    ▼
Merge aprovado ─────────────┐
                            ▼
                   [PIPELINE TIME]
                   ├─ Lint: 5s
                   ├─ Tests: 30s
                   ├─ Build: 1m
                   ├─ Deploy: 2m
                   │ Total: ~4 min
                   │
                    ▼
Em Produção

LEAD TIME PARA CHANGES = Development + Review + Pipeline
                       = 4h + 30m + 4m
                       ≈ 4.5 horas
```

### Onde o Tempo se Perde

```
Análise de 1000 commits em empresa típica:

Planning:           30% (refinamento lento)
Development:        25% (complexidade, review loops)
Review:             15% (espera por reviewer)
Pipeline:           3%  (muito rápido normalmente)
Approval Process:   20% (gestão de risco)
Deploy/Release:     7%  (coordenação)

Insights:
├─ Maior desperdício: Planning (refinar melhor?)
├─ Segundo: Approval (é necessário?)
├─ Pipeline é rápido (CI/CD está OK)
└─ Review esperado (normal em 1-3 dias)
```

### Como Melhorar

```
Reduzir Planning (30% → 15%):
├─ ✅ Histórias menores (ref: Definition of Done)
├─ ✅ Critérios de aceite claros
├─ ✅ Refinamento contínuo
└─ Ganho: 1 dia

Reduzir Approval (20% → 5%):
├─ ✅ Quality gates automáticos (ref: 05 - Quality Gates)
├─ ✅ Confiança em testes
├─ ✅ Feature flags para risco baixo
└─ Ganho: 2.25 dias

Reduzir Review (15% → 5%):
├─ ✅ Reviews rápidas (< 4h)
├─ ✅ Bot assistente
├─ ✅ Pull Request Testing (ref: 06 - CI/CD)
└─ Ganho: 1 dia

Total: De 1 semana para 2-3 dias
```

### Cálculo

```
Lead Time = Data Deploy - Data Commit

Exemplo:
Commit: 2024-02-20 14:00
Deploy: 2024-02-21 10:30
Lead Time = 20.5 horas

Rastreamento:
├─ Manual: Git timestamps + deploy logs
├─ Automático: GitHub GraphQL API
├─ Dashboard: Paginate GitOps
```

---

## 3. Mean Time to Recovery (MTTR)

**O que mede:** Tempo para recuperar de incidente em produção.

**Definição:** Tempo decorrido desde que um problema em produção é detectado até estar resolvido.

### Categorias de Desempenho

```
ELITE:        < 1 hora
HIGH:         1 a 5 horas
MEDIUM:       5 a 24 horas
LOW:          > 24 horas
```

### Cenários de Incidente

```
Cenário 1: Bug pequeno em produção
├─ Detecção: 5 min (alertas)
├─ Root cause: 15 min (logs)
├─ Fix: 10 min (rollback/patch)
├─ Test: 5 min
├─ Deploy: 2 min
└─ Total MTTR: ~40 minutos ✅ ELITE

Cenário 2: Problema de performance
├─ Detecção: 30 min (usuário avisa)
├─ Root cause: 2 horas (análise profunda)
├─ Fix: 1 hora (otimização)
├─ Test: 30 min
├─ Deploy: 10 min
└─ Total MTTR: ~4 horas ✓ HIGH

Cenário 3: Falha de infraestrutura (sem automação)
├─ Detecção: 15 min
├─ Root cause: 3 horas (manual)
├─ Fix: 2 horas (manual)
├─ Test: 1 hora
├─ Deploy: 30 min
└─ Total MTTR: ~7 horas ⚠️ MEDIUM
```

### Fatores que Melhoram MTTR

```
1. Observabilidade (60% de impacto)
   ├─ Logs centralizados
   ├─ Métricas em tempo real
   ├─ Alertas inteligentes
   └─ Dashboards de saúde

2. Automação (50% de impacto)
   ├─ Health checks
   ├─ Auto rollback
   ├─ Provisioning automático
   └─ Feature flag rollback

3. Confiança em Testes (40% de impacto)
   ├─ Cobertura adequada (ref: 02 - Test Coverage)
   ├─ E2E smoke tests (ref: 04 - E2E Strategy)
   ├─ Runbooks testados
   └─ Procedures conhecidas

4. Confiança em Deploy (30% de impacto)
   ├─ Feature flags (ref: 06 - Feature Flags)
   ├─ Blue-green deployment
   ├─ Canary releases
   └─ Rollback simples
```

### Como Melhorar

```
Baseline: 6 horas

Semana 1-2:
├─ ✅ Implementar alertas básicos
├─ ✅ Centralizar logs
└─ Resultado: 5 horas (-17%)

Semana 3-4:
├─ ✅ Feature flags para rollback rápido
├─ ✅ Dashboard de saúde
└─ Resultado: 3 horas (-40%)

Semana 5-8:
├─ ✅ Health checks automáticos
├─ ✅ Auto-remediation básica
└─ Resultado: 1.5 horas (-50%)

Meta Elite (< 1h):
├─ ✅ Observabilidade completa
├─ ✅ Automação avançada
├─ ✅ Runbooks otimizados
└─ Resultado: 45 minutos
```

### Cálculo

```
MTTR = Time Detected + Time Debugged + Time Fixed + Time Deployed

Exemplo:
2024-02-20 15:00 - Problema detectado
2024-02-20 15:15 - Causa identificada (15 min debug)
2024-02-20 15:25 - Fix implementado (10 min)
2024-02-20 15:35 - Testado (10 min)
2024-02-20 15:38 - Deployed (3 min)

MTTR = 38 minutos

Rastreamento:
├─ Incident tracking: PagerDuty, Opsgenie
├─ Correlação: Logs + Deploy logs
├─ Dashboard: Histórico de incidentes
```

---

## 4. Change Failure Rate

**O que mede:** Porcentagem de mudanças que causam problema em produção.

**Definição:** Percentual de deploys que resultam em falha, incidente ou degradação em produção.

### Categorias de Desempenho

```
ELITE:        0-15%
HIGH:         15-30%
MEDIUM:       30-50%
LOW:          > 50%
```

### Exemplos

```
Exemplo 1: Startup com muitos deploys
├─ Deploys/mês: 100
├─ Falhas/mês: 8
├─ CFR: 8%
└─ Análise: Elite, com boa automação

Exemplo 2: Banco tradicional
├─ Deploys/mês: 4
├─ Falhas/mês: 2
├─ CFR: 50%
└─ Análise: Baixo, falta confiança

Exemplo 3: E-commerce
├─ Deploys/dia: 20
├─ Falhas/dia: 2
├─ CFR: 10%
└─ Análise: Elite, automação completa
```

### Causas de Falha

```
Análise de 1000 falhas em produção:

Teste incompleto:        35% ← Cobertura baixa (ref: 02 - Test Coverage)
Deploy problem:          25% ← Automação ruim (ref: 06 - CI/CD)
Config issue:            15% ← Environment mismatch
Breaking changes:        15% ← Sem contract testing (ref: 03 - Contract)
Data migration:          10% ← Sem plano

Insights:
├─ 35% seria evitado com mais testes
├─ 25% seria evitado com melhor CI/CD
├─ 15% seria evitado com melhor env management
└─ 15% seria evitado com contract testing
```

### Como Melhorar

```
Reduzir para 15% (Elite):

Ação 1: Aumentar cobertura (80%+)
├─ Referência: 02 - Unit Testing
├─ Benefício: Evita 30-35% das falhas
└─ Tempo: 2-3 sprints

Ação 2: Implementar Test Pipeline
├─ Referência: 06 - Test Pipelines
├─ Benefício: Evita 20-25% das falhas
└─ Tempo: 4-6 semanas

Ação 3: Feature Flags
├─ Referência: 06 - Feature Flags
├─ Benefício: Permite rollback rápido (não evita, mitiga)
└─ Tempo: 2 semanas

Ação 4: Contract Testing
├─ Referência: 03 - Contract Testing
├─ Benefício: Evita 10-15% das falhas
└─ Tempo: 3-4 sprints

Resultado final: De 50% para 15% (Elite)
Timeline: ~4-5 meses
```

### Cálculo

```
Change Failure Rate = Failed Deploys / Total Deploys × 100

Exemplo:
Total de deploys em fevereiro: 40
Deploys que causaram problema: 4
CFR = (4 / 40) × 100 = 10%

O que contar como "falha":
├─ ✅ Incidente que precisou rollback
├─ ✅ Degração de performance > 10%
├─ ✅ Erro não tratado em produção
├─ ✅ Usuários afetados (> threshold)
│
├─ ❌ Warning em logs (não impacto)
├─ ❌ Deploy abortado antes de chegar prod
├─ ❌ Problema em staging (não prod)
└─ ❌ Pequenas correções de UI

Rastreamento:
├─ Manual: Correlnar deploys com incidents
├─ Automático: Monitoramento → Deploy logs
├─ Dashboard: Grafana/Datadog
```

---

## Dashboard DORA Ideal

```
┌─────────────────────────────────────────────────────────────┐
│          DORA Metrics Dashboard - Última 30 dias            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  🚀 Deployment Frequency:         12x/dia          ELITE ✅ │
│     Target: 1x/dia+               Trend: ↑ +15%  (good)   │
│                                                             │
│  ⏱️  Lead Time for Changes:        3.5 hours       HIGH ✓ │
│     Target: < 4 hours             Trend: ↓ -20%  (good)   │
│                                                             │
│  🔧 Mean Time to Recovery:        45 minutos      ELITE ✅ │
│     Target: < 1 hour              Trend: ↓ -30%  (good)   │
│                                                             │
│  📊 Change Failure Rate:           8%              ELITE ✅ │
│     Target: < 15%                 Trend: ↓ -5%   (good)   │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                   Histórico (últimos 3 meses)              │
│                                                             │
│  DF: ▁▃▅▇██ (crescendo) ✅                                 │
│  LT: ▇▆▅▃▁ (diminuindo) ✅                                 │
│  MTTR: ▇▆▆▅▅▄ (estável) ✅                                │
│  CFR: ▇▆▆▅▄▃ (melhorando) ✅                              │
│                                                             │
└─────────────────────────────────────────────────────────────┘

Status Geral: 🟢 ELITE PERFORMANCE
```

---

## Conexões com Outras Pastas

### Com Test Pyramid (01 - Foundations)
```
Distribuição correta das testes:
├─ Muitos unitários (rápidos) → Deploy seguro
├─ Moderada integração → Confiança em changes
└─ Poucos E2E (lento) → Validação final

Resultado: CFR baixo, Lead time curto
```

### Com Shift-Left (01 - Foundations)
```
Testes mais cedo:
├─ Desenvolvimento: TDD reduz bugs
├─ PR: Validação antes de merge
├─ Staging: Fumaça final

Resultado: CFR & MTTR baixos
```

### Com Unit Testing (02 - Unit Testing)
```
Testes unitários robustos:
├─ Cobertura 80%+
├─ Fast feedback
└─ Evita 35% das falhas

Resultado: CFR reduzido
```

### Com Test Coverage (02 - Test Coverage)
```
Métricas de cobertura:
├─ Line coverage: Cobertura básica
├─ Branch coverage: Mais rigoroso
└─ Meta: 80%+ para mission-critical

Resultado: CFR & MTTR melhoram
```

### Com Integration Testing (03 - Integration)
```
Testes de integração completos:
├─ Database testing
├─ Contract testing
└─ API validation

Resultado: Evita 15% das falhas
```

### Com E2E Testing (04 - E2E)
```
E2E bem estruturado:
├─ Fluxos críticos cobertos
├─ E2E strategy claro
├─ Sem flakiness (ref: Flakiness)

Resultado: Confiança em deploy
```

### Com Test Strategy (05 - Test Strategy)
```
Estratégia clara:
├─ DoD define testes obrigatórios
├─ Quality gates automáticos
├─ Code review integrado

Resultado: CFR & Lead time melhoram
```

### Com Quality Gates (05 - Quality Gates)
```
Gates que bloqueiam:
├─ Testes devem passar
├─ Cobertura deve estar OK
├─ Sem vulnerabilidades

Resultado: CFR reduzido
```

### Com CI/CD (06 - CI/CD)
```
Test Pipelines + PR Testing + Feature Flags:
├─ Feedback rápido (Lead time ↓)
├─ Deploy seguro (CFR ↓)
├─ Rollback fácil (MTTR ↓)
└─ Deploy frequente (DF ↑)

Resultado: ELITE DORA performance
```

---

## Jornada de Melhoria

### Timeline Realista

```
Mês 1: Baseline
├─ DF: 1x/mês (LOW)
├─ LT: 2 semanas (LOW)
├─ MTTR: 8 horas (LOW)
├─ CFR: 40% (LOW)
└─ Score: 🔴 LOW PERFORMANCE

Meta: Implementar CI/CD básico
├─ ✅ GitHub Actions
├─ ✅ Testes automatizados
├─ ✅ Deploy simples

Mês 2-3: Rápida melhoria
├─ DF: 2-3x/semana (MEDIUM)
├─ LT: 5 dias (MEDIUM)
├─ MTTR: 4 horas (MEDIUM)
├─ CFR: 30% (MEDIUM)
└─ Score: 🟡 MEDIUM PERFORMANCE

Meta: Quality gates + Feature flags
├─ ✅ Pull Request Testing
├─ ✅ Coverage gates
├─ ✅ Feature flags

Mês 4-6: Consolidação
├─ DF: 2-5x/dia (HIGH)
├─ LT: 1 dia (HIGH)
├─ MTTR: 1 hora (HIGH)
├─ CFR: 20% (HIGH)
└─ Score: 🟢 HIGH PERFORMANCE

Meta: Observabilidade + Automação
├─ ✅ Logs centralizados
├─ ✅ Auto remediation
├─ ✅ Canary deployments

Mês 7+: Elite
├─ DF: 10+x/dia (ELITE)
├─ LT: < 2 horas (ELITE)
├─ MTTR: < 30min (ELITE)
├─ CFR: 10% (ELITE)
└─ Score: 🔵 ELITE PERFORMANCE
```

---

## Anti-Patterns e Armadilhas

### ❌ Gaming Metrics

```
Errado: Aumentar Deployment Frequency artificialmente
├─ Deploy vazio (sem mudança real)
├─ Deploy que é revertido em 1h
└─ Resultado: Métrica sobe, CFR explode

Correto:
├─ Medir apenas deploys com valor
├─ Acompanhar com CFR
└─ Objetivo: Velocidade com confiança
```

### ❌ Ignorar Change Failure Rate

```
Errado: Priorizar deploy frequency acima de tudo
├─ 100 deploys/dia
├─ 50% precisam rollback
└─ Resultado: Caos em produção

Correto:
├─ Balancear velocidade com confiabilidade
├─ Deploy frequente + CFR baixo
└─ Objetivo: Velocidade sustentável
```

### ❌ Foco Apenas em Métricas Rápidas

```
Errado: Apenas medir DF e LT, ignorar CFR e MTTR
├─ Parece que está indo bem
├─ Mas produção é instável
└─ Resultado: Confiança falsa

Correto:
├─ Acompanhar all 4 DORA metrics
├─ Correlacionar entre si
└─ Objetivo: Visão 360° de saúde
```

---

## Conclusão

DORA Metrics são **4 indicadores simples mas poderosos** que correlacionam-se com sucesso real de software:

- ✅ Não são vanity metrics
- ✅ Aplicáveis a qualquer organização
- ✅ Orientam decisões claras
- ✅ Medem o que importa

Combine DORA com QA Metrics (próximo doc) para visão completa de qualidade and performance de delivery.
