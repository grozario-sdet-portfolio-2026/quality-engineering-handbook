# Test Coverage

## Origem e Conceito

Test Coverage (Cobertura de Testes) é uma métrica que mede a proporção do código-fonte que é executada durante a execução dos testes.

Originada na engenharia de software como forma de quantificar a qualidade e completude de testes, evoluiu de simples métricas para ferramentas sofisticadas de análise.

**Definição:** Cobertura é o percentual de linhas, branches ou caminhos de código que são testados por uma suíte de testes.

---

## Tipos de Cobertura

### 1. Line Coverage (Cobertura de Linha)

Mede quantas linhas de código foram executadas durante os testes.

**O que mede:**
- Cada linha de código foi executada?
- Linhas mortas ou perdidas?

**Exemplo:**

```javascript
function calculateTotal(items) {
  // Linha 1: sempre executada
  if (!items || items.length === 0) {
    return 0; // Linha 2: talvez não executada
  }
  return items.reduce((sum, item) => sum + item.price, 0); // Linha 3
}

// Teste com line coverage:
it('should calculate total', () => {
  expect(calculateTotal([{ price: 10 }])).toBe(10);
  // Line coverage: 66% (linhas 1 e 3 executadas, linha 2 não)
});
```

**Características:**
- Mais fácil de medir
- Pode não detectar lógica condicional não testada
- Primeira métrica a implementar

---

### 2. Branch Coverage (Cobertura de Ramificação)

Mede se todas as ramificações (if/else, switch/case) foram executadas.

**O que mede:**
- Todos os caminhos de if/else foram testados?
- Todos os cases do switch foram testados?

**Exemplo:**

```javascript
function getDiscount(type) {
  if (type === 'premium') {        // Branch 1
    return 20;
  } else if (type === 'standard') { // Branch 2
    return 10;
  } else {                          // Branch 3
    return 0;
  }
}

// Baixa cobertura de branch:
it('should return premium discount', () => {
  expect(getDiscount('premium')).toBe(20);
  // Branch coverage: 33% (1 de 3 branches testadas)
});

// Alta cobertura de branch:
it.each([
  ['premium', 20],
  ['standard', 10],
  ['basic', 0]
])('should return %s discount', (type, expected) => {
  expect(getDiscount(type)).toBe(expected);
  // Branch coverage: 100% (todos os 3 branches)
});
```

**Características:**
- Mais rigorosa que line coverage
- Detecta lógica condicional não testada
- Recomendada

---

### 3. Statement Coverage

Mede quantas declarações (statements) foram executadas.

Similar a line coverage, mas conta statements lógicas, não linhas físicas.

**Exemplo:**

```javascript
const result = x > 0 ? 'positive' : 'non-positive'; // 1 statement
const total = a + b; // 1 statement
```

---

### 4. Function Coverage

Mede quantas funções foram chamadas durante os testes.

**Exemplo:**

```javascript
function add(a, b) { return a + b; }
function subtract(a, b) { return a - b; }
function multiply(a, b) { return a * b; }

// Teste com 66% function coverage
it('should add', () => {
  expect(add(1, 1)).toBe(2);
});
```

---

### 5. Path Coverage

Mede quantos caminhos possíveis (combinações de branches) foram testados.

**Exemplo:**

```javascript
function logic(a, b) {
  if (a > 0) {        // Branch A: true/false
    if (b > 0) {      // Branch B: true/false
      return 'both positive';
    }
    return 'a positive';
  }
  return 'a non-positive';
}

// Caminhos possíveis: 4 (2^2 combinações)
// it.each com 4 casos = 100% path coverage
```

**Características:**
- Mais completo e rigoroso
- Combinações exponenciais
- Nem sempre prático (com muitos branches)

---

## Correlação Entre Métricas

```
Line Coverage
    ↓ (mais rigoroso)
Branch Coverage
    ↓ (mais rigoroso)
Path Coverage
```

**Exemplo prático:**

|Cenário|Line|Branch|Path|
|---|---|---|---|
|Sem nenhum teste|0%|0%|0%|
|Apenas linha principal|50%|0%|0%|
|1 caminho de if/else|100%|50%|33%|
|Todos os branches|100%|100%|66%|
|Todos os caminhos|100%|100%|100%|

---

## Não Confundir: Cobertura ≠ Qualidade

Uma alta cobertura não garante testes de qualidade.

**Cobertura 100% ruim:**

```javascript
it('should call setValue', () => {
  const obj = new MyClass();
  spyOn(obj, 'setValue');
  obj.setValue(10);
  expect(obj.setValue).toHaveBeenCalled();
  // 100% coverage, mas não testa comportamento
});
```

**Cobertura 80% boa:**

```javascript
it('should set and retrieve value', () => {
  const obj = new MyClass();
  obj.setValue(10);
  expect(obj.getValue()).toBe(10);
  // Testa comportamento real, menos linhas mas mais valor
});
```

**Princípio:** Cobertura é métrica de quantidade, não qualidade.

---

## Recomendações de Limites

### Por Tipo de Código

|Tipo|Recomendado|Realista|
|---|---|---|
|Lógica de negócio crítica|90-100%|85%|
|Funcionalidades padrão|80-90%|75%|
|Utilitários e helpers|70-80%|60%|
|Código legado com testes|60-70%|40-50%|
|Gerado automaticamente|N/A|N/A|

### Estratégia Progressiva

```
Fase 1 (novo projeto)   → 60% line, 40% branch
Fase 2 (estável)        → 75% line, 60% branch
Fase 3 (maduro)         → 85% line, 75% branch
Fase 4 (crítico)        → 90% line, 85% branch
```

---

## Ferramentas de Cobertura

### Jest (JavaScript)

```bash
# Gerar relatório de cobertura
jest --coverage

# Detalhado
jest --coverage --verbose

# Com threshold (verificar limites)
jest --coverage --coverage-threshold=global,lines=80
```

**Saída:**

```
PASS  src/utils.test.js
────────────────────────────────────────────
File          | % Stmts | % Branch | % Funcs | % Lines
────────────────────────────────────────────
All files     |   85.5  |   72.3   |   90.2  |   85.1
 utils.js     |   85.5  |   72.3   |   90.2  |   85.1
────────────────────────────────────────────
```

### Istanbul

Ferramenta padrão de cobertura em JavaScript.

```javascript
// .nycrc.json
{
  "reporter": ["html", "text", "lcov"],
  "require": ["@babel/register"],
  "sourceMap": true,
  "instrument": true,
  "lines": 80,
  "statements": 80,
  "functions": 80,
  "branches": 75
}
```

### SonarQube

Análise estática e cobertura centralizada.

```bash
npm install -D sonarqube-scanner
```

### Codecov / Coveralls

Integração com CI/CD para rastrear cobertura ao longo do tempo.

---

## Implementar Cobertura em CI/CD

### Configuração GitHub Actions

```yaml
name: Tests with Coverage

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - uses: actions/setup-node@v2
        with:
          node-version: '18'
      
      - run: npm install
      
      - run: npm run test:coverage
      
      - uses: codecov/codecov-action@v2
        with:
          files: ./coverage/lcov.info
          fail_ci_if_error: true
          verbose: true
```

### Pre-commit Hook

```bash
#!/bin/bash
# .husky/pre-commit

npm run test:coverage
if [ $? -ne 0 ]; then
  echo "Coverage threshold not met"
  exit 1
fi
```

---

## Anti-Patterns de Cobertura

### 1. Obsessão por 100%

Buscar 100% é ineficiente.

```javascript
// Código praticamente impossível de deixar falhar
function DEFAULT_VALUE() {
  return { id: 1 };
}

// Teste apenas para atingir 100%
it('should return default', () => {
  expect(DEFAULT_VALUE()).toEqual({ id: 1 });
});
```

**Realidade:** 80-90% é mais prático e sustentável.

---

### 2. Ignorar Cobertura Completamente

Deixar código sem testes reduz confiabilidade.

**Solução:** Implementar threshold mínimo.

---

### 3. Coverage Falso

Linhas executadas mas sem assertions.

```javascript
// ❌ Cobertura falsa
it('should process', () => {
  processData({ x: 10 }); // Linha executada, sem assertion
});

// ✅ Cobertura real
it('should process data', () => {
  const result = processData({ x: 10 });
  expect(result).toBe(20); // Assertion validando
});
```

---

### 4. Testes Snapshotados Sem Revisão

Snapshots geram "cobertura" falsa.

```javascript
// ❌ Snapshot sem semântica
it('should match', () => {
  expect(largeObject).toMatchSnapshot();
});

// ✅ Assertions semânticas
it('should have correct structure', () => {
  expect(result).toHaveProperty('id');
  expect(result.status).toBe('active');
});
```

---

## Cenários de Exclusão de Cobertura

Nem todo código precisa de teste.

### Código Legítimo de Excluir

**1. Console e logs:**
```javascript
/* c8 ignore start */
if (process.env.DEBUG) {
  console.log('Debug info');
}
/* c8 ignore end */
```

**2. Código defensivo:**
```javascript
function mustNeverHappen() {
  // istanbul ignore next
  throw new Error('This should never happen');
}
```

**3. Code gerado:**
```javascript
// auto-generated, do not modify
```

---

## Rastrear Cobertura ao Longo do Tempo

### Importante monitorar:

- Tendência de cobertura (aumentando/diminuindo?)
- Regressões (mudanças quebraram testes?)
- Coverage debt (código sem testes acumulando?)

**Exemplo com badge:**

```markdown
![Coverage](https://img.shields.io/badge/coverage-85%-brightgreen)
```

**Ferramentas:**
- Codecov
- Coveralls
- Sonarqube

---

## Boas Práticas

### 1. Definir Thresholds Claros

```javascript
// package.json
{
  "jest": {
    "coverageThreshold": {
      "global": {
        "lines": 80,
        "statements": 80,
        "functions": 80,
        "branches": 75
      }
    }
  }
}
```

### 2. Aumentar Progressivamente

Não tentar atingir 100% de repente.

```
Semana 1: 60% → 65%
Semana 2: 65% → 70%
Semana 3: 70% → 75%
```

### 3. Revisar Regularmente

Coverage report deve ser parte do PR review.

```bash
# Gerar relatório HTML
jest --coverage --coverage-reporters=html

# Abrir em navegador
open coverage/index.html
```

### 4. Focar em Código Crítico Primeiro

```javascript
// src/critical/
→ Atacar 95% coverage primeiro

// src/utils/
→ 80% coverage

// src/helpers/
→ 70% coverage
```

### 5. Combinar com Mutation Testing

Cobertura + mutação = confiança real.

```bash
npm install -D stryker

# Rodar testes de mutação
npx stryker run
```

---

## Casos de Uso Reais

### Startup / MVP

- **Target:** 60-70% line coverage
- **Foco:** Features críticas
- **Objetivo:** Velocidade e iteração

### Aplicação Estável

- **Target:** 75-85% line, 60-70% branch
- **Foco:** Lógica de negócio + erros
- **Objetivo:** Confiabilidade

### Sistema Crítico / Financeiro

- **Target:** 85-95% line, 80-90% branch
- **Foco:** Todos os caminhos
- **Objetivo:** Poucos defeitos em produção

### Biblioteca / SDK

- **Target:** 90-100% line, 85-95% branch
- **Foco:** Todos os APIs e edge cases
- **Objetivo:** Compatibilidade garantida

---

## Conclusão

Test Coverage é uma métrica valiosa quando:

- ✅ Usada como guia, não como meta
- ✅ Combinada com qualidade de testes
- ✅ Monitorada tendência ao longo do tempo
- ✅ Implementada de forma progressiva
- ✅ Integrada ao pipeline de CI/CD

Relembrando: **Cobertura alta ≠ Código confiável, mas cobertura baixa ≈ Código arriscado.**
