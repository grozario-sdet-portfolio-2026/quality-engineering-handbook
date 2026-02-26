# Quality Gates

## Introdução

Um **Quality Gate** é um checkpoint automatizado que valida se o código atende aos critérios de qualidade definidos pelo time antes de ser integrado ao repositório principal ou enviado para ambientes de deploy.

Diferente de verificações manuais que dependem de vigilância humana, quality gates são:

- 🤖 **Automatizados** – Executam sem intervenção
- 📏 **Objetivos** – Critérios claros e mensuráveis
- ⚡ **Imediatos** – Feedback instantâneo
- 🚪 **Bloqueadores** – Impedem avanço se falharem

### Analogia

Um quality gate é como um **sistema de segurança de porta**:

```
Sem quality gate:
Dev ➜ pressiona botão ➜ QUALQUER COISA entra ➜ Chaos

Com quality gate:
Dev ➜ tira foto ➜ Reconhecimento facial ➜ Se aprova ➜ Entra
                                           Se nega ➜ Bloqueado
```

---

## Por Que Quality Gates?

### Problemas que Resolvem

#### Sem Quality Gates:

- 🔴 Código com bugs passa para produção
- 🔴 Testes incompletos ou falhando
- 🔴 Vulnerabilidades de segurança ignoradas
- 🔴 Padrões de código inconsistentes
- 🔴 Surpresas em deploy

#### Com Quality Gates:

- 🟢 Apenas código validado passa
- 🟢 Testes obrigatórios
- 🟢 Segurança verificada automaticamente
- 🟢 Padrões consistentes
- 🟢 Previsibilidade aumentada

### Impacto em Números

```
Métrica                     Sem QG    Com QG    Redução
──────────────────────────────────────────────────────
Bugs em produção/mês        45        8         82% ↓
Incidents críticos/mês      12        2         83% ↓
Tempo de retrabalho         18h/week  2h/week   89% ↓
Deploy rollbacks/ano        24        3         87% ↓
Lead time                   10 dias   4 dias    60% ↓
Team burnout                HIGH      LOW       -
```

---

## Princípios de Quality Gates

### 1. Automação Total

Quality gates não devem depender de pessoas.

```
❌ "Por favor, verifique se os testes passaram"
✅ Tests passam? SIM → continua. NÃO → bloqueia
```

### 2. Fail-Fast

Detectar problemas o mais cedo possível.

```
Ordem de execução ideal:
1. Linter (mais rápido, 5 segundos)
2. Testes unitários (15 segundos)
3. Testes integração (2 minutos)
4. SonarQube (1 minuto)

Se falhar em qualquer etapa ➜ PARA. Não continua.
```

### 3. Bloqueador Intransigente

Não há "mas" ou "por favor". Ou passa ou não passa.

```
Quality Gate Check:
✅ Testes: PASSED
✅ Cobertura: 82% (meta: 80%)
❌ Linter: 3 ERRORS (meta: 0)

Resultado: BLOQUEADO até corrigir linter
```

### 4. Critérios Claros

Antes de rodar, everyone deve entender o que é esperado.

```
Quality Gate "Prod Release"
├── Bugs detectados: MÁXIMO 0
├── Vulnerabilidades críticas: MÁXIMO 0
├── Cobertura de testes: MÍNIMO 80%
├── Code smell: MÁXIMO 5
└── Duplicação: MÁXIMO 5%
```

---

## Arquitetura de Quality Gates

### Pipeline em Camadas

```
Developer lança PR
        │
        ▼
┌──────────────────────────────────────────┐
│  CAMADA 1: Validação Estática (5 seg)   │
│                                          │
│  ✓ Linter (ESLint, Prettier, etc)      │
│  ✓ Type checking (TypeScript)          │
└──────────────────────┬───────────────────┘
                       │
        ┌──────────────┴──────────────┐
        │                             │
      PASSA                         FALHA
        │                             │
        ▼                             ▼
┌──────────────────────────────────────┐  ┌──────────────────────┐
│ CAMADA 2: Testes (45 seg)            │  │ ❌ PIPELINE ABORTADA │
│                                      │  │                      │
│ ✓ Testes unitários                   │  │ Dev precisa corrigir │
│ ✓ Testes de integração               │  │ o linter e reenviar  │
└──────────────────────┬───────────────┘  └──────────────────────┘
                       │
        ┌──────────────┴──────────────┐
        │                             │
      PASSA                         FALHA
        │                             │
        ▼                             ▼
┌──────────────────────────────────────┐  ┌──────────────────────┐
│  CAMADA 3: Análise Código (1 min)    │  │ ❌ PIPELINE ABORTADA │
│                                      │  │                      │
│  ✓ SonarQube analysis                │  │ Testes falharam!     │
│  ✓ Quality gates                     │  │ Dev precisa corrigir │
│  ✓ Security scanning                 │  │ os testes e reenviar │
└──────────────────────┬───────────────┘  └──────────────────────┘
                       │
        ┌──────────────┴──────────────┐
        │                             │
      PASSOU                        FALHOU
        │                             │
        ▼                             ▼
┌──────────────────────────────────────┐  ┌──────────────────────┐
│  ✅ APROVADO PARA MERGE              │  │ ❌ PIPELINE ABORTADA │
│                                      │  │                      │
│  PR pode fazer merge!                │  │ SonarQube falhou!    │
│  Segue para próxima etapa            │  │ Quality gate violado │
└──────────────────────────────────────┘  │ Dev deve corrigir e  │
                                          │ reenviar             │
                                          └──────────────────────┘
```

---

## Types of Quality Gates

### 1. Quality Gates de Desenvolvimento (Local)

Executam **antes** de fazer push.

**Configuração:**
```json
{
  "husky": {
    "hooks": {
      "pre-commit": "npm run lint && npm run test:unit"
    }
  }
}
```

**Comportamento:**
```bash
$ git commit -m "Add new feature"

🔍 Running linter...
✅ Lint passed

🧪 Running unit tests...
✅ Tests passed (12/12)

✅ Commit allowed

$ git push
```

**Se falhar:**
```bash
$ git commit -m "Add new feature"

🔍 Running linter...
❌ Lint failed with 3 errors

❌ COMMIT BLOQUEADO
Você PRECISA corrigir os erros antes de fazer commit
```

### 2. Quality Gates de Pull Request

Executam **no CI/CD** quando PR é aberto.

**Configuração:**
```yaml
name: PR Quality Gates

on: [pull_request]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm run lint
      
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm run test
      - run: npm run coverage
      
  sonarqube:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: sonar-scanner

  # Status checks require all jobs to pass
  # If ANY job fails, PR cannot be merged
```

**Resultado no GitHub:**

```
✅ lint (passed)
✅ test (passed)
❌ sonarqube (failed - Quality gate violated)

Blocked: This branch has 1 failing check
└─ Resolve all conversations to enable auto-merge
```

### 3. Quality Gates de Deploy

Executam **antes de deploy** em ambientes.

```
Atividade em TST → Passed all tests → Pronta para PROD
                    ↓
              Quality Gate Check:
              - Bugs: 0? ✅
              - Vulnerabilities: 0? ✅
              - Coverage: 80%? ✅
              - Downtime SLA: OK? ✅
                    ↓
              ✅ Deploy para PROD
```

---

## O Que Cada Quality Gate Valida

### Quality Gate: Linting

**O que valida:**
- Estilo de código
- Erros óbvios
- Padrões de código

**Critérios:**
```
Linter Errors:   MÁXIMO 0
Linter Warnings: MÁXIMO 5
```

**Se falhar:**

```
❌ LINTER FAILED

Errors:
  src/index.js:12:5  - 'unusedVariable' is declared but never used (no-unused-vars)
  src/utils.js:8:1   - Unexpected var, use let or const instead (no-var)

Warnings:
  src/api.js:45:3    - Unexpected console statement (no-console)

🚫 PIPELINE ABORTADA

Ação necessária:
1. Dev corrige os 2 erros
2. Dev roda linter localmente: npm run lint
3. Dev faz commit com as correções
4. Dev faz push novamente
5. Pipeline roda novamente

Até corrigir, NINGUÉM consegue fazer merge dessa branch!
```

### Quality Gate: Testes Automatizados

**O que valida:**
- Código tem testes
- Testes cobrem lógica
- Testes passam

**Critérios:**
```
Cobertura:        MÍNIMO 80%
Testes falhando:  MÁXIMO 0
Flaky tests:      MÁXIMO 0
```

**Se falhar:**

```
❌ TEST FAILED

Failed tests (3):
  ✗ calculateDiscount should apply 15% for premium users
  ✗ validateEmail should reject invalid domain
  ✗ getUserOrders should handle empty results

Coverage:
  Lines:       68% (meta: 80%) ❌
  Branches:    55% (meta: 80%) ❌

🚫 PIPELINE ABORTADA

Ação necessária:
1. Dev executa localmente: npm run test
2. Dev identifica por que testes falharam
3. Dev corrige o código ou os testes
4. Dev verifica cobertura: npm run coverage
5. Dev faz commit e push com correções
6. Pipeline roda novamente

Até passar nos testes E atingir 80% cobertura, 
NINGUÉM consegue fazer merge!
```

### Quality Gate: SonarQube (Análise de Código)

**O que valida:**
- Bugs detectados
- Vulnerabilidades de segurança
- Code smells
- Duplicação
- Complexidade

**Critérios:**
```
Bugs bloqueadores:        MÁXIMO 0
Vulnerabilidades críticas: MÁXIMO 0
Vulnerabilidades altas:    MÁXIMO 0
Security hotspots:        MÁXIMO 3
Code smells:              MÁXIMO 10
Duplicação:               MÁXIMO 5%
```

**Se falhar:**

```
❌ SONARQUBE QUALITY GATE FAILED

Issues encontrados:
  🔴 BLOCKER (1): SQL Injection vulnerability in getUserData function
  🔴 CRITICAL (1): Hardcoded password in config.js
  🟠 HIGH (2): Duplicate code blocks (5% duplicação)
  🟡 MEDIUM (8): Complex functions (cyclomatic complexity > 15)

🚫 PIPELINE ABORTADA

Ação necessária:
1. Dev acessa SonarQube dashboard
2. Dev revisa cada issue relatado
3. Dev corrige vulnerabilidades
4. Dev refatora código complexo
5. Dev roda análise local: sonar-scanner
6. Dev faz commit com correções
7. Dev faz push
8. Pipeline roda SonarQube novamente

Até resolver TODAS as issues, 
NINGUÉM consegue fazer merge!
```

---

## O Que Acontece Com Falhas: Fluxo Completo

### Cenário Real: PR é Bloqueada

```
TIMELINE:
────────────────────────────────────────────────────────────

T0 → Dev implementa feature: "Add user search"
     $ git push origin feature/user-search
     
T1 → GitHub dispara CI/CD pipeline
     📋 Workflow initiated
     
T2 → Linter roda (5 seg)
     ✅ Linter PASSED
     
T3 → Testes unitários rodam (15 seg)
     ✅ Testes PASSED (45/45)
     
T4 → Cobertura validada (5 seg)
     ✅ Coverage: 82% (meta: 80%)
     
T5 → SonarQube análise (1 min)
     ⏳ Analisando...
     
T6 → SonarQube resultado
     ❌ Quality Gate FALHOU
     Motivo: Complexity demasiada (score 45, max 20)
     
T7 → Pipeline para imediatamente
     🚫 ALL CHECKS MUST PASS
     
     GitHub PR status:
     └─ ❌ sonarqube-gate (failed)
        └─ This branch has 1 failing check
           └─ Cannot be merged until this is resolved
     
T8 → Dev recebe notificação
     📧 Email: "PR #123 - Checks failed"
     🔔 GitHub notification: "1 check failed"
     
T9 → Dev toma ação
     Dev analisa SonarQube report
     Dev identifica: função getUserData muito complexa
     Dev refatora função
     Dev testa localmente
     
T10 → Dev faz novo commit
      $ git add src/userService.js
      $ git commit -m "Refactor getUserData - reduce complexity"
      $ git push
      
T11 → Pipeline roda NOVAMENTE
      ✅ Linter PASSED
      ✅ Testes PASSED
      ✅ Coverage OK
      ✅ SonarQube PASSED (complexity: 18)
      
T12 → PR Status muda
      ✅ All checks passed
      └─ ✅ Ready to merge
      
T13 → Dev (ou Tech Lead) faz merge
      $ Merge branch 'feature/user-search' into 'main'
      
────────────────────────────────────────────────────────────

Lição:
Dev NÃO conseguiu fazer merge até resolver o problema.
A pipeline foi bloqueante e não negociável.
```

### Rejeição em Cascata: Múltiplas Falhas

```
Pior caso scenario:

$ git push origin feature/complex-feature

Pipeline rodando...

❌ FALHA 1: Linter
   ✗ 5 console.log() statements found (no-console rule)
   ✗ 2 unused variables
   
   → PIPELINE ABORTADA AQUI
   
   Dev corrige linter, faz push novamente...
   
Pipeline rodando...

❌ FALHA 2: Testes
   ✗ 3 testes falhando
   ✗ Coverage caiu para 65% (meta: 80%)
   
   → PIPELINE ABORTADA AQUI
   
   Dev corrige testes, faz push novamente...
   
Pipeline rodando...

❌ FALHA 3: SonarQube
   ✗ SQL Injection vulnerability detectada
   ✗ 15% código duplicado
   ✗ Complexity muito alta
   
   → PIPELINE ABORTADA AQUI
   
   Dev refatora, faz push novamente...
   
Pipeline rodando...

✅ TUDO PASSOU

Tempo total: ~2 horas
Lições aprendidas: MUITAS

O código só merge depois de estar realmente pronto!
```

---

## Configurando Quality Gates em Diferentes Plataformas

### GitLab Quality Gates

```yaml
stages:
  - lint
  - test
  - sonarqube

lint:
  stage: lint
  script:
    - npm run lint
  allow_failure: false  # ← Bloqueante!

test:
  stage: test
  script:
    - npm run test
    - npm run coverage
  coverage: '/Coverage: \d+\.\d+%/'
  allow_failure: false  # ← Bloqueante!

sonarqube:
  stage: sonarqube
  script:
    - sonar-scanner
  allow_failure: false  # ← Bloqueante!
  only:
    - merge_requests

# Merge bloqueada se algum job falhar
```

### GitHub Status Checks

```yaml
name: Quality Gates

on: [pull_request]

jobs:
  check-lint:
    runs-on: ubuntu-latest
    name: "Lint Check"
    steps:
      - uses: actions/checkout@v3
      - run: npm run lint
        # Se falhar o step, job falha automaticamente
        
  check-tests:
    runs-on: ubuntu-latest
    name: "Test Check"
    steps:
      - uses: actions/checkout@v3
      - run: npm run test
      - run: npm run coverage
      
  check-sonar:
    runs-on: ubuntu-latest
    name: "SonarQube Check"
    steps:
      - uses: actions/checkout@v3
      - run: sonar-scanner

# No branch protection rules:
# - Require status checks to pass before merging
# - Require all of: Lint Check, Test Check, SonarQube Check
```

**Resultado:** PR não pode fazer merge se qualquer check falhar.

---

## Quality Gates por Ambiente

### DEV Environment Quality Gate

```
Atividade entra em DEV
        ↓
┌──────────────────────────────────┐
│  Stack de Testes Completa       │
│  - Linter: DEVE PASSAR          │
│  - Testes: DEVE PASSAR          │
│  - Coverage ≥ 80%: DEVE PASSAR  │
│  - SonarQube: DEVE PASSAR       │
└──────────────┬───────────────────┘
               │
       ┌───────┴────────┐
       │                │
     PASSOU           FALHOU
       │                │
       ▼                ▼
  Continua para    ❌ BLOQUEADA
  Cross Validation Dev recebe
                   Erro report
                   Dev corrige
                   Push novamente
```

### TST Environment Quality Gate

```
Atividade vai de DEV para TST
        ↓
┌──────────────────────────────────┐
│  Stack de Testes Completa       │
│  (Mesmo que em DEV)             │
│  - Testes: DEVE PASSAR          │
│  - SonarQube: DEVE PASSAR       │
└──────────────┬───────────────────┘
               │
       ┌───────┴────────┐
       │                │
     PASSOU           FALHOU
       │                │
       ▼                ▼
  Liberado para    ❌ BLOQUEADA
  Produção        Retorna
  Deploy pode      para DEV para
  acontecer        correção
```

### PROD Environment Quality Gate

```
Deploy para Produção
        ↓
┌──────────────────────────────────┐
│  Final Quality Gate Checks       │
│  - 0 bugs críticos               │
│  - 0 vulnerabilidades críticas   │
│  - Observabilidade OK            │
│  - Rollback plan definido        │
│  - Communication done            │
└──────────────┬───────────────────┘
               │
       ┌───────┴────────┐
       │                │
     PASSOU           FALHOU
       │                │
       ▼                ▼
  Deploy para      ❌ DEPLOY
  PRODUÇÃO         CANCELADO
  ✅ Go Live!     "Algo não está
                   pronto"
```

---

## Mensagens de Falha: Exemplos

### Mensagem Padrão de Linter

```
══════════════════════════════════════════════════════════════
❌ LINTER CHECK FAILED - BLOCKED FROM MERGE
══════════════════════════════════════════════════════════════

Arquivo: src/components/UserForm.js

Erros encontrados (2):
  Line 12:5    error     'userData' is assigned but never used  (no-unused-vars)
  Line 45:10   error     Unexpected var, use let or const       (no-var)

Advertências (1):
  Line 78:3    warning   Unexpected console statement          (no-console)

⏹️  AÇÃO REQUERIDA:
1. Corrija os erros acima
2. Execute: npm run lint --fix
3. Faça novo commit e push
4. Pipeline será acionada novamente

🔗 Documentação: https://docs.seu-projeto.com/linter-setup

Até resolver TODOS os erros, seu código NÃO pode fazer merge!
══════════════════════════════════════════════════════════════
```

### Mensagem Padrão de Teste Falhando

```
══════════════════════════════════════════════════════════════
❌ TESTS FAILED - BLOCKED FROM MERGE
══════════════════════════════════════════════════════════════

Testes que falharam (2):
  ✗ calculateDiscount
    └─ Should apply 15% for premium users
       Expected: 15
       Received: 20
       
  ✗ validateUserEmail
    └─ Should reject emails without domain
       TypeError: Cannot read property 'domain' of undefined

Cobertura:
  Linhas:   68.5% (meta: 80%) ❌ ABAIXO DA META
  Branches: 62% (meta: 80%) ❌ ABAIXO DA META

⏹️  AÇÃO REQUERIDA:
1. Rode testes localmente: npm run test
2. Corrija os testes falhando
3. Aumente cobertura para 80%+
4. Verifique: npm run coverage
5. Faça novo commit e push

Até passar em TODOS os testes E atingir 80% cobertura,
seu código NÃO pode fazer merge!

🔗 Documentação: https://docs.seu-projeto.com/testing
══════════════════════════════════════════════════════════════
```

### Mensagem Padrão de SonarQube Gate Falhando

```
══════════════════════════════════════════════════════════════
❌ SONARQUBE QUALITY GATE FAILED - BLOCKED FROM MERGE
══════════════════════════════════════════════════════════════

Issues Críticos:
  🔴 BLOCKER (1):
     SQL Injection Risk in getUserData function
     File: src/services/userService.js:45
     Motivo: Query sem parameterized statement
     
  🔴 CRITICAL (1):
     Hardcoded API key detected
     File: src/config.js:12
     Motivo: Secret exposto no código

Issues Altos:
  🟠 HIGH (3):
     - Duplicate code (22% duplicação)
     - Function too complex (complexity: 28, max: 15)
     
⏹️  AÇÃO REQUERIDA:
1. Acesse: https://sonarqube.seu-projeto.com
2. Revise cada issue reportada
3. Corrija vulnerabilidades de segurança
4. Refatore código complexo
5. Execute: sonar-scanner
6. Faça novo commit e push

Até resolver TODOS os blockers e críticos,
seu código NÃO pode fazer merge!

🔗 SonarQube Report: [link-para-análise]
🔗 Documentação: https://docs.seu-projeto.com/sonarqube
══════════════════════════════════════════════════════════════
```

---

## Impacto de Quality Gates

### Antes

```
Semana típica SEM quality gates:
├─ Segunda: 20 PRs abertas
├─ Terça: 18 merged, 2 com bugs encontrados em DEV
├─ Quarta: 5 hotfixes urgentes em produção
├─ Quinta: 12 PRs abertas, 2 rolledback
├─ Sexta: P1 incident em produção (2 usuários afetados)
└─ Sábado: On-call respondendo issues

Bugs em produção: 47 por mês
Incidents: 8 por mês
Team stress: MUITO ALTO
```

### Depois

```
Semana típica COM quality gates:
├─ Segunda: 20 PRs abertas
├─ Terça: 3 precisaram re-work (linter/testes/SonarQube)
│         17 merged sem problemas
├─ Quarta: 0 bugs encontrados em DEV
├─ Quinta: 19 PRs opened, todas passando QG
├─ Sexta: 0 incidents em produção
└─ Sábado: Tranquilo, ninguém chamado

Bugs em produção: 2 por mês (95% redução!)
Incidents: 0 por mês
Team stress: MUITO MENOR
Confiança: MUITO MAIOR
```

---

## Boas Práticas para Quality Gates

### 1. Configure Realista, Não Perfeito

**Ruim:**
```
Cobertura: 100% (impossível)
Bugs: 0 em codebase existente (irreal)
```

**Bom:**
```
Cobertura: ≥ 80% (atingível)
Bugs: 0 (para novo código)
Histórico de bugs: < 5 por 1000 LOC (atingível)
```

### 2. Fail-Fast: Ordem Importa

**Ruim:**
```
1. Análise SonarQube (2 min)
2. Testes (15 seg)
3. Linter (5 seg)
```

**Bom:**
```
1. Linter (5 seg) ← Falha rápido
2. Testes (15 seg)
3. SonarQube (2 min) ← Só roda se passou nos anteriores
```

### 3. Seja Transparente

Cada falha deve deixar claro:
- ✅ O que falhou
- ✅ Por quê
- ✅ Como corrigir
- ✅ Onde procurar mais info

### 4. Não Use Como Puniçao

Quality gates são **guardrails**, não puniçao.

```
❌ Atitude errada:
"Você violou o quality gate!"
"Seu código é ruim!"

✅ Atitude certa:
"Vamos ajudar você a passar no QG"
"Qual parte está difícil?"
```

### 5. Evolua Gradualmente

Mês 1:
```
Cobertura: ≥ 50%
Linter: avisos
SonarQube: informativo
```

Mês 3:
```
Cobertura: ≥ 70%
Linter: erros
SonarQube: bloqueante
```

Mês 6:
```
Cobertura: ≥ 85%
Linter: zero tolerance
SonarQube: standards industriais
```

### 6. Revise Regularmente

A cada sprint, pergunte:
- QG está muito rigoroso?
- QG está muito leve?
- Precisamos de novos critérios?
- Algum critério é obsoleto?

---

## Conclusão

Quality gates são **investimentos em confiiance**:

- 🤖 Automatizados e intransigentes
- 📏 Objetivos e mensuráveis
- ⚡ Feedback imediato
- 🚪 Bloqueadores de qualidade
- 📉 Reduzem bugs dramaticamente

**Princípio fundamental:**

> "Código só entra em repositório e ambientes se passar por todos os quality gates. Sem exceções. Sem paliativos. Sem 'mas por favor'."

Quando quality gates estão bem configuradas, a qualidade não é uma esperança, é uma garantia.
