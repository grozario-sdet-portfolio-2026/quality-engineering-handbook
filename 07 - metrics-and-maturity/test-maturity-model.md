# Test Maturity Model

## Origem e Conceito

Um Test Maturity Model é um framework que descreve os níveis de evolução da capacidade de teste em uma organização, do caótico até a excelência.

Originado em modelos de maturidade de software (como CMM - Capability Maturity Model), o Test Maturity Model guia organizações em sua jornada de qualidade.

**Definição:** Test Maturity Model é um framework que descreve estágios progressivos de maturidade em testes, desde sem testes até testes completamente integrados na cultura, arquitetura e processo.

---

## Por Que Test Maturity Model?

### Problemas que resolvem:

1. **Falta de direção** - Onde começar com testes?
2. **Saltos inadequados** - Implementar CI/CD sem testes base
3. **Inconsistência entre times** - Cada time faz diferente
4. **Falta de priorização** - Tudo parece igualmente importante
5. **Avaliação subjetiva** - É maduro ou não?
6. **Roadmap confuso** - Qual o próximo passo?
7. **Time desmotivado** - Sem ver progresso

### Benefícios:

- ✅ Caminho claro para maturidade
- ✅ Métricas objetivas de progresso
- ✅ Priorização guiada de investimentos
- ✅ Consistência entre times
- ✅ Motivação através de marcos

---

## Os 5 Níveis de Maturidade

### Nível 1: Caótico (Ad-Hoc)

**Características:**
- Sem strategy formal de testes
- Testes manuais apenas
- Sem automação
- Débito técnico incontrolável
- Sem métricas
- Testes reacionários

**Descrição:**
```
Cenário:
├─ Dev escreve feature
├─ QA testa manualmente (1-2 dias)
├─ Se passar, vai para produção
├─ Se problema em produção, hotfix urgente
└─ Voltar ao passo 1

Problemas:
├─ Lead time: 2-4 semanas
├─ Deployment frequency: 1-2x/mês
├─ Change failure rate: 30-50%
├─ Bugs em produção: Frequente
├─ Frustração do time: ALTA

Exemplo:
├─ Startup com 3 devs, 0 QAs
├─ Lançando MVP
├─ "Depois temos tempo para testes"
```

**Passagem para Nível 2:**

```
Investimento: 3-4 sprints
Ações necessárias:
├─ ✅ Reconhecer problema
├─ ✅ Treinar dev em unit testing
├─ ✅ Implementar first 10 unit tests
├─ ✅ Setup CI/CD básico (GitHub Actions)
├─ ✅ Define DoD que inclui "testes"
└─ Resultado: ~20 unit tests

Métrica de sucesso: 30+ unit tests executando
```

---

### Nível 2: Repetível (Managed)

**Características:**
- Unit tests básicos
- Alguns testes de integração
- CI/CD inicial (github actions)
- Quality gates iniciais
- Métricas começam ser coletadas
- Processo documentado

**Descrição:**
```
Cenário:
├─ Dev escreve código + testes unitários
├─ Push dispara CI (linter, unit tests)
├─ Se passar, cria PR
├─ Code review + QA manual
├─ Se OK, merge para main
├─ Deploy manual em produção
└─ Monitorar em produção

Melhorias vs Nível 1:
├─ Lead time: 1-2 semanas
├─ Deployment frequency: 1x/semana
├─ Change failure rate: 20-30%
├─ Bugs em produção: Reduzido
├─ Frustração do time: MODERADA

Exemplo:
├─ SaaS com 8 devs, 2 QAs
├─ Começou a investir em testes
├─ CI/CD rodar sem humanos

Métricas:
├─ Coverage: 50-70%
├─ Unit tests: 100-300
├─ Defect leakage: 20-30%
```

**Passagem para Nível 3:**

```
Investimento: 6-8 sprints
Ações necessárias:
├─ ✅ Aumentar coverage para 80%+
├─ ✅ Tests de integração (ref: 03)
├─ ✅ E2E smoke tests (ref: 04)
├─ ✅ Test Pipeline completo (ref: 06)
├─ ✅ Pull Request Testing (ref: 06)
├─ ✅ Feature flags (ref: 06)
├─ ✅ SonarQube ou similar
└─ ✅ Defect metrics coletadas

Métrica de sucesso: 80%+ coverage, < 15% leakage
```

---

### Nível 3: Definido (Defined)

**Características:**
- Test strategy clara
- Pirâmide de testes implementada
- Coverage 80%+
- Test pipeline completo
- PR Testing automatizado
- Feature flags para deploy seguro
- Code quality gates
- Defect metrics baixas

**Descrição:**
```
Cenário:
├─ Planning: DoD inclui estratégia de testes
├─ Dev escreve código com TDD (ref: 01 Shift-Left)
├─ Testes: Unit + Integration + E2E
├─ PR: Automático linting, types, tests, coverage
├─ Merge: Feature flag ativado
├─ Deploy: CD automático se tudo OK
├─ Release: Feature flag em 10% usuários
├─ Monitor: Métricas e alertas
├─ Escalação: Feature para 100% progressivamente

Melhorias vs Nível 2:
├─ Lead time: 2-4 dias
├─ Deployment frequency: 5-10x/dia
├─ Change failure rate: 10-15%
├─ Bugs em produção: Raros
├─ Frustração do time: BAIXA

Exemplo:
├─ SaaS maduro com 30 devs, 5 QAs
├─ Testes como parte da cultura
├─ "Definition of Done = inclui testes"

Métricas:
├─ Coverage: 80-90%
├─ Unit tests: 500-1000
├─ Integration tests: 50-100
├─ E2E smoke: 10-20
├─ Defect leakage: 7-12%
├─ Quality grade: A/B
├─ DORA DF: 1-10x/dia
├─ DORA LT: 1 dia
├─ DORA CFR: 10-15%
```

**Características-chave:**
```
Planning Fase:
└─ DoD includes "strategy de testes"
  ├─ Unit testes obrigatórios
  ├─ Integration testes se necessário
  └─ E2E para jornadas críticas

Development Fase:
├─ TDD é a norma
├─ Code review inclui "tem testes?"
├─ 80%+ coverage obrigatório
└─ Test pairs em código complexo

PR Phase:
├─ Linting automático
├─ Type checking
├─ Testes rodam automaticamente
├─ Coverage verificado
├─ Code quality scanned
├─ Security checks
└─ → Merge ou comentário automático

Deploy Phase:
├─ Feature flag OFF por padrão
├─ Deploy em produção (confiança alta)
├─ Feature flag ON para 10% (canary)
├─ Monitora por 2h
├─ Se OK, 25%, depois 100%
└─ Se problema, rollback automático

Quality Metrics:
├─ Dashboard em tempo real
├─ DORA metrics diário
├─ Quality metrics semanal
├─ Leakage rate monitorado
└─ Alertas para degradação
```

**Passagem para Nível 4:**

```
Investimento: 8-12 sprints
Ações necessárias:
├─ ✅ Test coaching culture
├─ ✅ TDD é padrão (not just recommended)
├─ ✅ Observabilidade em produção (ref: MTTR)
├─ ✅ Auto remediation básica
├─ ✅ Flakey test detection e fixing
├─ ✅ Test data management
├─ ✅ API contract testing (ref: 03)
│
Meta de sucesso: 
├─ 90%+ coverage
├─ < 8% defect leakage
├─ DORA Elite em 2+ métricas
```

---

### Nível 4: Gerenciado Quantitativamente (Quantitatively Managed)

**Características:**
- Testes instrumentados e otimizados
- Métricas em tempo real
- Predicted quality (pode prever bugs)
- Performance tests automatizados
- Chaos engineering inicial
- Observabilidade completa
- Auto remediation para falhas conhecidas
- Data-driven decision making

**Descrição:**
```
Cenário:
├─ Todo Sprint: Data-driven retrospectives
├─ Todo Dia: Métricas DORA publicadas
├─ Todo Deploy: Canary review automático
├─ Produção: Observabilidade completa
│           Health checks contínuos
│           Alertas inteligentes
│           Runbooks automatizadas
├─ Feedback: Direto do usuário
└─ Otimização: Baseada em dados, não opinions

Melhorias vs Nível 3:
├─ Lead time: 1-4 horas
├─ Deployment frequency: 50-500x/dia
├─ Change failure rate: 5-10%
├─ Bugs em produção: Muito raro
├─ Frustração do time: MÍNIMA

Exemplo:
├─ Google, Netflix, Amazon
├─ Deploy 100x/dia
├─ Confiança total
├─ Cultura de qualidade

Métricas:
├─ Coverage: 90-95%+
├─ Performance tests: Contínuos
├─ Chaos tests: Regular
├─ Defect leakage: 2-5%
├─ Quality grade: A
├─ DORA DF: 50-500x/dia ELITE
├─ DORA LT: < 1 hora ELITE
├─ DORA CFR: < 10% ELITE
├─ DORA MTTR: < 30min ELITE
```

**Características-chave:**
```
Automação Completa:
├─ Testes rodam em paralelo, não sequencial
├─ GPU-accelerated test runs
├─ ML para predizer flaky tests
├─ Browser farm para E2E
└─ Testes são muito rápido (< 2 minutos total)

Observabilidade:
├─ Distributed tracing (Jaeger, Datadog)
├─ Metrics (Prometheus, Graphite)
├─ Logs (ELK, Splunk)
├─ APM (Newrelic, Datadog)
└─ Correlação automática entre eventos

Auto Remediation:
├─ Rollback automático em degradação
├─ Feature flag disabled auto
├─ Database failover automático
├─ Scale up/down automático
└─ Runbook Executor automático

Quality Prediction:
├─ ML models predizem bugs
├─ A/B testing contínuo
├─ Experiment framework
└─ Gradual rollouts baseados em dados
```

**Passagem para Nível 5:**

```
Investimento: 12+ sprints
Ações necessárias:
├─ ✅ Cultura de experimentação
├─ ✅ Observabilidade avançada
├─ ✅ Auto remediation sofisticada
├─ ✅ Machine learning para quality prediction
├─ ✅ Performance engineering
├─ ✅ Chaos engineering avançado
└─ ✅ Feedback loops em tempo real

Meta de sucesso:
├─ < 5% defect leakage
├─ 4/4 DORA Metrics ELITE
```

---

### Nível 5: Otimizado (Optimized)

**Características:**
- Inovação contínua em testes
- Culture of experimentation
- Chaos engineering production-ready
- AI/ML para detecção de anomalias
- Zero-incident mindset
- Continuous learning e improvement
- Industry leadership

**Descrição:**
```
Cenário:
├─ Toda feature é um experimento
├─ Deploy sem medo (confiança 100%)
├─ Problemas detectados antes de impactar usuários
├─ Mudanças tão pequenas que risco é ~zero
├─ Feedback de usuários em tempo real
├─ Otimização contínua baseada em dados
└─ Inovação acelerada sem sacrificar qualidade

Melhorias vs Nível 4:
├─ Lead time: < 1 hora (ou instantâneo)
├─ Deployment frequency: 1000+x/dia
├─ Change failure rate: 0-2%
├─ Bugs em produção: Praticamente zero
├─ Frustração do time: ZERO

Exemplo:
├─ Google (production systems)
├─ Netflix
├─ Amazon Prime
├─ Meta (Facebook)

Métricas:
├─ Coverage: 95%+
├─ Defect leakage: 0-2%
├─ Quality grade: A+
├─ DORA DF: 1000x/dia ELITE
├─ DORA LT: < 30 minutos ELITE
├─ DORA CFR: < 5% ELITE
├─ DORA MTTR: < 5 minutes ELITE
```

**Características-chave:**
```
Innovation Culture:
├─ Qualquer um pode fazer deploy
├─ Qualquer mudança é segura por design
├─ Falhas são dados, não crimes
├─ Postmortems construtivos
└─ Aprendizado contínuo

Advanced Testing:
├─ Formal methods (verificação matemática)
├─ Simulation e emulation
├─ Chaos engineering em produção
├─ Resilience testing
├─ Synthetic monitoring global
└─ User journey validation

AI/ML Powered:
├─ Anomaly detection
├─ Predictive failures
├─ Auto remediation inteligente
├─ Test generation automática
├─ Bug prediction com high accuracy
└─ Performance optimization automática

Zero Incident Mindset:
├─ Incidents são learning opportunities
├─ Blameless postmortems
├─ Continuous improvement
├─ Resilience built-in
└─ Recovery is tested regularly
```

---

## Matriz de Evolução

```
Aspecto              Nível 1    Nível 2      Nível 3      Nível 4      Nível 5
─────────────────────────────────────────────────────────────────────────────
Unit Tests           None       Basic (50%)  Complete (80%) Auto (90%)    Perfect (95%+)
Integration Tests    None       Some         Moderate       Extensive      Complete
E2E Tests            None       Manual       Automated      Optimized      AI-Driven
Testes Strategy      None       Basic        Defined        Refined        Innovated
CI/CD                None       Basic        Complete       Advanced       Autonomous
Coverage             0%         50-70%       80-90%         90-95%         95%+
Quality Metrics      None       Collected    Monitored      Predicted      Optimized
Deployment Freq      1-2/mês    1/semana     5-10/dia       50-500/dia     1000+/dia
Lead Time            2-4 weeks  1-2 weeks    1-4 days       1-4 hours      < 1 hour
MTTR                 24+ hours  5-8 hours    1-2 hours      15-30 min      < 5 min
CFR                  30-50%     20-30%       10-15%         5-10%          0-2%
Defect Leakage       > 30%      20-30%       7-12%          2-5%           0-2%
Team Size            2-5 dev    8 dev+QA     20+ mixed      50+ mixed      100+ mixed
Culture              Chaotic    Managed      Disciplined    Data-driven    Experimental
```

---

## Jornada Típica de Organização

### Exemplo: SaaS em Crescimento

```
2023 Q1: Lançamento (Nível 1)
├─ 3 devs, 0 QAs
├─ MVP com 10 features
├─ Zero testes
├─ Bugs em produção diariamente
├─ DORA: DF 1x/semana, CFR 40%
└─ "Depois temos tempo para testes"

2023 Q2-Q3: Primeiro Investimento (Nível 1.5 → 2)
├─ Contratado 1 QA
├─ Dev team treinado em unit testing
├─ 50 unit tests escritos
├─ CI básico implementado
├─ Coverage: 40%
├─ Bugs reduzidos 20%
└─ DORA: DF 1x/semana, CFR 25%

2023 Q4: Expansão (Nível 2.5 → 3)
├─ 8 devs, 2 QAs
├─ Test strategy documentado
├─ Coverage: 70%
├─ Integration tests: 20
├─ PR testing automatizado
├─ Feature flags implemented
├─ SonarQube setup
└─ DORA: DF 3-5x/dia, CFR 15%, LT 3-5 dias

2024 Q1: Consolidação (Nível 3)
├─ 10 devs, 3 QAs
├─ Coverage: 82%
├─ All testes automated
├─ E2E suite: 15 tests
├─ Defect leakage: 10%
├─ Deploy frequency: 10x/dia
└─ DORA: DF 10x/dia, CFR 12%, LT 2-3 hours, MTTR 1 hour

2024 Q2-Q3: Optimization (Nível 3.5 → 4)
├─ 15 devs, 3 QAs (QA focus on building test infrastructure)
├─ Coverage: 88%
├─ Observability implemented
├─ Auto remediation para known issues
├─ Chaos testing started
├─ Performance tests integrated
├─ Defect leakage: 5%
└─ DORA: DF 50x/dia, CFR 8%, LT 1 hour, MTTR 20 min

2024 Q4+: Elite (Nível 4)
├─ 20+ devs, 2 QAs (QA → Quality architects)
├─ Coverage: 92%
├─ Testes rodam em < 2 min
├─ Deploy 100x+/dia
├─ Defect leakage: 2-3%
├─ Zero production incidents (monthly)
└─ DORA: All metrics ELITE ✅
```

**Investimento Total:** ~18 meses para Nível 4

---

## Autossessment: Qual é Seu Nível?

### Questionário Rápido

```
1. Unit Testing
   [ ] Nenhum ou quase nenhum (Nível 1)
   [ ] ~50% cobertura em código novo (Nível 2)
   [ ] 80%+ cobertura, maior parte do código (Nível 3)
   [ ] 90%+ cobertura, automaticamente verificado (Nível 4)
   [ ] 95%+, muito rápido, muito confiável (Nível 5)

2. CI/CD
   [ ] Manual ou inexistente (Nível 1)
   [ ] Básico, alguns testes rodam (Nível 2)
   [ ] Completo, linting+testes+análise (Nível 3)
   [ ] Muito rápido, paralelo, otimizado (Nível 4)
   [ ] Autônomo, preditivo, AI-assisted (Nível 5)

3. Deployment Frequency
   [ ] 1-2x/mês (Nível 1)
   [ ] 1x/semana (Nível 2)
   [ ] 5-10x/dia (Nível 3)
   [ ] 50-500x/dia (Nível 4)
   [ ] 1000x/dia (Nível 5)

4. Change Failure Rate
   [ ] 30-50% (Nível 1)
   [ ] 20-30% (Nível 2)
   [ ] 10-15% (Nível 3)
   [ ] 5-10% (Nível 4)
   [ ] 0-2% (Nível 5)

5. Defect Leakage
   [ ] > 30% (Nível 1)
   [ ] 20-30% (Nível 2)
   [ ] 7-12% (Nível 3)
   [ ] 2-5% (Nível 4)
   [ ] 0-2% (Nível 5)

Pontuação:
Maioria L1: Nível 1 (Ad-Hoc)
Maioria L2: Nível 2 (Repeated)
Maioria L3: Nível 3 (Defined)
Maioria L4: Nível 4 (Quantitatively Managed)
Maioria L5: Nível 5 (Optimized)
```

---

## Roadmap de Evolução

### Nível 1 → 2 (3-4 meses)

```
Sprint 1-2: Foundation
├─ Training: Unit testing, TDD basics
├─ Setup: GitHub Actions CI
├─ DoD update: "testes obrigatórios"
└─ Goal: First 20 tests

Sprint 3-4: Scale
├─ Coverage tools setup
├─ Quality gates: Linter + testes
├─ Integrate PR checks
└─ Goal: 50+ tests, 50% coverage

Result: Nível 2 achieved ✅
```

### Nível 2 → 3 (6-8 meses)

```
Sprint 1-2: Integration Tests
├─ Setup database tests (ref: 03)
├─ API integration tests
└─ Goal: 20+ integration tests

Sprint 3-4: E2E Basics
├─ Setup Playwright/Cypress
├─ Smoke tests critical flows
├─ Deal with flakiness (ref: 04)
└─ Goal: 10+ E2E tests

Sprint 5-6: Feature Flags
├─ Feature flag implementation (ref: 06)
├─ Blue-green/canary setup
├─ Rollout strategy
└─ Goal: Live feature flags

Sprint 7-8: SonarQube & Refinement
├─ SonarQube integration
├─ Quality gates enforcement
├─ Runbooks & documentation
└─ Goal: Defined process

Result: Nível 3 achieved ✅
```

### Nível 3 → 4 (8-12 meses)

```
Sprint 1-4: Performance & Chaos
├─ Performance tests setup
├─ Chaos engineering intro
├─ Resilience testing
└─ Goal: Infra testing

Sprint 5-8: Observability
├─ Distributed tracing
├─ Metrics & dashboards
├─ Alert setup
├─ Runbook automation
└─ Goal: Complete visibility

Sprint 9-12: Auto Remediation
├─ Basic auto-remediation
├─ Blue-green deployment
├─ ML/Prediction models
└─ Goal: Autonomous systems

Result: Nível 4 achieved ✅
```

### Nível 4 → 5 (12+ months)

```
Continuous Innovation:
├─ Advanced chaos engineering
├─ ML-powered testing
├─ Formal verification
├─ Experimentation framework
├─ Continuous learning culture
└─ Goal: Industry-leading practices

Result: Nível 5 maintained ✅
```

---

## Conexões com Pastas

```
Níveis de Maturidade Utilizam Práticas De:

Nível 1: Nenhum (ou muito pouco)

Nível 2: 
├─ 01 - Foundations (Test Pyramid basics)
├─ 02 - Unit Testing (escrever testes)
└─ 05 - Definition of Done

Nível 3:
├─ 01 - Shift-Left (TDD cultura)
├─ 02 - Test Coverage (80% meta)
├─ 03 - Integration Testing
├─ 04 - E2E Strategy
├─ 05 - Quality Gates
└─ 06 - CI/CD & Automation

Nível 4:
├─ Tudo anterior +
├─ 06 - Feature Flags (canary)
├─ Performance testing
├─ Observability tools
├─ Auto remediation
└─ Resiliência testing

Nível 5:
├─ Tudo anterior +
├─ Experimentation framework
├─ Formal methods
├─ AI/ML powered
└─ Innovation culture
```

---

## Conclusão

Test Maturity Model fornece um roadmap claro para evolução, da desorganização para a excelência.

**Não é sobre ser Nível 5.**  
**É sobre progredir consistentemente**, entendendo onde você está e qual é o próximo passo.

Cada nível tem:
- ✅ Características claras
- ✅ Métricas definidas
- ✅ Benefícios reais
- ✅ Caminho tangível

Combine com [DORA Metrics](dora-metrics.md) e [Quality Metrics](quality-metrics.md) para uma visão completa de sua jornada de qualidade.
