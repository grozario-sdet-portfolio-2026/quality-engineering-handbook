# Test Flakiness

## Origem e Conceito

Flakiness em testes refere-se a testes que passam e falham inconsistentemente, sem mudanças no código.

Problema comum em testes E2E, especialmente com timing, estado compartilhado e dependências externas.

**Definição:** Um teste flakey é aquele que passa em algumas execuções e falha em outras, sem alterações no código da aplicação ou do teste.

---

## Por Que Flakiness é Problemático?

### Consequências:

1. **Perda de confiança** - Podem ignorar falhas reais
2. **Reexecução desnecessária** - Time perde tempo re-rodando
3. **CI/CD lento** - Precisa re-rodar até passar
4. **Difícil debugar** - Falha não é reproduzível
5. **Métricas enganosas** - Taxa de sucesso incorreta
6. **Atrasos em deploy** - Esperar testes ficarem verdes
7. **Burnout** - Time frustrado com testes não confiáveis

### Impacto Financeiro:

```
1 teste flakey (50% fail rate)
→ 1 min por falha
→ 5 testes por dia que falham
→ ~30 min/dia de reexecução
→ 2h/semana
→ 8h/mês
→ 2 dias/ano por desenvolvedor

× 10 devs = 20 dias/ano
```

---

## Causas Comuns de Flakiness

### 1. Timing Issues (Mais Comum)

Testes não esperando elementos aparecerem.

**Exemplo:**

```javascript
// ❌ Flakey: sem esperar
it('should display results', async () => {
  await page.fill('input[name="search"]', 'laptop');
  await page.press('input', 'Enter');
  
  const results = await page.$$('.result'); // Pode não existir ainda!
  expect(results.length).toBeGreaterThan(0);
});

// ✅ Robusto: espera elemento
it('should display results', async () => {
  await page.fill('input[name="search"]', 'laptop');
  await page.press('input', 'Enter');
  
  // Espera resultado aparecer (até 5s)
  await page.waitForSelector('.result', { timeout: 5000 });
  
  const results = await page.$$('.result');
  expect(results.length).toBeGreaterThan(0);
});
```

**Cenários:**
- Elementos não aparecerem rápido o bastante
- API lenta respondendo
- Animações não completarem

---

### 2. Estado Compartilhado

Testes afetando uns aos outros.

**Exemplo:**

```javascript
// ❌ Flakey: estado compartilhado
describe('User Tests', () => {
  let userId = null;

  it('test 1: create user', async () => {
    userId = await createUser('test@example.com');
  });

  it('test 2: find user', async () => {
    // Se test 1 não rodar ou falhar, userId é null e testa falha
    const user = await findUser(userId);
    expect(user).toBeDefined();
  });

  it('test 3: update user', async () => {
    // Falha se test 1 ou 2 falhou
    await updateUser(userId, { name: 'Updated' });
  });
});

// ✅ Isolado: sem estado compartilhado
describe('User Tests', () => {
  it('test 1: create user', async () => {
    const userId = await createUser('test@example.com');
    expect(userId).toBeDefined();
  });

  it('test 2: find user', async () => {
    const userId = await createUser('test@example.com');
    const user = await findUser(userId);
    expect(user).toBeDefined();
  });

  it('test 3: update user', async () => {
    const userId = await createUser('test@example.com');
    await updateUser(userId, { name: 'Updated' });
    // assertions
  });
});
```

---

### 3. Dependências Externas Lentas/Instáveis

API, BD, 3º partes falhando.

**Exemplo:**

```javascript
// ❌ Flakey: chamada real lenta
it('should load user data', async () => {
  // API externa pode ter downtime ou ser lenta
  const user = await fetch('https://api.external.com/users/1')
    .then(r => r.json());
  
  expect(user.id).toBe(1);
});

// ✅ Robusto: mock com timeout
it('should load user data', async () => {
  // Mock com timeout garantido
  jest.spyOn(api, 'getUser').mockImplementation(() => {
    return new Promise(resolve => {
      setTimeout(() => resolve({ id: 1, name: 'John' }), 100);
    });
  });

  const user = await api.getUser(1);
  expect(user.id).toBe(1);
});
```

---

### 4. Race Conditions

Múltiplas operações async sem sincronização.

**Exemplo:**

```javascript
// ❌ Flakey: race condition
it('should save and load data', async () => {
  await saveData({ key: 'value1' });
  const data = await loadData(); // Race com save
  expect(data.key).toBe('value1');
});

// ✅ Robusto: espera completar
it('should save and load data', async () => {
  const savePromise = saveData({ key: 'value1' });
  await savePromise; // Garante completo
  
  const data = await loadData();
  expect(data.key).toBe('value1');
});
```

---

### 5. Seletores Frágeis

CSS/selectors mudando.

**Exemplo:**

```javascript
// ❌ Flakey: selector específica demais
it('should click button', async () => {
  // Se classe mudar, falha
  await page.click('button.primary-btn-large-blue-v2');
});

// ✅ Robusto: data-testid
it('should click button', async () => {
  // Não quebra com CSS changes
  await page.click('[data-testid="submit-button"]');
});
```

---

### 6. Timing Absoluto

Usar `setTimeout` ou `sleep`.

**Exemplo:**

```javascript
// ❌ Flakey: espera fixa pode ser insuficiente/excessiva
it('should load data', async () => {
  await page.click('button');
  await page.waitForTimeout(2000); // 2 seg, pode ser pouco/muito
  
  const data = await page.textContent('[data-testid="data"]');
  expect(data).toBeDefined();
});

// ✅ Robusto: espera elemento
it('should load data', async () => {
  await page.click('button');
  
  // Espera até elemento existir (ou timeout)
  await page.waitForSelector('[data-testid="data"]', { timeout: 5000 });
  
  const data = await page.textContent('[data-testid="data"]');
  expect(data).toBeDefined();
});
```

---

### 7. Dados de Teste Inconsistentes

BD inicial inconstante.

**Exemplo:**

```javascript
// ❌ Flakey: dados randômicos
it('should find product', async () => {
  // Dados vêm de API externa não determinística
  const products = await fetchProducts();
  const product = products[Math.floor(Math.random() * products.length)];
  
  expect(product).toBeDefined();
});

// ✅ Robusto: dados determinísticos
it('should find product', async () => {
  // Setup: dados conhecidos
  await db.insert('products', { id: 1, name: 'Laptop', stock: 10 });
  
  const product = await findProduct(1);
  
  expect(product.name).toBe('Laptop');
});
```

---

### 8. Ordem de Execução Importando

Testes passam/falham conforme ordem.

**Exemplo:**

```javascript
// ❌ Flakey: ordem importa
describe('Tests', () => {
  it('test A: creates global state', () => {
    window.globalUser = { id: 1 };
  });

  it('test B: depends on test A', () => {
    // Falha se test A não rodar primeiro
    expect(window.globalUser.id).toBe(1);
  });
});

// ✅ Robusto: sem dependência
describe('Tests', () => {
  beforeEach(() => {
    // Reset estado antes de cada teste
    window.globalUser = null;
  });

  it('test A', () => {
    window.globalUser = { id: 1 };
    expect(window.globalUser.id).toBe(1);
  });

  it('test B', () => {
    // Roda independente
    window.globalUser = { id: 2 };
    expect(window.globalUser.id).toBe(2);
  });
});
```

---

### 9. Memory Leaks

Recursos não liberados entre testes.

**Exemplo:**

```javascript
// ❌ Flakey: memory leak
describe('WebSocket Tests', () => {
  it('test 1: connect', () => {
    const ws = new WebSocket('ws://localhost:8080');
    // Não fecha
  });

  it('test 2: connect again', () => {
    const ws = new WebSocket('ws://localhost:8080');
    // Falha: muitas conexões abertas
  });
});

// ✅ Robusto: cleanup
describe('WebSocket Tests', () => {
  let ws;

  afterEach(() => {
    if (ws) ws.close();
  });

  it('test 1: connect', () => {
    ws = new WebSocket('ws://localhost:8080');
  });

  it('test 2: connect again', () => {
    ws = new WebSocket('ws://localhost:8080');
  });
});
```

---

### 10. Browser State

Cache, cookies, storage entre testes.

**Exemplo:**

```javascript
// ❌ Flakey: compartilha cache
it('test 1', async () => {
  localStorage.setItem('user', 'john');
});

it('test 2', async () => {
  // localStorage ainda tem 'john' de test 1
  const user = localStorage.getItem('user');
  expect(user).toBeNull(); // Falha!
});

// ✅ Limpa antes de cada teste
beforeEach(async () => {
  localStorage.clear();
  sessionStorage.clear();
  await page.context().clearCookies();
});

it('test', async () => {
  // Limpo, sem state anterior
});
```

---

## Estratégias para Detectar Flakiness

### 1. Rodar Múltiplas Vezes

```bash
# Cypress: executar N vezes
npx cypress run --spec "tests/e2e/login.spec.js" --repeat 10
```

Se falha em algumas mas não todas = flakey.

---

### 2. Rodar em Ordem Aleatória

```bash
# Jest: shuffle ordem
npm run test:e2e -- --randomize
```

Detecta dependências invisíveis.

---

### 3. Executar Paralelo

```bash
# Playwright: múltiplos workers
npx playwright test --workers 4
```

Expõe race conditions.

---

### 4. Monitorar Flakiness Rate

```javascript
// Rastrear falhas por teste
const flakiness = {
  'login-test': 2, // Failed 2x em 50 runs = 4%
  'checkout-test': 15 // Failed 15x em 50 runs = 30% - muito flakey!
};
```

---

## Best Practices para Evitar Flakiness

### 1. Use Explicit Waits

Nunca `sleep()`, sempre espere elemento.

```javascript
// ❌ Ruim
await page.waitForTimeout(2000);

// ✅ Bom
await page.waitForSelector('[data-testid="result"]', { timeout: 5000 });
```

---

### 2. Isolate Tests Completamente

Cada teste independente.

```javascript
// ❌ Ruim: estado global
let userId;
console.log(userId); // undefined se outro teste não rodou

// ✅ Bom: setup local
beforeEach(async () => {
  userId = await createTestUser();
});
```

---

### 3. Use data-testid

Não quebra com CSS changes.

```html
<!-- HTML -->
<button class="btn primary" data-testid="submit">Submit</button>
```

```javascript
// Teste
await page.click('[data-testid="submit"]');
```

---

### 4. Clean Up After Each Test

Reset state completamente.

```javascript
afterEach(async () => {
  localStorage.clear();
  sessionStorage.clear();
  await page.context().clearCookies();
  
  // Limpar BD
  await db.query('DELETE FROM users WHERE name LIKE ?', ['test%']);
});
```

---

### 5. Mock External Dependencies

Não chamar APIs reais.

```javascript
// ❌ Flakey: API real pode falhar
it('test', async () => {
  const data = await fetch('https://api.external.com/data');
});

// ✅ Robusto: mock
it('test', async () => {
  jest.spyOn(api, 'getData').mockResolvedValue({ id: 1 });
  const data = await api.getData();
});
```

---

### 6. Use Deterministic Test Data

Dados conhecidos, não aleatórios.

```javascript
// ❌ Flaky
it('test', () => {
  const randomId = Math.floor(Math.random() * 1000);
});

// ✅ Robusto
it('test', () => {
  const id = 1; // Sempre mesmo
});
```

---

### 7. Avoid Hard Sleeps

Sempre use explicit waits.

```javascript
// ❌ Flakey
await new Promise(resolve => setTimeout(resolve, 2000));

// ✅ Robusto
await page.waitForLoadState('networkidle');
```

---

### 8. Test in Isolation

Rodar um teste de cada vez durante debug.

```bash
# Cypress: apenas um teste
npx cypress run --spec "tests/e2e/login.spec.js"

# Jest: apenas um teste
npm test -- --testNamePattern="should login"
```

---

### 9. Retry Failed Tests

Algumas falhas são realmente de infraestrutura.

```javascript
// Playwright
test.describe.configure({ retries: 2 });

test('flaky test', async () => {
  // Se falha, roda novamente
});
```

**Cuidado:** Retry mascara flakey genuinho!

---

### 10. Monitor Flakiness Over Time

Track por Sprint/Release.

```javascript
/**
 * Flakiness Metrics:
 * - Week 1: 15 flaky tests (5%)
 * - Week 2: 12 flaky tests (4%)
 * - Week 3: 8 flaky tests (3%)
 * 
 * Target: < 1% flakiness rate
 */
```

---

## Anti-Patterns em Flakiness

### 1. Aumentar Timeouts

```javascript
// ❌ Ruim: mascara problema
await page.waitForSelector('[data-testid="slow"]', { timeout: 30000 }); // 30s!

// ✅ Bom: investigar por que é lento
// A) Otimizar código
// B) Mock se externo
// C) Aumentar timeout se legítimo (até 5s)
await page.waitForSelector('[data-testid="data"]', { timeout: 5000 });
```

---

### 2. Usar sleep() para "Estabilidade"

```javascript
// ❌ Ruim: falsa sensação
await new Promise(r => setTimeout(r, 5000));

// ✅ Bom: explicit wait
await page.waitForLoadState('networkidle');
```

---

### 3. Skip Testes Flakey Sem Debugar

```javascript
// ❌ Ruim
xit('should checkout', () => {
  // Flakey, vou pular... ERRADO!
});

// ✅ Bom
it('should checkout', () => {
  // Investigar e consertar
});
```

---

### 4. Retry Indiscriminadamente

```javascript
// ❌ Ruim: retry tudo
test.describe.configure({ retries: 5 });

// ✅ Bom: retry seletivamente
test('potentially flaky', { retries: 2 }, async () => {
  // Apenas este teste
});
```

---

## Ferramentas para Detectar/Debugar Flakiness

### Cypress Built-in

```bash
# Debug um teste flakey
npx cypress run --spec "tests/e2e/flaky.spec.js" --headed
```

Time-travel entre steps.

---

### Playwright Inspector

```bash
# Debug interativo
PWDEBUG=1 npx playwright test --headed
```

---

### Flakiness Detectors

```bash
# jest-circus (built-in no Jest 27+)
npm test -- --randomize

# grep-flakiness (encontra padrões)
npm install -D @bahmutov/find-flakiness
```

---

### CI/CD Analytics

Track flakiness em cada build.

```yaml
# GitHub Actions
- name: Parse test results
  run: npm run analyze:flakiness
  
- name: Report
  run: npm run report:flakiness
```

---

## Caso Prático: Debug Flakiness

```javascript
// Teste flakey identificado
it('should add to cart and checkout', async () => {
  // Steps...
  await page.click('[data-testid="add-to-cart"]');
  const cartCount = await page.textContent('[data-testid="cart-count"]');
  expect(parseInt(cartCount)).toBe(1);
});
```

**Investigação:**

```
1. Roda 10x: falha 3x (30% flakiness)
   → Timing issue provável

2. Adiciona wait:
   await page.waitForSelector('[data-testid="cart-count"]');
   → Falha agora 1x em 10 (10%)

3. Mock API lenta:
   → Falha 0x em 10 ✅
   → Era dependência externa lenta

4. Solução: mock API nos testes
```

---

## Conclusão

Flakiness é:

- ❌ Desnecessário
- ❌ Prejudicial ao time
- ❌ Evitável com práticas corretas

**Pontos-chave:**

| Problema | Solução |
|----------|---------|
| Timing | Explicit waits, não sleep |
| Estado | Isolamento completo, cleanup |
| Seletores | data-testid, não CSS |
| Dados | Determinísticos, não randômicos |
| Dependências | Mock externas |
| Ordem | Sem dependência entre testes |
| Memory | Cleanup resources |

**Meta:** < 1% flakiness rate

Testes confiáveis você pode confiar, aumentando confiança da equipe e velocidade de entrega.
