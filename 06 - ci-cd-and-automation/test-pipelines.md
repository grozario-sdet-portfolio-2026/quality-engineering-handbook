# Test Pipelines

## Origem e Conceito

Test Pipelines são sequências automatizadas de validações que executam entre o momento que um desenvolvedor faz commit até o deploy em produção.

Originado nas práticas de Integração Contínua (CI/CD), o teste pipeline garante que apenas código validado e de qualidade avança através das etapas de desenvolvimento.

**Definição:** Um pipeline de testes é uma cadeia automatizada de verificações (linting, testes unitários, testes de integração, análise de código, testes E2E) que valida a qualidade do código antes de permitir merge ou deploy.

---

## Por Que Test Pipelines?

### Problemas que resolvem:

1. **Código ruim em produção** - Sem validação, bugs chegam
2. **Validação manual lenta** - Humanos não escalam
3. **Inconsistência entre ambientes** - Desenvolvimento vs produção
4. **Débito técnico** - Falta de validação contínua acumula problemas
5. **Deploy arriscado** - Sem confiança no código
6. **Feedback lento** - Desenvolvedor espera horas para saber se quebrou algo
7. **Regressões escondidas** - Mudanças quebram código antigo sem avisar

### Benefícios:

- ✅ Código validado antes de merge
- ✅ Feedback imediato ao desenvolvedor
- ✅ Reduz bugs em produção
- ✅ Aumenta velocidade de deploy
- ✅ Documentação através de testes
- ✅ Confiança na qualidade

### Impacto em Números

```
Métrica                    Sem Pipeline    Com Pipeline    Melhoria
─────────────────────────────────────────────────────────────────
Bugs/mês em produção       ~50             ~5              90% ↓
Lead time para fix         3 dias          2 horas         99% ↓
Deploy frequency           2x/mês          2x/dia          30x ↑
Mean time to recovery      8 horas         30 min          94% ↓
Change fail rate           35%             5%              86% ↓
```

---

## Arquitetura do Pipeline

### Modelo em Camadas

A arquitetura ideal de um pipeline segue a estratégia Shift-Left com fail-fast:

```
Developer faz commit e push
        │
        ▼
┌─────────────────────────────────────────────────────┐
│ STAGE 1: Validação Local (5 segundos)              │
│ - Pre-commit hooks                                  │
│ - Linter (ESLint, Prettier)                         │
│ - Type checking (TypeScript)                        │
└────────────────┬────────────────────────────────────┘
                 │
        ┌────────┴────────┐
        │                 │
     PASSA             FALHA ──► ❌ BLOQUEIA
        │
        ▼
┌─────────────────────────────────────────────────────┐
│ STAGE 2: Testes Rápidos (45 segundos)              │
│ - Unit tests                                        │
│ - Component tests                                   │
│ - Integration tests (rápidos)                       │
└────────────────┬────────────────────────────────────┘
                 │
        ┌────────┴────────┐
        │                 │
     PASSA             FALHA ──► ❌ BLOQUEIA
        │
        ▼
┌─────────────────────────────────────────────────────┐
│ STAGE 3: Análise de Código (1 minuto)              │
│ - Code coverage check                               │
│ - SonarQube / Code quality                          │
│ - Security scanning (SAST)                          │
│ - Dependency check                                  │
└────────────────┬────────────────────────────────────┘
                 │
        ┌────────┴────────┐
        │                 │
     PASSA             FALHA ──► ❌ BLOQUEIA
        │
        ▼
┌─────────────────────────────────────────────────────┐
│ STAGE 4: Testes E2E (3-5 minutos)                  │
│ - End-to-end tests                                  │
│ - Smoke tests                                       │
│ - Critical user journeys                            │
└────────────────┬────────────────────────────────────┘
                 │
        ┌────────┴────────┐
        │                 │
     PASSA             FALHA ──► ❌ BLOQUEIA
        │
        ▼
   ✅ APROVADO
   PR pode fazer merge!
```

---

## Etapas do Pipeline

### STAGE 1: Validação Estática (Fail-Fast)

Executa **antes** do desenvolvedor fazer commit/push.

**Objetivos:**
- Catch erros óbvios rapidamente
- Feedback instantâneo
- Evitar envio de código ruim

**Verificações:**

```javascript
// ESLint - Padrões de código
// ✅ PASS
const user = { name: 'John', age: 30 };

// ❌ FAIL: variável não usada
const unused = 5;

// Prettier - Formatação
// ✅ PASS: código formatado
const status = condition ? 'yes' : 'no';

// ❌ FAIL: código mal formatado
const status=condition?'yes':'no';

// TypeScript - Type checking
// ✅ PASS: tipos corretos
function add(a: number, b: number): number {
  return a + b;
}
add(1, 2); // OK

// ❌ FAIL: tipo errado
add('1', '2'); // Error: string não é number
```

**Implementação:**

```json
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged",
      "pre-push": "npm run type-check && npm run lint"
    }
  },
  "lint-staged": {
    "*.{js,ts}": ["eslint --fix", "prettier --write"],
    "*.ts": "tsc --noEmit"
  }
}
```

---

### STAGE 2: Testes Automatizados (Rápidos)

Executa **com cada push** para branch.

**Objetivos:**
- Validar lógica da aplicação
- Garantir regressões não ocorrem
- Feedback em minutos, não horas

**Verificações:**

```bash
# Testes unitários (rápido, sem I/O)
npm run test:unit
# ✅ 150 testes em 15s

# Testes de componente (renderização)
npm run test:component
# ✅ 45 testes em 20s

# Testes de integração (com BD real)
npm run test:integration
# ✅ 35 testes em 10s

# Total: ~45 segundos
```

**Cobertura obrigatória:**

```yaml
---
STAGE: Test Coverage
Minimo_Obrigatorio:
  line_coverage: 80%
  branch_coverage: 75%
  function_coverage: 85%

Verificacao:
  if coverage < minimo:
    result: ❌ FALHA
  else:
    result: ✅ PASSA
```

---

### STAGE 3: Análise de Código

Executa **após testes** passarem.

**Objetivos:**
- Detectar code smell
- Vulnerabilidades de segurança
- Duplicação de código
- Débito técnico

**Verificações:**

```javascript
// SonarQube
const analysis = {
  bugs: 0,           // Máximo: 0
  vulnerabilities: 0, // Máximo: 0
  code_smells: 5,    // Máximo: 10
  coverage: 82,      // Mínimo: 80
  duplication: 3     // Máximo: 5%
};

// Resultado: ✅ PASSA

// --- Outro exemplo ---
const analysis2 = {
  bugs: 3,
  vulnerabilities: 1
};
// Resultado: ❌ FALHA - bugs > 0
```

**Ferramentas:**
- **SonarQube** - Análise completa de código
- **Snyk** - Vulnerabilidades de dependências
- **SAST** - Static Application Security Testing
- **Checkmarx/Fortify** - Análise de segurança

---

### STAGE 4: Testes E2E (Pronta-Entrega)

Executa **em ambiente de staging**.

**Objetivos:**
- Validar fluxos completos
- Testar integração real
- Confiança para deploy

**Exemplo:**

```javascript
describe('E2E Tests - Pre-deployment', () => {
  beforeAll(async () => {
    await testEnv.start();
    await testEnv.seed(); // Dados de teste
  });

  it('should complete signup flow', async () => {
    await page.goto('https://staging.app.com/signup');
    await page.fill('input[name="email"]', 'test@example.com');
    await page.fill('input[name="password"]', 'Test123@');
    await page.click('button[type="submit"]');
    await page.waitForNavigation();
    expect(page.url()).toContain('/dashboard');
  });

  it('should process payment', async () => {
    await page.goto('https://staging.app.com/checkout');
    await page.fill('input[name="card"]', '4111111111111111');
    await page.click('button:has-text("Pay")');
    expect(await page.textContent('.success')).toContain('Success');
  });

  afterAll(async () => {
    await testEnv.cleanup();
  });
});
```

---

## Triggers do Pipeline

### 1. Trigger Local (Pre-commit)

Executa **antes** do commit ser criado.

```bash
$ git commit -m "Add feature"

🔍 Running pre-commit hooks...
✅ Linter passed
✅ Type check passed

Commit criado e pronto
```

---

### 2. Trigger em Push (CI/CD)

Executa **após push** para branch.

```yaml
name: CI Pipeline
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Install dependencies
        run: npm install
      
      - name: Lint
        run: npm run lint
      
      - name: Unit Tests
        run: npm run test:unit
      
      - name: Coverage
        run: npm run test:coverage
      
      - name: SonarQube
        run: npm run sonar
```

---

### 3. Trigger em Pull Request

Executa **antes** de permitir merge.

```yaml
name: PR Check
on: pull_request

jobs:
  approval:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: All validations
        run: npm run ci:full
      
      - name: Comment PR
        with:
          body: |
            ✅ CI Passed
            Coverage: 82%
            0 Security Issues
```

---

### 4. Trigger em Merge para Main

Executa **antes** de permitir deploy.

```yaml
name: Pre-Deploy
on:
  push:
    branches: [main]

jobs:
  pre-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Full test suite
        run: npm run test:all
      
      - name: E2E tests
        run: npm run test:e2e
      
      - name: Smoke tests
        run: npm run test:smoke
      
      - name: Deploy to staging
        run: npm run deploy:staging
```

---

## Configuração Prática

### GitHub Actions (Exemplo Completo)

```yaml
name: Complete CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  validate:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        node-version: [16.x, 18.x]
    
    steps:
      # Setup
      - uses: actions/checkout@v3
      
      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      
      # Install
      - name: Install dependencies
        run: npm ci
      
      # STAGE 1: Validação Estática
      - name: Lint code
        run: npm run lint
      
      - name: Type check
        run: npm run type-check
      
      # STAGE 2: Testes Rápidos
      - name: Run unit tests
        run: npm run test:unit
      
      - name: Run component tests
        run: npm run test:component
      
      - name: Run integration tests
        run: npm run test:integration
      
      # STAGE 3: Análise
      - name: Check coverage
        run: npm run test:coverage
        env:
          COVERAGE_THRESHOLD: 80
      
      - name: SonarQube analysis
        run: npm run sonar
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      
      - name: Security scanning
        run: npm run security:check
      
      # STAGE 4: E2E (se for main)
      - name: E2E tests (main only)
        if: github.ref == 'refs/heads/main'
        run: npm run test:e2e
      
      # Report
      - name: Upload coverage reports
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
      
      - name: Comment PR
        if: github.event_name == 'pull_request'
        uses: actions/github-script@v6
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: '✅ All CI checks passed!'
            })
```

---

## Boas Práticas

### 1. Fail-Fast

Executar verificações mais rápidas primeiro.

```
Ordem recomendada:
1. Linter (5s)
2. Type check (10s)
3. Unit tests (20s)
4. Integration tests (30s)
5. Coverage (5s)
6. SonarQube (60s)

❌ ERRADO:
E2E (300s) → Linter (5s) → Cobertura (5s)
Precisa esperar 300s para descobrir falta espaço

✅ CERTO:
Linter (5s) → Coverage (5s) → Unit (20s) → E2E (300s)
Feedback rápido em erros óbvios
```

---

### 2. Bloqueadores Intransigentes

Quality gates devem **bloquear**, não avisar.

```javascript
// ❌ ERRADO: apenas avisos
if (coverage < 80) {
  console.warn('⚠️ Coverage baixa, mas continuando...');
}

// ✅ CORRETO: bloqueador
if (coverage < 80) {
  throw new Error('❌ Coverage < 80%. Precisa corrigir.');
  process.exit(1);
}
```

---

### 3. Paralelização

Executar stages independentes em paralelo.

```yaml
jobs:
  # Executam em paralelo
  unit-tests:
    runs-on: ubuntu-latest
    steps: [...]
  
  integration-tests:
    runs-on: ubuntu-latest
    steps: [...]
  
  lint:
    runs-on: ubuntu-latest
    steps: [...]
  
  # Aguarda tudo completar
  merge-check:
    needs: [unit-tests, integration-tests, lint]
    steps:
      - name: All passed
        run: echo "✅ Ready to merge"
```

---

### 4. Caching

Cache de dependências para acelerar pipeline.

```yaml
- name: Cache node modules
  uses: actions/cache@v3
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-

- name: Install dependencies
  run: npm ci
```

---

### 5. Timeouts

Prevenir travamentos indefinidos.

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    timeout-minutes: 30
    
    steps:
      - name: Unit tests
        run: npm run test:unit
        timeout-minutes: 5
      
      - name: E2E tests
        run: npm run test:e2e
        timeout-minutes: 15
```

---

## Métricas e Monitoramento

### Métricas de Pipeline

```javascript
const pipelineMetrics = {
  // Performance
  average_execution_time: '2m 30s',
  max_execution_time: '5m',
  min_execution_time: '1m 20s',
  
  // Confiabilidade
  success_rate: '98%',
  flaky_test_rate: '2%',
  avg_failures_per_run: 0.05,
  
  // Feedback
  avg_feedback_time: '2m 50s',
  developers_blocked_daily: 2,
  
  // Quality
  bugs_caught_before_production: 45,
  escape_rate: '5%',
  
  // Cost
  pipeline_cost_per_month: '$500',
  cost_per_successful_deploy: '$2.50'
};
```

### Dashboard Ideal

```
Pipeline Health Dashboard
═══════════════════════════════════════════

Current Status:
├─ Last 100 runs: 98% success rate ✅
├─ Flaky tests: 2 (marked for fix)
├─ Avg execution: 2m 50s
└─ Quality gate violations: 0

Per Stage:
├─ Lint: 98.5% pass rate (avg 5s)
├─ Unit Tests: 99% pass rate (avg 20s)
├─ Integration: 97% pass rate (avg 30s)
├─ Coverage: 82% average ✅
├─ Security: 0 vulnerabilities ✅
└─ E2E: 96% pass rate (avg 300s)

Trends (last 7 days):
├─ Success rate: ↑ 2% (trending up ✅)
├─ Avg time: ↓ 30s (getting faster ✅)
├─ Bugs escaped: ↓ 8 (fewer bugs ✅)
└─ Cost: stable
```

---

## Troubleshooting

### Pipeline Lento

```
Problema: Pipeline demorando > 10 minutos

Causas comuns:
1. Testes sequenciais (devem ser paralelos)
2. Testes lentos sem mock (BD real)
3. Sem cache de dependências
4. E2E rodando em tudo (só main)

Solução:
✅ Paralelizar stages
✅ Mock dependências lentas
✅ Implementar caching
✅ E2E apenas em main/prod
```

### Testes Flakey

```
Problema: Testes passam e falham aleatoriamente

Causas:
1. Timing issues no E2E
2. Estado compartilhado entre testes
3. Dependências instáveis
4. Dados de teste não isolados

Solução:
✅ Adicionar timeouts adequados
✅ Isolar estado de testes
✅ Mock dependências externas
✅ Resetar BD entre testes
```

### Coverage Baixa

```
Problema: Cobertura caiu < 80%

Causas:
1. Novo código sem testes
2. Dead code não removido
3. Branches não testadas
4. Threshold muito ambicioso

Solução:
✅ Adicionar testes para novo código
✅ Remover dead code
✅ Testar todos os branches
✅ Revisar threshold com time
```

---

## Conclusão

Um pipeline de testes bem estruturado é a base de qualidade contínua. Ao automatizar validações, obtemos:

- ✅ Confiança em deploys
- ✅ Feedback rápido
- ✅ Redução de bugs
- ✅ Escalabilidade
- ✅ Qualidade sustentável

Combine com as estratégias das outras pastas (Pirâmide de Testes, Shift-Left, Definition of Done) para máxima efetividade.
