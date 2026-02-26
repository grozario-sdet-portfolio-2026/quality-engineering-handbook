# Quality Metrics

## Origem e Conceito

Quality Metrics são indicadores quantificáveis que medem a qualidade, confiabilidade e manutenibilidade de um produto de software.

Diferente de DORA Metrics que focam em **velocidade de delivery**, Quality Metrics focam em **qualidade do código e produto** entregues.

**Definição:** Quality Metrics são medições objetivas sobre o estado de qualidade do software, incluindo cobertura de testes, bugs, vulnerabilidades de segurança, complexidade de código e defect leakage.

---

## Por Que Quality Metrics?

### Problemas que resolvem:

1. **Qualidade subjetiva** - "Código está bom" sem dados
2. **Débito técnico invisível** - Acumula sem perceber
3. **Confiança falsa** - Código parece OK, mas não é
4. **Regressões escondidas** - Mudanças quebram código antigo
5. **Segurança negligenciada** - Vulnerabilidades passam
6. **Complexidade crescente** - Código fica cada vez pior
7. **Surpresas em produção** - Bugs descobertos tarde

### Benefícios:

- ✅ Visibilidade de qualidade objetiva
- ✅ Detecta débito técnico cedo
- ✅ Guia priorização de work
- ✅ Monitora segurança contínua
- ✅ Reduz bugs em produção

---

## Categorias de Quality Metrics

### 1. Cobertura de Testes

Mede a porcentagem de código testado.

**Referência:** [Test Coverage (02 - Unit Testing)](../02%20-%20unit-testing/test-coverage.md)

```yaml
Tipos de Cobertura:
├─ Line coverage: % de linhas executadas
├─ Branch coverage: % de ramificações testadas
├─ Function coverage: % de funções chamadas
└─ Path coverage: % de caminhos possíveis

Meta Recomendada:
├─ Mínimo aceitável: 70%
├─ Bom: 80%
├─ Excelente: 90%+
└─ Crítico: 95%+

Cálculo:
coverage = (linhas_testadas / linhas_totais) × 100

Exemplo:
├─ Total linhas: 1000
├─ Linhas testadas: 820
├─ Coverage: 82% ✅ BOM

Dashboard:
├─ Trend: Mês passado 79%, agora 82% ✓ aumentando
├─ Por arquivo: Admin.js (95%), API.js (75%)
└─ Novos commits: Média 85% (obrigatório manter)
```

**Interpretação:**
```
Coverage 95%+ ✅ EXCELENTE
├─ Mudanças podem ser feitas com confiança
├─ Refatorações seguras
├─ Débito técnico baixo

Coverage 80-90% ✓ BOM
├─ Cobertura adequada
├─ Risco aceitável
└─ Pode focar em cobertura de risco alto

Coverage 70-80% ⚠️ ACEITÁVEL
├─ Risco moderado
├─ Problemas podem ser perdidos
└─ Priorizar aumentar cobertura

Coverage < 70% ❌ POBRE
├─ Risco alto de bugs
├─ Mudanças arriscadas
└─ PARAR e focar em testes
```

---

### 2. Defect Leakage Rate

Mede quantos bugs chegam em produção que deveriam ter sido detectados.

**Definição:** Percentual de bugs encontrados **em produção** vs **total de bugs** (encontrados antes + em produção).

```
Defect Leakage = Bugs em Produção / (Bugs Encontrados + Bugs em Produção) × 100

Exemplo:
├─ Bugs encontrados em development/QA: 20
├─ Bugs encontrados em produção: 4
├─ Total: 24 bugs
├─ Leakage Rate: (4 / 24) × 100 = 16.7%

Interpretação:
├─ ELITE: < 5% (excelente triagem)
├─ HIGH: 5-10% (bom, alguns escapam)
├─ MEDIUM: 10-20% (moderado, investir em QA)
└─ LOW: > 20% (pobre, muitos escapam)
```

**Como Melhorar:**

```
Aumentar Detecção Pré-Produção:

Ação 1: Aumentar cobertura de testes
├─ Referência: 02 - Unit Testing, Coverage
├─ Impacto: Reduz 30-40% dos bugs
└─ Lead time: 2-3 sprints

Ação 2: Implementar Test Pipeline
├─ Referência: 06 - Test Pipelines
├─ Impacto: Reduz 25-35% dos bugs
└─ Lead time: 4-6 semanas

Ação 3: Quality Gates + Code Analysis
├─ Referência: 05 - Quality Gates
├─ Impacto: Reduz 15-20% dos bugs
└─ Lead time: 2-3 sprints

Ação 4: E2E + Flakiness Fix
├─ Referência: 04 - E2E Strategy, Flakiness
├─ Impacto: Reduz 20-25% dos bugs
└─ Lead time: 3-4 sprints

Resultado: De 20% para 5% (ELITE)
Timeline: ~3-4 meses
```

### 3. Bugs Density

Mede quantidade de bugs por tamanho de código.

```
Bugs Density = Bugs Encontrados / Lines of Code

Exemplo:
├─ 150 bugs encontrados em 1 ano
├─ 50,000 linhas de código
├─ Density: (150 / 50,000) × 1000 = 3 bugs por 1000 linhas

Categorias:
├─ < 1 bug/1000 LOC: EXCELENTE ✅
├─ 1-2 bugs/1000 LOC: BOM ✓
├─ 2-4 bugs/1000 LOC: ACEITÁVEL ⚠️
└─ > 4 bugs/1000 LOC: POBRE ❌

Insight:
├─ Densidade alta = Código complexo ou mal testado
├─ Densidade baixa = Bom design + testes
└─ Monitor por arquivo para identificar problemáticos
```

### 4. Code Smell & Technical Debt

Mede qualidade estrutural do código.

**Ferramentas:** SonarQube, CodeClimate, Codacy

```
Métricas rastreadas:
├─ Code smells: Padrões problemáticos
├─ Duplicação: Código repetido
├─ Complexidade: Funções muito complexas
├─ Vulnerabilidades: Segurança
└─ Hotspots: Áreas de risco

Exemplo SonarQube Report:
┌─────────────────────────────────────┐
│ Qualidade do Código                 │
├─────────────────────────────────────┤
│ A (Grade A) - Confiável ✅          │
├─────────────────────────────────────┤
│ Problemas:                          │
│ ├─ 12 Code Smells (9 minor)         │
│ ├─ 3 Bugs (Prioridade: HIGH)        │
│ ├─ 1 Vulnerabilidade (CRITICAL)     │
│ ├─ 2.5% Duplicção                   │
│ └─ Complexidade Ciclomática: 4 avg  │
├─────────────────────────────────────┤
│ Hotspots:                           │
│ ├─ payment.js (Complexidade 15)     │
│ └─ auth.js (Duplicação 5%)          │
└─────────────────────────────────────┘

Interpretação:
├─ Grade A/B: Qualidade boa
├─ Grade C: Qualidade aceitável
├─ Grade D: Qualidade pobre
└─ Grade E: Qualidad crítica
```

**Technical Debt Index:**

```
Debt Index = (Estimated Effort to Fix / Development Time) × 100

Exemplo:
├─ Tempo de desenvolvimento até agora: 2000 horas
├─ Esforço estimado para limpar: 200 horas
├─ Debt Index: (200 / 2000) × 100 = 10%

Categorias:
├─ < 5%: Baixo (saudável)
├─ 5-10%: Moderado (gerir)
├─ 10-20%: Alto (investir em limpeza)
└─ > 20%: Crítico (parar e refatorar)
```

### 5. Vulnerabilidade de Segurança

Mede exposição de segurança.

```
Tipos rastreados:
├─ CRITICAL: Exploração imediata (0-1 dia para fix)
├─ HIGH: Risco alto (1-3 dias para fix)
├─ MEDIUM: Risco moderado (1-2 semanas)
└─ LOW: Risco baixo (nice to have)

Exemplo Dashboard:
┌──────────────────────────────────┐
│ Security Report                  │
├──────────────────────────────────┤
│ CRITICAL: 0 ✅                   │
│ HIGH: 1 ⚠️ (jquery 2.x)          │
│ MEDIUM: 3                        │
│ LOW: 8                           │
├──────────────────────────────────┤
│ Action Items:                    │
│ ├─ Upgrade jquery to 3.6         │
│ ├─ Fix SQL injection in API      │
│ └─ Update dependencies (npm)     │
└──────────────────────────────────┘

SLA:
├─ CRITICAL: Fixar < 24 horas
├─ HIGH: Fixar < 1 semana
├─ MEDIUM: Fixar < 30 dias
└─ LOW: Fixar em próximo release
```

### 6. Mean Time Between Failures (MTBF)

Mede estabilidade em produção.

```
MTBF = Uptime / Número de Falhas

Exemplo:
├─ Uptime: 29 dias
├─ Falhas: 2
├─ MTBF: (29 × 24h) / 2 = 348 horas = ~14.5 dias
├─ Significado: Em média, 1 falha a cada 14.5 dias

Categorias:
├─ > 1 ano: Excepcional ✅
├─ 6-12 meses: Excelente ✅
├─ 1-6 meses: Bom ✓
├─ 1-4 semanas: Aceitável ⚠️
└─ < 1 semana: Pobre ❌

SLA típicos:
├─ 99.9% (2.75h downtime/mês) → MTBF ~30 dias
├─ 99.99% (4m downtime/mês) → MTBF ~300 dias
└─ 99.999% (26s downtime/mês) → MTBF ~3000 dias
```

### 7. Test Execution Metrics

Mede performance da suíte de testes.

```
Métricas:
├─ Total test time: Tempo para rodar todos testes
├─ Test pass rate: % de testes que passam
├─ Test flakiness: % de testes que falham aleatoriamente
├─ Test coverage by priority: Cobertura por criticidade
└─ Test maintenance: Tempo gasto mantendo testes

Exemplo:
┌───────────────────────────────────┐
│ Test Execution Report             │
├───────────────────────────────────┤
│ Total tests: 1450                 │
│ ├─ Unit: 1000 (14s)              │
│ ├─ Integration: 350 (25s)        │
│ └─ E2E: 100 (300s)               │
│                                   │
│ Total time: 5m 20s                │
│ Pass rate: 99.3% (1436/1450)      │
│ Flaky tests: 2 (0.1%)             │
│ Coverage:                         │
│ ├─ Critical paths: 95%            │
│ ├─ Happy paths: 90%               │
│ └─ Edge cases: 75%                │
│                                   │
│ Trend: Tempo -10%, Coverage +5%   │
└───────────────────────────────────┘
```

---

## Dashboard de Quality Metrics

```
┌──────────────────────────────────────────────────────┐
│       Quality Metrics Dashboard - Últimos 30 dias    │
├──────────────────────────────────────────────────────┤
│                                                      │
│ Test Coverage:           82%          BOM ✓         │
│ Target: 80%              Trend: ↑ +2%               │
│ Critical files: 95%                                 │
│ New code: 88%                                       │
│                                                      │
│ Defect Leakage:          12%          ACEITÁVEL ⚠️  │
│ Target: < 10%            Trend: ↓ -3% (melhorando) │
│ Bugs em produção: 6                                 │
│ Bugs encontrados: 44                                │
│                                                      │
│ Bugs Density:            2/1k LOC     BOM ✓         │
│ Target: < 2.5            Trend: ↓ -0.5             │
│ Total bugs/ano: 85                                  │
│ Lines of code: 42,000                               │
│                                                      │
│ Code Quality:            Grade A      EXCELENTE ✅  │
│ Code Smells: 8 (minor)   Trend: ↓ -20%             │
│ Vulnerabilidades: 0      Critical ✅                │
│ Duplicação: 2%                                      │
│                                                      │
│ Security Issues:         0 CRITICAL   SEGURO ✅    │
│ HIGH: 1 (em progresso)   MEDIUM: 3                 │
│ SLA de fixes: 95%                                   │
│                                                      │
│ MTBF:                    45 dias      BOM ✓         │
│ Target: > 30 dias        Failures: 0 este mês      │
│ Uptime: 99.97%                                      │
│                                                      │
└──────────────────────────────────────────────────────┘

Status Geral: 🟢 QUALIDADE ÓTIMA
```

---

## Correlação entre Métricas

### Coverage vs Defect Leakage

```
Research mostra correlação forte:

Coverage 95%+ → Leakage < 5% ✅ IDEAL
├─ Bugs encontrados principalmente em QA
├─ Poucos escapat para produção
└─ Exemplo: Google, Netflix

Coverage 80-90% → Leakage 8-12% ✓ BOM
├─ Maioria dos bugs encontrados
├─ Alguns bugs escapam
└─ Exemplo: SaaS típico

Coverage 70-80% → Leakage 15-25% ⚠️ RISCO
├─ Muitos bugs escapam para produção
├─ Precisão em testes necessária
└─ Exemplo: Startups em crescimento

Coverage < 70% → Leakage > 30% ❌ CRÍTICO
├─ Confiança baixa
├─ Produção instável
└─ Exemplo: Código legado
```

### Code Quality vs MTTR

```
Código complexo = MTTR alto

Grade A/B → MTTR < 1h ✅
├─ Debugging rápido
├─ Problemas claros
└─ Logs úteis

Grade C → MTTR 1-3h ⚠️
├─ Mais tempo procurando
├─ Alguns problemas obscuros
└─ Refatoração necessária

Grade D/E → MTTR > 3h ❌
├─ Muito tempo debugando
├─ Código confuso
└─ Critério: Parar e refatorar
```

---

## Fixando Métricas de Qualidade Ruins

### Cenário: Coverage Baixa (50%)

```
Problema: Coverage 50%, meta 80%

Diagnóstico:
├─ Código novo sem testes
├─ Código antigo não coberto
├─ Testes mantêm old código vivo?

Solução:

Fase 1 (2 sprints): Baseline
├─ ✅ Medir coverage por arquivo
├─ ✅ Identificar arquivos críticos
├─ ✅ Meta: Arquivos críticos 80%+
└─ Coverage esperada: 50% → 60%

Fase 2 (2-3 sprints): Clean sweep
├─ ✅ Unit tests para novos commits
├─ ✅ Excluir código morto
├─ ✅ Test coverage gates (bloqueia merge < 80%)
└─ Coverage esperada: 60% → 75%

Fase 3 (4-6 sprints): Completar
├─ ✅ E2E tests para journeys críticas
├─ ✅ Integration tests completar
├─ ✅ Code review checklist: "tests?"
└─ Coverage esperada: 75% → 85%+

Timeline: ~3 meses para 85%
```

### Cenário: Defect Leakage Alta (25%)

```
Problema: 25% dos bugs escapam para produção

Diagnóstico:
├─ Coverage baixa não detecta bugs
├─ Test quality ruim (não detecta real issues)
├─ QA manual insuficiente
├─ Deploy sem validação

Solução:

Ação 1: Aumentar Coverage (ref: 02 - Unit Testing)
├─ Impact: -5-10% leakage
├─ Timeline: 1-2 meses

Ação 2: Melhorar Test Quality (ref: 04 - E2E Strategy)
├─ Remover testes flakey
├─ Melhorar assertions
├─ Impact: -5-10% leakage
├─ Timeline: 2-3 semanas

Ação 3: Test Pipeline + PR Testing (ref: 06 - CI/CD)
├─ Bloquear merge sem testes passando
├─ Revisar cobertura antes de merge
├─ Impact: -5-10% leakage
├─ Timeline: 1 mês

Ação 4: E2E Smoke tests em prod (ref: 04 - E2E)
├─ Validar happy path antes de release
├─ Impact: -2-3% leakage (mitiga resto)
├─ Timeline: 1-2 semanas

Resultado esperado: De 25% para 5-8% (ELITE)
Timeline: ~3 meses
```

---

## Monitoramento Contínuo

### Alerts Recomendados

```
Alert: Coverage baixa em novo commit
├─ Trigger: Coverage < base - 2%
├─ Action: Comentário na PR alertando
└─ Severity: ⚠️ Warning

Alert: Vulnerabilidade CRITICAL detectada
├─ Trigger: Vulnerability score CRITICAL
├─ Action: Notificação Slack imediata
├─ SLA: 24 horas para fix
└─ Severity: 🚨 Critical

Alert: MTBF degradando
├─ Trigger: MTBF < 7 dias (alerta) ou < 1 dia (crítico)
├─ Action: Postmortem necessário
└─ Severity: 🔴 High

Alert: Code Smell acumulando
├─ Trigger: Code Smell score +20% vs último mês
├─ Action: Refactoring sprint necessária
└─ Severity: ⚠️ Warning
```

### Relatório Semanal

```
Subject: Weekly Quality Report

Hi Team,

Quality Metrics - Week of Feb 20-26:

✅ GOOD:
- Coverage: 82% (up 2% from last week)
- Security: 0 CRITICAL issues
- MTBF: 60 days

⚠️ ATTENTION:
- Defect Leakage: 15% (was 12% last week - investigate)
- Code Smells: +5 new issues
- Flaky test: payment.test.js (50% pass rate)

🚨 URGENT:
- HIGH security vulnerability in lodash (upgrade to 4.17.21)

Next Steps:
1. Review cause of leakage increase
2. Remove flaky tests from CI until fixed
3. Update lodash dependency

Dashboard: [link]
```

---

## Boas Práticas

### 1. Não Apenas Medir, Interpretar

```
❌ ERRADO:
"Coverage é 82%"
(Sem contexto)

✅ CORRETO:
"Coverage é 82%, subiu de 79% em janeiro. Crítico é 95% (ok), novos commits têm 88% (obrigatório 80%, está ok)."
```

### 2. Balancear Métricas

```
❌ ERRADO:
Focar apenas em coverage

✅ CORRETO:
├─ Coverage (quantidade de código testado)
├─ Defect Leakage (qualidade real dos testes)
├─ Code Quality (manutenibilidade)
├─ Security (proteção)
└─ Performance (MTBF - estabilidade)
```

### 3. Contextualizar por Projeto

```
Startup MVP:
├─ Coverage: 70-80% (rápido crescimento)
├─ Leakage: 15-20% (aceitável)
└─ Code Quality: Grade B-C (prototipagem)

SaaS maduro:
├─ Coverage: 85-95% (confiança)
├─ Leakage: 5-10% (qualidade alta)
└─ Code Quality: Grade A (sustentável)

Sistema crítico (healthcare, fintech):
├─ Coverage: 95%+ (rigidez total)
├─ Leakage: < 2% (quase zero bugs)
└─ Code Quality: Grade A+ (perfeição)
```

### 4. Usar para Guiar, não Punir

```
❌ ERRADO:
"Coverage deve ser 100% ou PR não passa"

✅ CORRETO:
"Coverage deve ser 80% ou mais. Se < 80%, comentamos e ajudamos a aumentar."
```

---

## Conexões com Outras Pastas

Veja como Quality Metrics conecta com:
- **02 - Unit Testing & Test Coverage:** Métricas de cobertura
- **03 - Integration Testing:** Defect leakage reduction
- **04 - E2E Testing:** Product quality validation
- **05 - Test Strategy:** Quality gates enforcement
- **06 - CI/CD:** Métrica automation
- **07 - DORA Metrics:** CFR redução through quality

---

## Conclusão

Quality Metrics transformam qualidade de conceito vago para ciência objetiva. Ao monitorar Coverage, Leakage, Code Quality, e Security, guiamos melhorias contínuas que resultam em:

- ✅ Código mais confiável
- ✅ Produção mais estável
- ✅ Desenvolvimento mais rápido
- ✅ Equipe mais satisfeita

Combine com DORA Metrics para visão 360° de saúde organizacional.
