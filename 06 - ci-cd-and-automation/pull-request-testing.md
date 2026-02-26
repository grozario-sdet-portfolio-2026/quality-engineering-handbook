# Pull Request Testing

## Origem e Conceito

Pull Request Testing (Testes em Pull Requests) refere-se às validações automatizadas que ocorrem quando um desenvolvedor cria uma PR, antes que o código seja mesclado com a branch principal.

Originado nas práticas de Code Review e CI/CD, o teste em PR garante que apenas código validado chega à main branch.

**Definição:** Pull Request Testing é o conjunto de verificações automatizadas que validam qualidade, cobertura, segurança e funcionalidade de código proposto em um Pull Request, bloqueando merge se critérios não forem atendidos.

---

## Por Que Pull Request Testing?

### Problemas que resolvem:

1. **Código ruim em main** - PR sem validação prossegue
2. **Regressões escondidas** - Mudanças quebram código existente
3. **Débito técnico acumula** - Sem vigilância contínua
4. **Code reviews "às cegas"** - Reviewer não vê impacto real
5. **Deploy arriscado** - Main branch comprometida
6. **Comunicação intransigente** - Sem feedback automático
7. **Surpresas tarde na pipeline** - Problemas descobertos em staging/prod

### Benefícios:

- ✅ Validação objetiva antes de merge
- ✅ Reduz carga de code review humano
- ✅ Protege main branch
- ✅ Feedback imediato ao autor
- ✅ Confiança em merge
- ✅ Histórico de qualidade

### Impacto em Números

```
Métrica                           Sem PR Testing    Com PR Testing    Melhoria
──────────────────────────────────────────────────────────────────────────
PRs com defeitos que são mergeados    ~40%             ~2%              95% ↓
Regressões por mês                    ~15              ~1               93% ↓
Tempo gasto em code review/dev        6h/dia           3h/dia           50% ↓
Hotfixes por sprint                   ~8               ~1               87% ↓
Confiança para fazer merge            BAIXA            ALTA             -
```

---

## Pipeline de PR

### Fluxo Completo

```
Developer cria PR
        │
        ▼
┌────────────────────────────────────┐
│ Automático e Imediato              │
├────────────────────────────────────┤
│ 1. Validação Estática (2s)        │
│    - Lint check                    │
│    - Type check                    │
└────────────┬───────────────────────┘
             │
     ┌───────┴──────────┐
     │                  │
   PASSA              FALHA
     │                  │
     ▼                  ▼
┌──────────────────────┐  ┌──────────────────────────┐
│ Continua pipeline    │  │ ❌ Bloqueia merge        │
└────────┬─────────────┘  │ Comentário na PR:        │
         │                │ "Fix linter errors"      │
         ▼                └──────────────────────────┘
┌────────────────────────────────────┐
│ 2. Testes (20s)                   │
│    - Unit tests                    │
│    - Component tests               │
│    - Integration tests             │
└────────────┬───────────────────────┘
             │
     ┌───────┴──────────┐
     │                  │
   PASSA              FALHA ──► ❌ Bloqueia
     │
     ▼
┌────────────────────────────────────┐
│ 3. Análise (30s)                   │
│    - Coverage check                │
│    - Code quality                  │
│    - Security scan                 │
└────────────┬───────────────────────┘
             │
     ┌───────┴──────────┐
     │                  │
   PASSA              FALHA ──► ❌ Bloqueia
     │
     ▼
┌────────────────────────────────────┐
│ ✅ Todas as checks passaram!       │
├────────────────────────────────────┤
│ - Linter: ✅                       │
│ - Tests: 60/60 passed ✅           │
│ - Coverage: 82% ✅                 │
│ - Security: 0 issues ✅            │
│                                    │
│ Ready for Code Review! ✅          │
└────────────────────────────────────┘
         │
         ▼ (Reviewer analisa)
┌────────────────────────────────────┐
│ Code Review (humano)               │
│ - Lógica correta?                  │
│ - Padrões seguidos?                │
│ - Documentação OK?                 │
└────────────┬───────────────────────┘
             │
     ┌───────┴──────────┐
     │                  │
  APROVADO          MUDANÇAS
     │              SOLICITADAS
     ▼              │
  MERGE! ✅         ▼
                Volta desenvolvimento
```

---

## Validações Automáticas

### 1. Linting & Formatting (5 segundos)

Verifica padrões de código e formatação.

**Verificações:**
- Indentação correta
- Imports organizados
- Variáveis não usadas
- Sintaxe correta
- Console.logs removidos

**Exemplo:**

```javascript
// ❌ Falha em linter
const user = "John"    // espaçamento inconsistente
  const age = 30       // indentação errada
import { foo } from 'bar'
import { zed, bar } from 'baz'  // imports não organizados
console.log(user)      // console.log deixado

// ✅ Passa em linter
const user = 'John';
const age = 30;

import { bar, zed } from 'baz';
import { foo } from 'bar';

// sem console.logs
```

**Configuração:**

```json
{
  "scripts": {
    "lint": "eslint . --max-warnings 0",
    "format": "prettier --write ."
  },
  "eslintConfig": {
    "extends": ["airbnb"],
    "rules": {
      "no-unused-vars": "error",
      "no-console": "error"
    }
  }
}
```

**Feedback na PR:**

```
❌ Linter failed
3 errors found:

src/user.ts:15 - Unexpected console statement
src/api.ts:42 - 'unused' is assigned but never used
src/utils.ts:8 - Missing semicolon
```

---

### 2. Type Checking (10 segundos)

Valida tipos (TypeScript, Flow, etc.).

**Verificações:**
- Tipos de parâmetros corretos
- Return types válidos
- Type narrowing
- Generic types corretos

**Exemplo:**

```typescript
// ❌ Falha em type check
function add(a: number, b: number): number {
  return a + b;
}

add('1', '2'); // Error: string não é number
const result: string = add(1, 2); // Error: number não é string

// ✅ Passa em type check
function add(a: number, b: number): number {
  return a + b;
}

add(1, 2); // ✅ OK
const result: number = add(1, 2); // ✅ OK
```

**Feedback na PR:**

```
❌ Type check failed
2 errors found:

src/utils.ts:45:2 - Argument of type 'string' 
  is not assignable to parameter of type 'number'

src/api.ts:20:5 - Type 'number' is not assignable 
  to type 'string'
```

---

### 3. Testes Unitários (15 segundos)

Executa testes rápidos para validar lógica.

**Verificações:**
- Todos os testes passam
- Nenhum teste flakey
- Performance aceitável

**Exemplo:**

```javascript
// Teste que falha
describe('UserService', () => {
  it('should validate email', () => {
    const result = validateEmail('invalid@');
    expect(result).toBe(true); // ❌ Falha esperado: false
  });
});

// PR não pode ser mergeada até passar
```

**Feedback na PR:**

```
❌ Unit tests failed
1 failure:

FAIL src/__tests__/user.test.js
✓ should hash password (5ms)
✗ should validate email (3ms)
  Expected: true
  Received: false

Run tests locally:
npm run test -- --watch
```

---

### 4. Test Coverage (5 segundos)

Verifica se cobertura atende meta.

**Verificações:**
- Line coverage ≥ 80%
- Branch coverage ≥ 75%
- Sem linhas descobertas em mudanças

**Exemplo:**

```yaml
Coverage Report - PR Changes:
────────────────────────────────
Base coverage: 82%
PR coverage: 81%
Change: -1% ❌ DECLINED

Expected:
- Minimum: 80%
- Required: Maintain or improve

Comment: "Added 200 lines with only 160 lines of tests"
         "Need 160+ tests for this PR"
```

**Feedback na PR:**

```
⚠️ Coverage declined
Current: 82% → 81% (-1%)

Changes:
src/api.js:
  - Line 42-60: NOT COVERED (new code without tests)
  - Line 65-75: NOT COVERED (new code without tests)

Add tests for these lines or reduce coverage requirement.
```

---

### 5. Code Quality Analysis (30 segundos)

Analisa code smell, vulnerabilidades, etc.

**Verificações:**
- Code smell (funções muito longas, duplicação)
- Security issues (SQL injection, XSS)
- Architecture issues (circular dependencies)
- Maintainability

**Exemplo:**

```javascript
// ❌ Code smell: função muito longa
function processUser(id, name, email, age, salary, role, dept, ...) {
  // 300 linhas de lógica misturada
}

// ✅ Code quality: funções pequenas e focadas
function processUser(id: string, userData: UserData): User {
  validateUser(userData);
  const user = createUser(userData);
  return user;
}

// Vulnerabilidade: SQL Injection
// ❌ ERRADO
const query = `SELECT * FROM users WHERE id = ${userId}`;

// ✅ CORRETO
const query = 'SELECT * FROM users WHERE id = ?';
db.query(query, [userId]);
```

**Feedback na PR:**

```
⚠️ Code Quality Issues
SonarQube found 2 issues:

Critical:
├─ src/api.js:42 - SQL Injection vulnerability
│  Potential SQL injection in user query
│  Fix: Use parameterized queries
│
Minor:
├─ src/utils.js:15 - Code smell (function too long)
│  'processPayment' is 450 lines long
│  Suggestion: Refactor into smaller functions
```

---

### 6. Security Scanning (20 segundos)

Verifica vulnerabilidades de segurança.

**Verificações:**
- Dependências vulneráveis
- Segredos no código (API keys, passwords)
- SAST (Static Application Security Testing)
- Permissões de arquivo

**Exemplo:**

```javascript
// ❌ Segredo no código
const API_KEY = 'sk_live_abc123xyz';
process.env.DB_PASSWORD = 'postgres123';

// ✅ Segredo em ambiente
const API_KEY = process.env.API_KEY;
const DB_PASSWORD = process.env.DB_PASSWORD;
```

**Feedback na PR:**

```
🔒 Security Scan Results

Critical:
├─ Hardcoded secret in src/config.js:15
│  Type: AWS Access Key
│  Action: Revoke key immediately!

Warnings:
├─ bcryptjs@2.4.3 has known vulnerabilities
│  Update to: bcryptjs@2.4.5
│
├─ Cross-site scripting detected in UserInput.js
│  Need to sanitize user input
```

---

## Configuração em GitHub

### Exemplo Completo de PR Checks

```yaml
# .github/workflows/pr-checks.yml
name: PR Checks

on: 
  pull_request:
    branches: [main, develop]

jobs:
  lint-and-format:
    runs-on: ubuntu-latest
    name: Lint & Format
    
    steps:
      - uses: actions/checkout@v3
      
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - run: npm ci
      
      - name: Run linter
        run: npm run lint
      
      - name: Check formatting
        run: npm run format:check

  type-check:
    runs-on: ubuntu-latest
    name: Type Check
    
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - run: npm ci
      
      - name: TypeScript check
        run: npm run type-check

  test:
    runs-on: ubuntu-latest
    name: Tests
    
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - run: npm ci
      
      - name: Run tests
        run: npm run test:unit
      
      - name: Coverage report
        run: npm run test:coverage
        env:
          COVERAGE_THRESHOLD: 80

  quality:
    runs-on: ubuntu-latest
    name: Code Quality
    
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - run: npm ci
      
      - name: SonarQube analysis
        run: npm run sonar
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      
      - name: Security scan
        run: npm run security:check

  # Agregador de status
  pr-status:
    needs: [lint-and-format, type-check, test, quality]
    runs-on: ubuntu-latest
    name: PR Status Check
    
    if: always()
    
    steps:
      - name: Check all required checks pass
        run: |
          if [ "${{ needs.lint-and-format.result }}" != "success" ]; then
            echo "❌ Linter failed"
            exit 1
          fi
          
          if [ "${{ needs.type-check.result }}" != "success" ]; then
            echo "❌ Type check failed"
            exit 1
          fi
          
          if [ "${{ needs.test.result }}" != "success" ]; then
            echo "❌ Tests failed"
            exit 1
          fi
          
          if [ "${{ needs.quality.result }}" != "success" ]; then
            echo "❌ Quality checks failed"
            exit 1
          fi
          
          echo "✅ All required checks passed!"
```

---

## Comentários Automáticos na PR

### Informações Úteis para Reviewer

```markdown
## 🤖 Automated Checks

### ✅ Linting & Format
- ESLint: **PASSED** (0 warnings)
- Prettier: **PASSED**

### ✅ Type Check
- TypeScript: **PASSED** (0 errors)

### ✅ Tests
- Unit tests: **60/60 PASSED** (2.5s)
- Skipped tests: 0
- Flaky tests detected: 0

### ✅ Coverage
- Overall: **82%** (target: 80%) ✅
- Added lines covered: **95%** ✅
- Coverage change: **+0.5%** ✅

### ✅ Code Quality
- SonarQube: **PASSED**
  - Code smells: 0
  - Bugs: 0
  - Vulnerabilities: 0
- Duplicate code: **2%** (target: 5%) ✅

### 🔒 Security
- Dependency audit: **PASSED** (0 vulnerabilities)
- Secret scan: **PASSED** (0 hardcoded secrets)
- SAST: **PASSED** (0 issues)

---

**Ready for code review!** ✅

All automated checks passed. Reviewer can focus on:
- Business logic correctness
- Architecture decisions
- Code style alignment
```

---

## Proteção de Branch

### Configurar GitHub

```yaml
# Settings > Branches > Branch Protection Rules

Required status checks:
├─ ✅ lint-and-format
├─ ✅ type-check
├─ ✅ test
└─ ✅ quality

Required approvals:
├─ Require pull request reviews: 1
├─ Require code owner review: YES
│  (se houver CODEOWNERS)
├─ Dismiss stale pull request approvals: YES
│  (testes falharam? review expira)

Additional rules:
├─ Require branches be up to date before merging: YES
├─ Require conversation resolution before merging: YES
└─ Block force pushes: YES
```

---

## Exceções e Overrides

### Emergency Merge

```yaml
# Em casos raríssimos de bug crítico em prod:

GitHub Admin pode:
1. Ir em PR > Settings
2. Clicar "Merge without waiting for checks"
3. DEVE documentar motivo:
   "URGENT: Critical bug in production affecting 100+ users"

Reportar ao time:
- Slack: #eng-urgent
- Email: team@company.com
- Ticket: [referência ao bug]
```

---

## Métricas

### Monitorar Qualidade de PR

```javascript
const prMetrics = {
  // Eficiência
  avg_time_to_first_check: '2s',
  avg_time_to_all_checks: '1m 30s',
  percent_prs_passing_first_run: '75%',
  
  // Confiabilidade
  false_positive_rate: '2%',
  flaky_check_rate: '1%',
  
  // Impacto
  bugs_caught_in_pr: 45,      // bugs that would escape
  security_issues_caught: 8,
  coverage_violations_blocked: 12,
  
  // Time
  avg_pr_size_kb: 15,
  avg_lines_changed: 250,
  recheck_rate: '25%'  // quantas PRS precisaram re-rodar
};
```

---

## Troubleshooting

### Checks travando

```
Problema: PR checks demorando > 5 minutos

Causas:
1. GitHub Actions queue (muitos PRS simultâneos)
2. Testes lentos (sem mock, DB real)
3. Sem cache de dependências
4. Análise SonarQube offline

Solução:
✅ Verificar GitHub status
✅ Otimizar testes (mocks)
✅ Adicionar cache
✅ Reduzir análise (apenas mudanças)
```

### Check Flakey

```
Problema: Check passa e falha aleatoriamente

Causas:
1. Testes não-determinísticos
2. Ordem de execução aleatória
3. Timeout curto demais

Solução:
✅ Tornar testes determinísticos
✅ Usar seed para random
✅ Aumentar timeout
✅ Isolar testes
```

---

## Boas Práticas

### 1. Feedback Imediato

Garantir que checks rodem **logo após push**.

```
❌ Ruim: Dev espera 5m para saber resultado
✅ Bom: Dev vê resultado em 30s
```

### 2. Informações Claras

Mensagens de erro devem apontar **exatamente** o problema.

```
❌ Ruim: "Type check failed"
✅ Bom: 
"src/api.ts:42
 Type 'string' is not assignable to type 'number'
 Fix: Change parameter from 'id' to 'id: number'"
```

### 3. Acionável

Fornecer **como corrigir**.

```
❌ Ruim: "Coverage declined to 78%"
✅ Bom:
"Coverage declined to 78% (was 82%)
Added 45 lines with only 5 tests
Need: +40 test lines to maintain 82%
Example: npm run test -- --coverage --only=src/api.js"
```

### 4. Não Sobrecarregar

Não exigir perfeccionismo demais.

```
❌ Errado: Zero warnings, zero code smell, perfection
✅ Certo: Bugfree, no vulns, 80%+ coverage, reasonable code quality
```

---

## Integração com Code Review

PR Testing **não substitui** code review humano, mas **complementa**.

```
Code Review Humano:
├─ Lógica de negócio faz sentido?
├─ Arquitetura está boa?
├─ Padrões do projeto seguidos?
├─ Há forma melhor de fazer isso?
└─ Documentação clara?

PR Testing Automático:
├─ ✅ Código formatado?
├─ ✅ Testes passam?
├─ ✅ Tipos corretos?
├─ ✅ Cobertura OK?
└─ ✅ Sem vulnerabilidades?
```

Resultado: Code Review mais rápido e focado em **o que importa**.

---

## Conclusão

Pull Request Testing é essencial para manter qualidade contínua. Ao automatizar validações técnicas, o code review se torna mais efetivo, focado em aspectos que máquinas não conseguem avaliar.

Combine com as práticas de Shift-Left, Pirâmide de Testes e Test Pipelines para máximo impacto.
