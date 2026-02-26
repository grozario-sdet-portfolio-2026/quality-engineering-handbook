# Integration Testing Strategy

## Origem e Conceito

Testes de integração verificam como múltiplos componentes, módulos ou serviços funcionam em conjunto.

Diferente de testes unitários que isolam unidades, testes de integração validam **comportamento combinado** de múltiplos componentes.

**Definição:** Testes de integração validam interações entre módulos, serviços, BD e dependências externas no contexto de um fluxo real.

---

## Por Que Testes de Integração?

### Problemas que resolvem:

1. **Contratos quebrados** - Interfaces entre serviços incompatíveis
2. **Falhas em BD** - Queries com problemas
3. **Integrações externas** - APIs retornando dados inesperados
4. **Fluxos completos** - Múltiplas etapas falhando
5. **Efeito cascata** - Erro em layer abaixo quebrando tudo
6. **Configuração** - Variáveis de ambiente, secrets errados

### Benefícios:

- ✅ Detecta falhas de integração
- ✅ Valida contratos entre sistemas
- ✅ Testa fluxos completos
- ✅ Confiança antes de produção
- ✅ Documentação de comportamento

---

## Níveis de Integração

### 1. Integração Intra-aplicação

Componentes dentro mesma aplicação se comunicando.

**Exemplo:**

```javascript
// UserController → UserService → UserRepository

class UserRepository {
  async findById(id) {
    return db.query('SELECT * FROM users WHERE id = ?', [id]);
  }
}

class UserService {
  constructor(repository) {
    this.repo = repository;
  }

  async getUser(id) {
    const user = await this.repo.findById(id);
    return user ? { ...user, status: 'active' } : null;
  }
}

class UserController {
  constructor(service) {
    this.service = service;
  }

  async getUser(req, res) {
    const user = await this.service.getUser(req.params.id);
    res.json(user);
  }
}

// Teste de integração: 3 camadas
it('should return user from DB through all layers', async () => {
  const db = setupTestDatabase();
  await db.insert('users', { id: 1, name: 'John' });

  const repo = new UserRepository(db);
  const service = new UserService(repo);
  const controller = new UserController(service);

  const user = await service.getUser(1);

  expect(user.name).toBe('John');
  expect(user.status).toBe('active');
});
```

**Características:**
- Testa fluxo controller → service → repository
- Usa BD real ou in-memory
- Rápido e controlado

---

### 2. Integração com Banco de Dados

Validar queries, persistência, relacionamentos.

**Exemplo:**

```javascript
describe('UserRepository Integration with Database', () => {
  let db;

  beforeEach(async () => {
    db = await setupTestDatabase();
    await db.migrate();
  });

  afterEach(async () => {
    await db.teardown();
  });

  it('should persist and retrieve user', async () => {
    const repo = new UserRepository(db);

    const user = await repo.save({ name: 'John', email: 'john@example.com' });
    const retrieved = await repo.findById(user.id);

    expect(retrieved.name).toBe('John');
  });

  it('should update user record', async () => {
    const repo = new UserRepository(db);
    const user = await repo.save({ email: 'old@example.com' });

    await repo.update(user.id, { email: 'new@example.com' });
    const updated = await repo.findById(user.id);

    expect(updated.email).toBe('new@example.com');
  });

  it('should delete user and cascade', async () => {
    const repo = new UserRepository(db);
    const user = await repo.save({ name: 'John' });

    await repo.delete(user.id);
    const deleted = await repo.findById(user.id);

    expect(deleted).toBeNull();
  });
});
```

---

### 3. Integração com APIs Externas

Validar comunicação com serviços externos.

**Exemplo:**

```javascript
describe('PaymentService Integration with Gateway', () => {
  let paymentService;
  let nock; // HTTP mocking

  beforeEach(() => {
    paymentService = new PaymentService('https://gateway.example.com');
  });

  it('should charge card successfully', async () => {
    // Mock externa API
    nock('https://gateway.example.com')
      .post('/charge', { amount: 100, currency: 'USD' })
      .reply(200, { id: 'charge_123', status: 'completed' });

    const result = await paymentService.charge(100, 'USD');

    expect(result.id).toBe('charge_123');
    expect(result.status).toBe('completed');
  });

  it('should handle API errors', async () => {
    nock('https://gateway.example.com')
      .post('/charge')
      .reply(400, { error: 'Invalid amount' });

    await expect(paymentService.charge(-100, 'USD'))
      .rejects
      .toThrow('Invalid amount');
  });
});
```

---

### 4. Integração Entre Microsserviços

Serviços distintos comunicando via HTTP, mensagens, etc.

**Exemplo:**

```javascript
// Service A: OrderService
class OrderService {
  constructor(inventoryServiceUrl) {
    this.inventoryUrl = inventoryServiceUrl;
  }

  async createOrder(items) {
    // Verifica disponibilidade em outro serviço
    const available = await fetch(`${this.inventoryUrl}/check`, {
      method: 'POST',
      body: JSON.stringify(items)
    }).then(r => r.json());

    if (!available.inStock) {
      throw new Error('Out of stock');
    }

    return { orderId: '123', items, status: 'created' };
  }
}

// Teste de integração entre serviços
describe('OrderService integration with InventoryService', () => {
  it('should create order if inventory confirms stock', async () => {
    // Mock InventoryService endpoint
    nock('http://inventory-service:3000')
      .post('/check')
      .reply(200, { inStock: true });

    const orderService = new OrderService('http://inventory-service:3000');
    const order = await orderService.createOrder([{ sku: 'A1', qty: 5 }]);

    expect(order.status).toBe('created');
  });

  it('should fail if inventory out of stock', async () => {
    nock('http://inventory-service:3000')
      .post('/check')
      .reply(200, { inStock: false });

    const orderService = new OrderService('http://inventory-service:3000');

    await expect(orderService.createOrder([{ sku: 'A1', qty: 5 }]))
      .rejects
      .toThrow('Out of stock');
  });
});
```

---

## Estratégias de Teste de Integração

### 1. Bottom-Up (De Baixo para Cima)

Integrar desde camadas inferiores (BD) para superiores (API).

**Fluxo:**

```
Teste DB → Teste Repository
         → Teste Service
         → Teste Controller
```

**Vantagens:**
- Dependências inferiores testadas primeiro
- Encontra problemas de BD cedo

**Desvantagens:**
- Pode precisar stubs de services superiores
- Não testa fluxos de negócio completos cedo

---

### 2. Top-Down (De Cima para Baixo)

Integrar desde camadas superiores (API) para inferiores (BD).

**Fluxo:**

```
Teste Controller (com mocks de Service/Repo)
       ↓
Teste Service (com mocks de Repo)
       ↓
Teste Repository (integrado com BD)
```

**Vantagens:**
- Testa fluxos de negócio cedo
- API contratualmente validada

**Desvantagens:**
- Precisa muitos stubs
- Problemas de BD descobertos tarde

---

### 3. Big Bang

Integrar todos os componentes de uma vez.

```
Controller + Service + Repository + BD + API externa
        ↓
    Teste completo
```

**Vantagens:**
- Testa tudo junto
- Mais realista

**Desvantagens:**
- Difícil debugar falhas
- Muitas variáveis
- Lento

---

### 4. Sandwich (Hybrid)

Combinar abordagens: camadas críticas bottom-up, fluxos top-down.

```
Controller (mock service)
    ↓
Service (integrado com Repository)
    ↓
Repository (BD real)
    ↓
APIs externas (nock)
```

**Vantagens:**
- Balanceado
- Testa crítico profundamente
- Fluxos com mocks controlados

**Melhor abordagem em geral.**

---

## Ambientes de Teste de Integração

### 1. Testes em Memória (In-Memory)

Componentes rodando em processo único, dados em RAM.

**Exemplo:**

```javascript
// SQLite em memória
const db = new Database(':memory:');

// Redis em memória
const redis = new Redis();

// Mock filesystem
const fs = jest.requireActual('fs');
```

**Características:**
- ✅ Muito rápido
- ✅ Sem dependências externas
- ✅ Determinístico
- ❌ Não valida performance real
- ❌ Pode não ser representativo

---

### 2. Testcontainers

Containers Docker para dependências.

**Exemplo:**

```javascript
const { DockerComposeEnvironment } = require('testcontainers');

describe('with real services', () => {
  let environment;
  let postgres;

  beforeAll(async () => {
    environment = new DockerComposeEnvironment(
      '.',
      'docker-compose.test.yml'
    );
    await environment.up();

    postgres = environment.getContainer('postgres');
  });

  afterAll(async () => {
    await environment.down();
  });

  it('should connect to real postgres', async () => {
    const db = new Database(postgres.getConnectionString());
    const result = await db.query('SELECT 1');
    expect(result[0][0]).toBe(1);
  });
});
```

**Características:**
- ✅ Ambiente realista
- ✅ Valida performance
- ✅ Múltiplas dependências
- ❌ Mais lento
- ❌ Precisa Docker

---

### 3. Ambiente Compartilhado

BD de teste compartilhada entre testes.

```javascript
// ⚠️ Problema: estado compartilhado
beforeEach(async () => {
  // Não limpa dados anteriores
});

afterEach(async () => {
  // Pode não restaurar estado
});
```

**Problemas:**
- ❌ Testes podem interferir uns nos outros
- ❌ Ordem de execução importa
- ❌ Difícil debugar

---

### 4. Banco de Dados por Teste

Cada teste tem sua própria BD/schema.

```javascript
describe('UserRepository', () => {
  let db;

  beforeEach(async () => {
    // BD nova para cada teste
    db = new Database(':memory:');
    await db.migrate();
  });

  afterEach(async () => {
    await db.close();
  });

  it('test 1', async () => { /* isolado */ });
  it('test 2', async () => { /* isolado */ });
});
```

**Vantagens:**
- ✅ Testes completamente isolados
- ✅ Ordem não importa
- ✅ Fácil debugar
- ❌ Mais lento (múltiplas migrações)

**Melhor prática:**

---

## Anti-Patterns em Testes de Integração

### 1. Testes de Integração que são Unitários

Testes que mockamdependências, não integrando nada.

```javascript
// ❌ Não é teste de integração
it('should persist user', () => {
  const mockDb = { save: jest.fn() };
  const repo = new UserRepository(mockDb);

  repo.save({ name: 'John' });

  expect(mockDb.save).toHaveBeenCalled();
  // Não testa BD real
});

// ✅ Teste de integração real
it('should persist user', async () => {
  const db = new Database(':memory:');
  const repo = new UserRepository(db);

  const user = await repo.save({ name: 'John' });

  const retrieved = await repo.findById(user.id);
  expect(retrieved.name).toBe('John');
});
```

---

### 2. Testes Compartilhando Estado

Testes que dependem de execução prévia.

```javascript
// ❌ Problema: estado compartilhado
let userId;

it('test 1: create user', async () => {
  const user = await repo.save({ name: 'John' });
  userId = user.id;
});

it('test 2: get user', async () => {
  // Depende de test 1 ter rodado
  const user = await repo.findById(userId);
  expect(user).toBeDefined();
});

// ✅ Isolado
it('test 1: create user', async () => {
  const user = await repo.save({ name: 'John' });
  expect(user.id).toBeDefined();
});

it('test 2: get user', async () => {
  const user = await repo.save({ name: 'Jane' });
  const retrieved = await repo.findById(user.id);
  expect(retrieved.name).toBe('Jane');
});
```

---

### 3. Testes Lentos Sem Razão

Chamar APIs reais, multíplos bancos, sem necessidade.

```javascript
// ❌ Lento e frágil
it('should process order', async () => {
  // Chama API real
  const rates = await fetchExchangeRates();
  // Chama BD real
  const user = await db.findUser(123);
  // Chama outro serviço
  const inventory = await inventoryService.check();

  const result = await orderService.process({ user, rates, inventory });
  expect(result.status).toBe('success');
});

// ✅ Rápido com controls
it('should process order', async () => {
  // Mock rates
  jest.spyOn(api, 'fetchRates').mockResolvedValue({ USD: 5.0 });
  // BD real
  const user = await db.findUser(123);
  // Mock inventory
  jest.spyOn(inventory, 'check').mockResolvedValue({ stock: 100 });

  const result = await orderService.process({ user, rates, inventory });
  expect(result.status).toBe('success');
});
```

---

### 4. Falta Timeout em Testes Assincronos

Testes hangando se API externas são lentas.

```javascript
// ❌ Pode travar indefinidamente
it('should fetch user from API', async () => {
  const user = await api.getUser(1);
  expect(user).toBeDefined();
});

// ✅ Com timeout
it('should fetch user from API', async () => {
  jest.setTimeout(5000); // 5 segundos max
  const user = await api.getUser(1);
  expect(user).toBeDefined();
}, 10000); // Jest timeout
```

---

### 5. Dados de Teste Irrealistas

Dados não representando cenários reais.

```javascript
// ❌ Irrealista
it('should process order', async () => {
  const order = { items: [{ price: 1 }], total: 1 };
  // Ordem muito pequena, não representa realidade
});

// ✅ Realista
it('should process order', async () => {
  const order = {
    items: [
      { sku: 'A1', qty: 2, price: 50 },
      { sku: 'B2', qty: 3, price: 30 }
    ],
    total: 190,
    taxRate: 0.15
  };
  // Cenário mais próximo real
});
```

---

## Best Practices em Testes de Integração

### 1. Isolar Testes Completamente

Cada teste totalmente independente.

```javascript
describe('UserRepository', () => {
  let db;

  beforeEach(async () => {
    // BD nova para cada teste
    db = new Database(':memory:');
    await db.migrate();
  });

  afterEach(async () => {
    await db.close();
  });

  it('test 1', async () => { /* isolado */ });
  it('test 2', async () => { /* isolado */ });
});
```

---

### 2. Usar Factories para Dados

Criar dados de teste consistentemente.

```javascript
const createTestUser = (overrides = {}) => ({
  name: 'John',
  email: 'john@example.com',
  status: 'active',
  ...overrides
});

it('should find active users', async () => {
  const user1 = await repo.save(createTestUser());
  const user2 = await repo.save(createTestUser({ status: 'inactive' }));

  const active = await repo.findActive();

  expect(active).toHaveLength(1);
  expect(active[0].name).toBe('John');
});
```

---

### 3. Testar Fluxos Completos

Não apenas operações isoladas.

```javascript
// ✅ Fluxo completo
it('should complete full order workflow', async () => {
  // 1. Create order
  const order = await orderService.create({ items: [...] });
  expect(order.status).toBe('pending');

  // 2. Process payment
  await paymentService.charge(order.id, order.total);

  // 3. Confirm order
  const confirmed = await orderService.confirm(order.id);
  expect(confirmed.status).toBe('confirmed');

  // 4. Verify in DB
  const persisted = await repo.findById(order.id);
  expect(persisted.status).toBe('confirmed');
});
```

---

### 4. Validar Limites e Erros

Não apenas caminho feliz.

```javascript
describe('OrderService', () => {
  it('should create valid order', async () => {
    const order = await service.create({ items: [...] });
    expect(order.id).toBeDefined();
  });

  it('should fail with empty items', async () => {
    await expect(service.create({ items: [] }))
      .rejects
      .toThrow('Order must have items');
  });

  it('should fail if payment fails', async () => {
    jest.spyOn(payment, 'charge').mockRejectedValue(new Error('Declined'));
    
    await expect(service.create({ items: [...] }))
      .rejects
      .toThrow('Payment declined');
  });
});
```

---

### 5. Usar Testcontainers para Realismo

Containers reais para dependências, não mocks.

```javascript
import { PostgreSqlContainer } from 'testcontainers';

describe('User Repository with Real Postgres', () => {
  let container;
  let db;

  beforeAll(async () => {
    container = await new PostgreSqlContainer().start();
    db = new Database(container.getConnectionString());
    await db.migrate();
  });

  afterAll(async () => {
    await container.stop();
  });

  it('should handle concurrent writes', async () => {
    const promises = Array(10).fill(null).map((_, i) =>
      repo.save({ name: `User ${i}` })
    );

    const users = await Promise.all(promises);
    expect(users).toHaveLength(10);
  });
});
```

---

### 6. Documentar Setup Complexo

Deixar claro dependências e configuração.

```javascript
/**
 * Integration tests for OrderService
 * 
 * Dependencies:
 * - PostgreSQL (in-memory SQLite for tests)
 * - Redis (mocked)
 * - PaymentGateway API (nock mocked)
 * 
 * Setup:
 * - Migrates schema before each test
 * - Cleans tables after each test
 * - Restores all mocks
 */
describe('OrderService Integration', () => {
  let db;
  let redis;

  beforeEach(async () => {
    // Setup
  });

  afterEach(async () => {
    // Cleanup
  });
});
```

---

### 7. Separar Testes de Integração de Unitário

Estrutura clara de folders.

```
src/
  __tests__/
    unit/
      UserService.test.js
      UserRepository.test.js
    integration/
      UserService.integration.test.js
      OrderFlow.integration.test.js
    e2e/
      UserFlow.e2e.test.js
```

**Como executar:**

```bash
# Apenas unitários (rápido)
npm run test:unit

# Apenas integração
npm run test:integration

# Tudo
npm run test
```

---

### 8. Otimizar Performance

Reutilizar setup quando seguro.

```javascript
// ✅ Migração uma vez, cleanup entre testes
beforeAll(async () => {
  db = new Database(':memory:');
  await db.migrate();
});

afterEach(async () => {
  // Limpar dados
  await db.truncate('users');
});

afterAll(async () => {
  await db.close();
});
```

---

## Ferramentas para Testes de Integração

### Jest + Supertest

```javascript
const request = require('supertest');
const app = require('../app');

describe('GET /users/:id', () => {
  it('should return user', async () => {
    const response = await request(app)
      .get('/users/1')
      .expect(200);

    expect(response.body.name).toBe('John');
  });
});
```

### Nock (HTTP Mocking)

```javascript
const nock = require('nock');

nock('https://api.example.com')
  .get('/users/1')
  .reply(200, { name: 'John' });
```

### Testcontainers

```javascript
const { PostgreSqlContainer } = require('testcontainers');

const container = await new PostgreSqlContainer().start();
```

### SQLite / SQLite3 (In-Memory)

```javascript
const Database = require('better-sqlite3');
const db = new Database(':memory:');
```

---

## Conclusão

Testes de integração são essenciais para:

- ✅ Validar comportamento combinado de módulos
- ✅ Detectar falhas em contratos
- ✅ Testar fluxos reais de negócio
- ✅ Ganhar confiança antes de produção

**Pontos-chave:**

| Aspecto | Recomendação |
|---------|-------------|
| **Isolamento** | Cada teste independente, BD própria |
| **Estratégia** | Sandwich (top-down + bottom-up) |
| **Ambiente** | In-memory quando possível, realista quando necessário |
| **Velocidade** | Otimizar sem sacrificar realismo |
| **Documentação** | Deixar claro o que integra com o que |
| **Separação** | Pastas diferentes de unitários e E2E |

Testes de integração formam a camada média da pirâmide de testes: menos rápidos que unitários, mas mais rápidos que E2E, validando o comportamento real do sistema.
