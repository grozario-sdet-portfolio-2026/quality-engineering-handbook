# Análise de Código: Linters e SonarQube

## Introdução

Análise de código é uma prática que usa ferramentas automatizadas para inspecionar código em busca de:

- 🔴 **Bugs** – Código que provavelmente não funciona
- 🟠 **Vulnerabilidades de segurança** – Falhas que podem ser exploradas
- 🟡 **Code smells** – Padrões que indicam problemas
- 🔵 **Problemas de padrão** – Inconsistência com guia de estilo

Diferente de testes que validam **comportamento**, análise de código valida **qualidade estrutural** do código.

### Por Que Análise Automatizada?

Humanos são excelentes em algumas coisas, mas ruins em outras:

| Aspecto | Humano | Ferramenta |
|---|---|---|
| Entender intenção do código | ✅ Excelente | ❌ Limitado |
| Detectar padrões recorrentes | ❌ Inconsistente | ✅ Perfeito |
| Encontrar vulnerabilidades conhecidas | ⚠️ Depende de expertise | ✅ Sempre atualizado |
| Revisar 5000 linhas | ❌ Cansativo, propenso a erro | ✅ Consistente |
| Manter padrão ao longo do projeto | ⚠️ Varia | ✅ Automático |

**Solução ideal:** Combinar ambos.

---

## O que é um Linter?

### Definição

Um **linter** é uma ferramenta que analisa código estático em busca de:

- Erros de sintaxe
- Estilo inconsistente
- Padrões de código problemáticos
- Violações de regras configuradas

### Analogia

Se o código fosse texto em um documento:
- **Spell-checker** = encontra erros de digitação
- **Grammar-checker** = encontra erros gramaticais
- **Style-checker** = encontra inconsistências de formatação

Um **linter** faz tudo isso para código.

### Exemplos de Linters por Linguagem

| Linguagem | Linters |
|---|---|
| JavaScript/TypeScript | ESLint, TSLint, Prettier |
| Python | PyLint, Flake8, Black |
| Java | Checkstyle, SpotBugs |
| Go | golint, gofmt |
| C# | StyleCop, FxCop |
| Ruby | RuboCop |

### O Que Um Linter Detecta

#### Erros Reais

```javascript
// ❌ Variável não definida
function calculateDiscount(price) {
  return price * discountRate;  // discountRate não existe!
}

// ❌ Acesso a undefined
const user = null;
console.log(user.name);  // Vai dar erro!

// ❌ Condicional sem else (case faltante)
if (status === 'active') {
  doSomething();
}
// E se status for algo else?
```

**Linter avisa:** ✋ Variável foi usada mas nunca definida

#### Code Smells

```javascript
// ❌ Função complexa
function processOrder(order) {
  if (order.type === 'A') {
    if (order.value > 100) {
      if (order.user.isPremium) {
        // muita lógica aninhada
      }
    }
  }
}

// ❌ Função muito grande
function doEverything() {
  // 500 linhas de código
}

// ❌ Parâmetro não usado
function getUserData(userId, unused) {
  return User.findById(userId);
}
```

**Linter avisa:** ⚠️ Complexidade alta, função grande, parâmetro não usado

#### Inconsistências de Estilo

```javascript
// ❌ Mistura de estilos
const userName='john'  // string sem espaços
let user_age = 30      // snake_case
var EMAIL = 'john@example.com'  // CONSTANT_CASE para var

function calculate_total(price,tax) {  // sem espaços
  return price+tax
}

// ✅ Consistente
const userName = 'john'
let userAge = 30
const EMAIL = 'john@example.com'

function calculateTotal(price, tax) {
  return price + tax
}
```

**Linter avisa:** 🚨 Use camelCase, nunca var, adicione espaçamento

---

## Configurando um Linter (ESLint como Exemplo)

### Instalação Básica

```bash
npm install --save-dev eslint
npx eslint --init
```

### Arquivo de Configuração (.eslintrc.js)

```javascript
module.exports = {
  env: {
    browser: true,
    es2021: true,
    node: true,
  },
  extends: [
    'eslint:recommended',
    'plugin:react/recommended',
  ],
  parserOptions: {
    ecmaVersion: 'latest',
    sourceType: 'module',
  },
  rules: {
    // Overrides
    'no-console': 'warn',           // warn em logs
    'no-unused-vars': 'error',      // erro se variável não usada
    'quotes': ['error', 'single'],  // forçar aspas simples
    'semi': ['error', 'always'],    // forçar ; no final
  },
};
```

### Executando

```bash
# Verificar um arquivo
npx eslint src/index.js

# Verificar todos os arquivos
npx eslint src/

# Corrigir automaticamente
npx eslint src/ --fix
```

### Output

```
src/utils.js
  5:9  error    'discountRate' is not defined           no-undefined
  8:3  warning  Unexpected console statement            no-console
  12:1 error    Unexpected var, use let or const        no-var

✖ 3 problems (2 errors, 1 warning)
```

---

## O que é SonarQube?

### Definição

**SonarQube** é uma plataforma de qualidade de código que:

- 🔍 Analisa código em busca de bugs, vulnerabilidades e code smells
- 📊 Gera reports de qualidade
- 📈 Rastreia tendências ao longo do tempo
- 🚪 Define quality gates (porta de qualidade)
- 📋 Integra-se com CI/CD

### Diferenças Entre Linter e SonarQube

| Aspecto | Linter | SonarQube |
|---|---|---|
| **Escopo** | Estilo + erros básicos | Análise profunda |
| **Complexidade** | Simples | Complexa |
| **Múltiplas linguagens** | Um linter por linguagem | Múltiplas linguagens |
| **Histórico** | Não | Sim (tendências) |
| **Dashboard** | Não | Sim |
| **Quality Gates** | Não | Sim |
| **Custo** | Gratuito OSS | Gratuito OSS, pago Enterprise |

### O Que SonarQube Detecta

#### 1. Bugs

```python
# ❌ Lógica quebrada
def check_user(user):
    if user:
        return True
    elif not user:
        return False
    else:
        return True  # Nunca alcançável!

# ❌ Off-by-one error
def validate_index(items, index):
    if index <= len(items):  # Deveria ser <, não <=
        return items[index]
```

**SonarQube avisa:** 🔴 Code unreachable, potential array out of bounds

#### 2. Vulnerabilidades de Segurança

```javascript
// ❌ SQL Injection
app.get('/user/:id', (req, res) => {
  const query = `SELECT * FROM users WHERE id = ${req.params.id}`;
  db.query(query);
});

// ❌ Hardcoded credentials
const API_KEY = 'sk_live_abc123xyz';

// ❌ Sem validação de input
app.post('/data', (req, res) => {
  eval(req.body.code);  // MUITO perigoso!
});
```

**SonarQube avisa:** 🔴 CRITICAL: SQL injection risk, BLOCKER: Hardcoded secret

#### 3. Code Smells

```javascript
// ❌ Muita duplicação
function getUserName(userId) {
  const user = db.query(`SELECT * FROM users WHERE id = ${userId}`);
  return user.name;
}

function getUserEmail(userId) {
  const user = db.query(`SELECT * FROM users WHERE id = ${userId}`);
  return user.email;
}

// ❌ Função muito grande (200+ linhas)
function processOrder(order) {
  // 200 linhas de lógica misturada
}

// ❌ Altos índices de complexidade
function complexLogic(a, b, c, d) {
  if (a > 0) {
    if (b < 10) {
      if (c === 'yes') {
        if (d !== null) {
          // muita coisa aninhada!
        }
      }
    }
  }
}
```

**SonarQube avisa:** 🟠 Duplicated code, MEDIUM: Large method, HIGH: High complexity

#### 4. Cobertura de Testes

```
SonarQube Report:
├── Cobertura de Linhas: 65%
├── Linhas não cobertas: 845
├── Branches não testadas: 34%
└── Recomendação: Aumente cobertura para 80%
```

### Arquitetura do SonarQube

```
┌─────────────────────┐
│   Seu Código        │
│   (Git repository)  │
└────────────┬────────┘
             │
             ▼
┌─────────────────────┐
│   Scanner           │
│   (sonar-scanner)   │
│   Analisa código    │
└────────────┬────────┘
             │
             ▼
┌─────────────────────┐
│  SonarQube Server   │
│  - Processa análise │
│  - Calcula métricas │
│  - Compara histórico│
└────────────┬────────┘
             │
             ▼
┌─────────────────────┐
│  Dashboard Web      │
│  - Reports          │
│  - Metrics          │
│  - Quality Gates    │
└─────────────────────┘
```

---

## Configurando SonarQube

### Instalação Base

```bash
# Com Docker (recomendado)
docker run -d --name sonarqube \
  -p 9000:9000 \
  sonarqube:latest

# Acesse em http://localhost:9000
# User: admin / admin
```

### Arquivo de Configuração (sonar-project.properties)

```properties
# Identificação do projeto
sonar.projectKey=my-app
sonar.projectName=My Application
sonar.projectVersion=1.0.0

# Diretório de código fonte
sonar.sources=src

# Diretório de testes
sonar.tests=tests

# Exclusões
sonar.exclusions=**/node_modules/**,dist/**

# Cobertura de testes (se usando Jest/JUnit)
sonar.javascript.lcov.reportPaths=coverage/lcov.info

# Qualidade de código
sonar.qualityProfiles.defaults=default
```

### Rodando Análise

```bash
# Instalar scanner
npm install --save-dev sonarqube-scanner

# Executar análise
npx sonar-scanner \
  -Dsonar.projectKey=my-app \
  -Dsonar.sources=src \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.login=your-token
```

### Resultado no Dashboard

```
Dashboard SonarQube
┌──────────────────────────────────────────────┐
│ My Application v1.0.0                        │
├──────────────────────────────────────────────┤
│ Bugs:              2                         │
│ Vulnerabilities:   1                         │
│ Code Smells:       15                        │
│ Coverage:          68%                       │
│ Duplication:       3.2%                      │
│ Complexity:        HIGH                      │
├──────────────────────────────────────────────┤
│ ⏱️ Last analysis: 2 hours ago                │
│ ✅ Quality Gate:  PASSED                    │
└──────────────────────────────────────────────┘
```

---

## Quality Gates

Um **Quality Gate** é um checkpoint que define se o código está pronto para merge/deploy.

### Exemplo de Quality Gate

```
Quality Gate "Prod Release"
├── Regra: Bugs = 0 ✅
├── Regra: Vulnerabilidades críticas = 0 ✅
├── Regra: Cobertura ≥ 80% ❌ FALHOU (atual: 65%)
├── Regra: Duplicação < 5% ✅
└── RESULTADO: ❌ FALHOU

Ação: Bloqueia merge até que cobertura atinja 80%
```

### Quality Gates Comuns

| Gate | Valor | Motivo |
|---|---|---|
| Bugs bloqueadores | = 0 | Bugs críticos não podem passar |
| Vulnerabilidades altas | = 0 | Segurança não negocia sem custo |
| Cobertura | ≥ 80% | Confiança em testes |
| Densidade de bugs | < 5/1000 LOC | Medida de saúde |
| Duplicação | < 5% | Manutenibilidade |
| Complexidade | LOW/MEDIUM | Legibilidade |

---

## Linter vs SonarQube: Quando Usar Qual?

### Use Linter Quando:

- ✅ Feedback **instantâneo** (ao escrever código)
- ✅ **Rápido** (roda em segundos)
- ✅ Foco em **estilo e padrão**
- ✅ **Integração IDE** (extensions do VSCode)
- ✅ Fase de **desenvolvimento local**

### Use SonarQube Quando:

- ✅ Análise **profunda** de qualidade
- ✅ Necessidade de **histórico** e tendências
- ✅ Foco em **segurança** e **cobertura**
- ✅ **Quality gates** para CI/CD
- ✅ Relatórios para **stakeholders**
- ✅ Análise de **múltiplas linguagens**

### Estratégia Ideal: Combinar Ambos

```
Developer                  GitHub/GitLab           Production
    │                           │                        │
    ▼                           ▼                        ▼
┌─────────────┐         ┌──────────────────┐    ┌──────────────┐
│ Local Dev   │         │  CI/CD Pipeline  │    │   Deploy     │
│             │         │                  │    │              │
│ • Linter    │         │ • Linter         │    │ • SonarQube  │
│ • IDE       │         │ • SonarQube      │    │ • Quality    │
│ • Feedback  │         │ • Tests          │    │   Gates      │
│   imediato  │         │ • Security scan  │    │              │
└─────────────┘         └──────────────────┘    └──────────────┘
  (segundos)              (minutos)                 (bloqueador)
```

---

## Integrando Linter + SonarQube no CI/CD

### Pipeline Recomendado

```bash
#!/bin/bash
set -e

echo "📋 Step 1: Linter Check"
npm run lint --max-warnings=0
# Se falhar, para aqui

echo "✅ Step 2: Unit Tests"
npm run test:unit
# Se coverage < 80%, falha

echo "🔍 Step 3: SonarQube Analysis"
sonar-scanner \
  -Dsonar.projectKey=my-app \
  -Dsonar.host.url=$SONAR_HOST \
  -Dsonar.login=$SONAR_TOKEN

echo "🚪 Step 4: Check Quality Gate"
# Scripts espera quality gate passar
curl -s -u "${SONAR_TOKEN}:" \
  "${SONAR_HOST}/api/qualitygates/project_status?projectKey=my-app" \
  | jq '.projectStatus.status'
# Se não for OK, falha

echo "✨ All checks passed!"
```

### GitHub Actions Example

```yaml
name: Code Quality

on: [pull_request]

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Lint
        run: npm run lint
      
      - name: Tests
        run: npm run test
      
      - name: SonarQube Scan
        uses: SonarSource/sonarcloud-github-action@master
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

---

## Problemas Comuns e Soluções

### Problema 1: "Linter é muito rigoroso"

**Sintoma:**
```
200 eslint errors encontrados
"Isso é inútil, temos coisa mais importante"
```

**Solução:**
- Configure rules de acordo com realidade do time
- Ao invés de error, use warning
- Gradualmente aumente rigor conforme time matura

```javascript
// Less strict (Warning)
'no-console': 'warn',
'no-unused-vars': 'warn',

// Strict (Error)
'no-eval': 'error',
'no-throw-literal': 'error',
```

### Problema 2: "SonarQube está sempre falho"

**Sintoma:**
```
Quality Gate: FAILED
Bugs: 234
Coverage: 15%
```

**Solução:**
- Configure gates realistas (start small)
- Plano de melhoria gradual
- Quality gate initial ≥ 50% coverage, depois aumenta

### Problema 3: "Análise muito lenta em CI/CD"

**Sintoma:**
```
PR leva 30 minutos para feedback
Time começa a ignorar checks
```

**Solução:**
- Rode linter **antes** de SonarQube (falha rápido)
- Cache de análise SonarQube
- Análise em paralelo

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - run: npm run lint  # Falha rápido
      
  sonar:
    needs: lint  # Só roda se lint passou
    runs-on: ubuntu-latest
    steps:
      - run: sonar-scanner
```

### Problema 4: "False positives"

**Sintoma:**
```
SonarQube: SQL Injection Risk!
Código: const query = `SELECT * FROM users WHERE name = '${name}'`
Developer: "Eu sei, é escapado no ORM"
```

**Solução:**
- Configure regras específicas para contexto
- Use `@SuppressWarnings` para casos legítimos
- Documente decisões

```java
@SuppressWarnings("java:S2077")  // SQL Injection false positive
public List<User> search(String name) {
  return db.query("SELECT * FROM users WHERE name = ?", name);
}
```

---

## Métricas Importantes

### Acompanhando Saúde do Código

```
Métrica                    Antes          Depois      Meta
────────────────────────────────────────────────────────────
Bugs por 1000 LOC         15             2           < 3
Vulnerabilidades críticas  5              0           = 0
Duplicação                 12%            4%          < 5%
Cobertura                  45%            82%         ≥ 80%
Complexidade (avg)         VERY HIGH      MEDIUM      < 15
Tempo de análise           -              3min        < 5min
```

### Dashboard Exemplo

```
SonarQube Timeline (últimos 6 meses)

Bugs    ▲ 50
        │     ↘ ↘ ↘     ↘ ↘   ↘
        │ 25  ╱     ╲   ╱   ╲ ╱  ╲
        │    ╱       ╲ ╱     X     ╲
        └────────────────────────────
        
Tendência: ✅ Em melhora (queda consistente)
```

---

## Best Practices para Análise de Código

### 1. Comece Simples

- Configure rules básicas primeiro
- Gradualmente aumente rigor

### 2. Automatize Tudo

- Linter em pre-commit
- SonarQube em CI/CD
- Quality gates em merge

### 3. Revise Regularmente

- Toda sprint, revise rules configuradas
- Remova regras que geram muito false positive
- Adicione novas regras se necessário

### 4. Não Use Como Puniçao

- Análise de código é **guidance**, não puniçao
- Celebrate quando cobertura melhora
- Apoiar o time em melhoria

### 5. Configure por Contexto

```javascript
// Linter mais rigoroso em código crítico
// src/payments/ → ERROR
// src/utils/  → WARNING
```

### 6. Integre com Code Review

- Use feedback de linter/SonarQube em reviews
- "SonarQube mostrou N code smells, vamos refatorar?"
- Não ignore sinais automatizados

---

## Conclusão

Linters e SonarQube são **multiplicadores de eficiência**:

- 🎯 **Linters** caturam erros **imediatamente**
- 🔍 **SonarQube** fornece **visibilidade profunda**
- 🚪 **Quality Gates** **bloqueiam** código ruim
- 📊 **Métricas** rastreiam **progresso**
- 👥 **Cultura** de qualidade se reforça

Quando bem configuradas, essas ferramentas economizam mais tempo do que consomem.
