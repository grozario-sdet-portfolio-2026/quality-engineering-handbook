# Database Testing

## Origem e Conceito

Testes de banco de dados validam camada de persistência: queries, relacionamentos, constraints, transações e integridade de dados.

Historicamente negligenciado em favor de testes de lógica de negócio, testes de BD são críticos para confiabilidade do sistema.

**Definição:** Testes de BD validam operações (CRUD), queries, relacionamentos, constraints e comportamento transacional do banco de dados.

---

## Por Que Testes de BD?

### Problemas que resolvem:

1. **Queries incorretas** - SELECT retornando dados errados
2. **Constraints violados** - NULL em campo NOT NULL, duplicatas
3. **Relacionamentos quebrados** - Foreign keys malformadas
4. **Performance** - Queries N+1, falta de índices
5. **Migrations quebradas** - Schema changes causando downtime
6. **Concorrência** - Race conditions, deadlocks
7. **Data integrity** - Corrupção ou inconsistência de dados
8. **Transações** - Rollback/commit não funcionando

### Benefícios:

- ✅ Detecta problemas de persistência cedo
- ✅ Valida schema e dados
- ✅ Confiança em operações críticas
- ✅ Evita correções custosas pós-deploy
- ✅ Documenta comportamento de BD

---

## Tipos de Testes de BD

### 1. Testes de Query (SQL)

Validar retorno correto de dados.

**Exemplo:**

```javascript
describe('User Queries', () => {
  let db;

  beforeEach(async () => {
    db = new Database(':memory:');
    await db.migrate();
  });

  it('should find user by email', async () => {
    // Setup
    await db.insert('users', {
      id: 1,
      email: 'john@example.com',
      name: 'John'
    });

    // Execute query
    const user = await db.query(
      'SELECT * FROM users WHERE email = ?',
      ['john@example.com']
    );

    // Assert
    expect(user[0].name).toBe('John');
  });

  it('should find active users sorted by created_at', async () => {
    await db.insert('users', [
      { id: 1, status: 'active', name: 'Alice', created_at: '2024-01-01' },
      { id: 2, status: 'inactive', name: 'Bob', created_at: '2024-01-02' },
      { id: 3, status: 'active', name: 'Charlie', created_at: '2024-01-03' }
    ]);

    const active = await db.query(
      'SELECT * FROM users WHERE status = ? ORDER BY created_at DESC',
      ['active']
    );

    expect(active).toHaveLength(2);
    expect(active[0].name).toBe('Charlie');
    expect(active[1].name).toBe('Alice');
  });

  it('should return empty result for no matches', async () => {
    const users = await db.query(
      'SELECT * FROM users WHERE email = ?',
      ['nonexistent@example.com']
    );

    expect(users).toHaveLength(0);
  });
});
```

**Características:**
- Testa SELECT puro
- Valida filtros, ordenação, paginação
- Sem lógica de aplicação

---

### 2. Testes CRUD (Create, Read, Update, Delete)

Validar operações básicas.

**Exemplo:**

```javascript
describe('UserRepository CRUD', () => {
  let db;
  let repo;

  beforeEach(async () => {
    db = new Database(':memory:');
    await db.migrate();
    repo = new UserRepository(db);
  });

  // Create
  it('should create user', async () => {
    const user = await repo.create({
      name: 'John',
      email: 'john@example.com'
    });

    expect(user.id).toBeDefined();
    expect(user.name).toBe('John');
  });

  it('should throw error on duplicate email', async () => {
    await repo.create({ name: 'John', email: 'john@example.com' });

    await expect(
      repo.create({ name: 'Jane', email: 'john@example.com' })
    ).rejects.toThrow('Email already exists');
  });

  // Read
  it('should read user by id', async () => {
    const created = await repo.create({ name: 'John', email: 'john@example.com' });
    const read = await repo.findById(created.id);

    expect(read.name).toBe('John');
  });

  it('should return null for non-existent user', async () => {
    const user = await repo.findById(999);
    expect(user).toBeNull();
  });

  // Update
  it('should update user', async () => {
    const created = await repo.create({ name: 'John', email: 'john@example.com' });

    const updated = await repo.update(created.id, { name: 'Jane' });

    expect(updated.name).toBe('Jane');
    expect(updated.email).toBe('john@example.com'); // Não mudou
  });

  // Delete
  it('should delete user', async () => {
    const created = await repo.create({ name: 'John', email: 'john@example.com' });

    await repo.delete(created.id);

    const deleted = await repo.findById(created.id);
    expect(deleted).toBeNull();
  });
});
```

---

### 3. Testes de Relacionamentos

Validar relações entre tabelas.

**Exemplo:**

```javascript
describe('User and Order Relationships', () => {
  let db;

  beforeEach(async () => {
    db = new Database(':memory:');
    await db.migrate();
  });

  it('should find orders for user', async () => {
    // Create user
    const user = await db.insert('users', { name: 'John' });

    // Create orders
    await db.insert('orders', [
      { user_id: user.id, total: 100 },
      { user_id: user.id, total: 200 }
    ]);

    // Find orders
    const orders = await db.query(
      'SELECT * FROM orders WHERE user_id = ?',
      [user.id]
    );

    expect(orders).toHaveLength(2);
    expect(orders.map(o => o.total)).toEqual([100, 200]);
  });

  it('should handle cascade delete', async () => {
    const user = await db.insert('users', { name: 'John' });
    const order = await db.insert('orders', { user_id: user.id, total: 100 });

    // Delete user (should cascade)
    await db.delete('users', { id: user.id });

    // Orders should be deleted too
    const remaining = await db.query('SELECT * FROM orders WHERE id = ?', [order.id]);
    expect(remaining).toHaveLength(0);
  });

  it('should prevent orphan records', async () => {
    // Try to create order with non-existent user
    await expect(
      db.insert('orders', { user_id: 999, total: 100 })
    ).rejects.toThrow('Foreign key constraint');
  });
});
```

---

### 4. Testes de Constraints

Validar regras de integridade.

**Exemplo:**

```javascript
describe('Database Constraints', () => {
  let db;

  beforeEach(async () => {
    db = new Database(':memory:');
    await db.migrate();
  });

  // NOT NULL constraint
  it('should enforce NOT NULL constraint', async () => {
    await expect(
      db.insert('users', { name: 'John', email: null })
    ).rejects.toThrow('NOT NULL constraint failed');
  });

  // UNIQUE constraint
  it('should enforce UNIQUE constraint', async () => {
    await db.insert('users', { name: 'John', email: 'john@example.com' });

    await expect(
      db.insert('users', { name: 'Jane', email: 'john@example.com' })
    ).rejects.toThrow('UNIQUE constraint failed');
  });

  // CHECK constraint
  it('should enforce CHECK constraint', async () => {
    await expect(
      db.insert('users', { name: 'John', email: 'john@example.com', age: -5 })
    ).rejects.toThrow('CHECK constraint failed');
  });

  // DEFAULT values
  it('should apply default values', async () => {
    const user = await db.insert('users', {
      name: 'John',
      email: 'john@example.com'
    });

    expect(user.status).toBe('active'); // Default
    expect(user.created_at).toBeDefined(); // Default timestamp
  });
});
```

---

### 5. Testes de Transações

Validar comportamento transacional.

**Exemplo:**

```javascript
describe('Database Transactions', () => {
  let db;

  beforeEach(async () => {
    db = new Database(':memory:');
    await db.migrate();
  });

  it('should commit transaction on success', async () => {
    const transaction = db.transaction();

    try {
      await transaction.insert('users', { name: 'John' });
      await transaction.insert('users', { name: 'Jane' });
      await transaction.commit();
    } catch (e) {
      await transaction.rollback();
    }

    const users = await db.query('SELECT COUNT(*) as count FROM users');
    expect(users[0].count).toBe(2);
  });

  it('should rollback transaction on error', async () => {
    const transaction = db.transaction();

    try {
      await transaction.insert('users', { name: 'John' });
      await transaction.insert('users', { name: null }); // Erro!
      await transaction.commit();
    } catch (e) {
      await transaction.rollback();
    }

    const users = await db.query('SELECT COUNT(*) as count FROM users');
    expect(users[0].count).toBe(0); // Rollback
  });

  it('should prevent dirty reads with isolation', async () => {
    const txn1 = db.transaction();
    const txn2 = db.transaction();

    // Txn1 insere, mas não faz commit
    await txn1.insert('users', { name: 'John' });

    // Txn2 não deve ver a inserção pendente
    const users = await txn2.query('SELECT COUNT(*) as count FROM users');
    expect(users[0].count).toBe(0);

    await txn1.rollback();
    await txn2.rollback();
  });
});
```

---

### 6. Testes de Performance

Validar velocidade de queries.

**Exemplo:**

```javascript
describe('Database Performance', () => {
  let db;

  beforeEach(async () => {
    db = new Database(':memory:');
    await db.migrate();

    // Insert test data
    for (let i = 1; i <= 10000; i++) {
      db.insert('users', { name: `User ${i}`, status: 'active' });
    }
  });

  it('should find users quickly with index', async () => {
    const start = Date.now();

    await db.query('SELECT * FROM users WHERE status = ?', ['active']);

    const duration = Date.now() - start;
    expect(duration).toBeLessThan(50); // Must be < 50ms
  });

  it('should handle pagination efficiently', async () => {
    const start = Date.now();

    // Get page 100
    await db.query(
      'SELECT * FROM users LIMIT 10 OFFSET 1000'
    );

    const duration = Date.now() - start;
    expect(duration).toBeLessThan(100);
  });

  it('should detect N+1 queries', async () => {
    const queryLog = [];
    db.on('query', (sql) => queryLog.push(sql));

    // Simulate N+1
    const users = await db.query('SELECT * FROM users LIMIT 10');
    for (const user of users) {
      await db.query('SELECT * FROM orders WHERE user_id = ?', [user.id]);
    }

    // Should be 1 + 10 = 11 queries
    expect(queryLog).toHaveLength(11); // Alert!
  });
});
```

---

### 7. Testes de Migrações

Validar schema changes funcionam corretamente.

**Exemplo:**

```javascript
describe('Database Migrations', () => {
  it('should migrate to v1', async () => {
    const db = new Database(':memory:');
    await db.migrate('001_initial.sql');

    const tables = await db.query(
      "SELECT name FROM sqlite_master WHERE type='table'"
    );

    expect(tables.map(t => t.name)).toContain('users');
    expect(tables.map(t => t.name)).toContain('orders');
  });

  it('should migrate to v2 with new column', async () => {
    const db = new Database(':memory:');
    await db.migrate('001_initial.sql');
    await db.migrate('002_add_user_status.sql');

    const columns = await db.query('PRAGMA table_info(users)');
    expect(columns.map(c => c.name)).toContain('status');
  });

  it('should rollback migration', async () => {
    const db = new Database(':memory:');
    await db.migrate('001_initial.sql');
    await db.migrate('002_add_user_status.sql');

    await db.rollback();

    const columns = await db.query('PRAGMA table_info(users)');
    expect(columns.map(c => c.name)).not.toContain('status');
  });

  it('should handle migration with data', async () => {
    const db = new Database(':memory:');
    await db.migrate('001_initial.sql');

    // Add data
    await db.insert('users', { name: 'John' });

    // Migrate
    await db.migrate('002_add_status.sql');

    // Data preserved
    const users = await db.query('SELECT * FROM users');
    expect(users[0].name).toBe('John');
    expect(users[0].status).toBe('active'); // Default
  });
});
```

---

## Ambientes de Teste de BD

### 1. SQLite Em Memória

Rápido, sem dependências externas.

```javascript
const Database = require('better-sqlite3');
const db = new Database(':memory:');
```

**Vantagens:**
- ✅ Muito rápido
- ✅ Sem setup
- ✅ Totalmente isolado

**Desvantagens:**
- ❌ Sintaxe SQL pode diferir (SQLite vs PostgreSQL)
- ❌ Sem validação de performance real

---

### 2. Testcontainers

BD real em Docker container.

```javascript
const { PostgreSqlContainer } = require('testcontainers');

let postgres;

beforeAll(async () => {
  postgres = await new PostgreSqlContainer()
    .withEnvironment('POSTGRES_PASSWORD', 'password')
    .start();
});

afterAll(async () => {
  await postgres.stop();
});
```

**Vantagens:**
- ✅ BD real (PostgreSQL, MySQL)
- ✅ Testa performance real
- ✅ Validação de SQL dialeto

**Desvantagens:**
- ❌ Mais lento que in-memory
- ❌ Precisa Docker
- ❌ Setup extra

---

### 3. Snapshot / Seeding

Pré-popular BD com dados conhecidos.

```javascript
const seedDatabase = async (db) => {
  await db.insert('users', [
    { id: 1, name: 'Alice', status: 'active' },
    { id: 2, name: 'Bob', status: 'inactive' }
  ]);

  await db.insert('orders', [
    { id: 1, user_id: 1, total: 100 },
    { id: 2, user_id: 1, total: 200 }
  ]);
};

beforeEach(async () => {
  await seedDatabase(db);
});
```

---

## Anti-Patterns em Testes de BD

### 1. Não Limpar Estado Entre Testes

Estados compartilhados causam testes frágeis.

```javascript
// ❌ Problema: estado compartilhado
describe('User Repository', () => {
  beforeAll(async () => {
    db = new Database(':memory:');
    await db.migrate();
  });

  it('test 1', async () => {
    await db.insert('users', { name: 'John' });
    // Não limpa
  });

  it('test 2', async () => {
    const users = await db.query('SELECT * FROM users');
    expect(users).toHaveLength(1); // Falha dependente
  });
});

// ✅ Limpar entre testes
describe('User Repository', () => {
  beforeEach(async () => {
    db = new Database(':memory:');
    await db.migrate();
  });

  it('test 1', async () => {
    await db.insert('users', { name: 'John' });
    // BD nova em cada teste
  });

  it('test 2', async () => {
    const users = await db.query('SELECT * FROM users');
    expect(users).toHaveLength(0); // Sempre verdadeiro
  });
});
```

---

### 2. Testes Muito Lentos Sem Reason

Usar BD real quando in-memory seria suficiente.

```javascript
// ❌ Lento: startup PostgreSQL real
it('should insert user', async () => {
  const db = new PostgreSqlContainer()...start(); // 5+ segundos!
  // test
  await db.stop();
});

// ✅ Rápido: in-memory para unidade
it('should insert user', async () => {
  const db = new Database(':memory:'); // Imediato
  await db.insert('users', { name: 'John' });
  expect(user.id).toBeDefined();
});
```

---

### 3. Testar SQL Direto Sem Abstração

Duplicar queries em testes.

```javascript
// ❌ Problema: SQL duplicado
class UserRepository {
  async find() {
    return this.db.query('SELECT * FROM users WHERE status = ?', ['active']);
  }
}

it('should find active users', async () => {
  const query = 'SELECT * FROM users WHERE status = ?'; // Repetindo query!
  const result = await db.query(query, ['active']);
  // ...
});

// ✅ Usar repository em testes
it('should find active users', async () => {
  const repo = new UserRepository(db);
  const users = await repo.find();
  expect(users).toHaveLength(1);
});
```

---

### 4. Não Validar Constraints

Deixar bugs de integridade passarem.

```javascript
// ❌ Não valida constraints
it('should create user', async () => {
  const user = await repo.create({ name: 'John', email: null }); // Bug!
  expect(user).toBeDefined();
});

// ✅ Validar constraints
it('should enforce email not null', async () => {
  await expect(
    repo.create({ name: 'John', email: null })
  ).rejects.toThrow('NOT NULL constraint');
});
```

---

### 5. Dados de Teste Irrealistas

Não representar cenários reais.

```javascript
// ❌ Irrealista
it('should handle many users', async () => {
  await db.insert('users', { name: 'X' }); // 1 usuário
});

// ✅ Realista
it('should handle many users efficiently', async () => {
  const users = Array(10000).fill(null).map((_, i) => ({
    name: `User ${i}`,
    email: `user${i}@example.com`
  }));

  const start = Date.now();
  await db.insertMany('users', users);
  const duration = Date.now() - start;

  expect(duration).toBeLessThan(5000);
});
```

---

## Best Practices em Testes de BD

### 1. Usar BD Nova Por Teste

Isolamento total.

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

### 2. Usar Builders para Dados de Teste

Facilitar criação consistente.

```javascript
const userBuilder = {
  defaults: {
    name: 'John',
    email: 'john@example.com',
    status: 'active'
  },

  create: (overrides = {}) => ({
    ...userBuilder.defaults,
    ...overrides
  })
};

it('should find active users', async () => {
  await db.insert('users', userBuilder.create({ status: 'active' }));
  await db.insert('users', userBuilder.create({ name: 'Jane', status: 'inactive' }));

  const active = await repo.findActive();
  expect(active).toHaveLength(1);
});
```

---

### 3. Separar Testes por Tipo

Organização clara.

```
src/
  __tests__/
    db/
      queries.test.js
      crud.test.js
      relationships.test.js
      constraints.test.js
      transactions.test.js
      migrations.test.js
      performance.test.js
```

---

### 4. Testar Tanto Queries Quanto Integração

Cobertura completa.

```javascript
// Teste query isolada
it('SELECT should return active users', async () => {
  const result = await db.query('SELECT * FROM users WHERE status = ?', ['active']);
  expect(result).toHaveLength(1);
});

// Teste integração com repository
it('repository should find active users', async () => {
  const repo = new UserRepository(db);
  const users = await repo.findActive();
  expect(users).toHaveLength(1);
});
```

---

### 5. Documentar Estrutura de BD

Deixar schema claro.

```javascript
/**
 * Database Schema for Tests
 * 
 * users
 *   - id (PK)
 *   - name (TEXT, NOT NULL)
 *   - email (TEXT, NOT NULL, UNIQUE)
 *   - status (TEXT, DEFAULT 'active')
 *   - created_at (TIMESTAMP, DEFAULT NOW())
 * 
 * orders
 *   - id (PK)
 *   - user_id (FK → users.id, CASCADE)
 *   - total (DECIMAL(10,2))
 *   - created_at (TIMESTAMP)
 */
describe('Database', () => {
  // ...
});
```

---

### 6. Validar Índices e Performance

Detectar queries lentas.

```javascript
it('should use index on email column', async () => {
  // Criar 10k users
  for (let i = 0; i < 10000; i++) {
    await db.insert('users', {
      name: `User ${i}`,
      email: `user${i}@example.com`
    });
  }

  const start = Date.now();
  await db.query('SELECT * FROM users WHERE email = ?', ['user5000@example.com']);
  const duration = Date.now() - start;

  expect(duration).toBeLessThan(10); // Must be fast
});
```

---

### 7. Testar Migrações como Código

Validar schema changes.

```javascript
it('migration 001_initial should create tables', async () => {
  const db = new Database(':memory:');
  await runMigration(db, '001_initial.sql');

  const tables = await db.query("SELECT name FROM sqlite_master WHERE type='table'");
  expect(tables.map(t => t.name)).toEqual(['users', 'orders']);
});

it('migration should be reversible', async () => {
  const db = new Database(':memory:');
  await runMigration(db, '001_initial.sql');
  await runMigration(db, '002_add_status.sql');
  await rollbackMigration(db);

  const columns = await db.query('PRAGMA table_info(users)');
  expect(columns.map(c => c.name)).not.toContain('status');
});
```

---

### 8. Usar Fixtures Pré-preparadas

Para dados complexos.

```javascript
// fixtures/users.json
{
  "john": {
    "id": 1,
    "name": "John",
    "email": "john@example.com",
    "status": "active"
  },
  "jane": {
    "id": 2,
    "name": "Jane",
    "email": "jane@example.com",
    "status": "inactive"
  }
}

// Uso em teste
const fixtures = require('../fixtures/users.json');

beforeEach(async () => {
  await db.insert('users', Object.values(fixtures));
});
```

---

## Ferramentas para Testes de BD

### SQLite / better-sqlite3

```javascript
const Database = require('better-sqlite3');
const db = new Database(':memory:');

db.exec(`
  CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT UNIQUE
  )
`);

db.prepare('INSERT INTO users (name, email) VALUES (?, ?)')
  .run('John', 'john@example.com');
```

### Testcontainers

```javascript
const { PostgreSqlContainer } = require('testcontainers');

const container = await new PostgreSqlContainer().start();
const connectionString = container.getConnectionString();
```

### Knex.js (Query Builder)

```javascript
const knex = require('knex');

const db = knex({
  client: 'sqlite3',
  connection: ':memory:',
  useNullAsDefault: true
});

await db('users').insert({ name: 'John' });
const users = await db('users').select();
```

### TypeORM / Prisma

ORM com suporte a testes.

```javascript
const { PrismaClient } = require('@prisma/client');

const db = new PrismaClient();

await db.user.create({ data: { name: 'John' } });
```

---

## Estrutura de Testes de BD Recomendada

```javascript
describe('Database - User Operations', () => {
  let db;

  // Setup: criar BD nova
  beforeEach(async () => {
    db = new Database(':memory:');
    await db.migrate();
  });

  // Cleanup
  afterEach(async () => {
    await db.close();
  });

  describe('CRUD Operations', () => {
    it('should create user', async () => {
      const user = await db.insert('users', { name: 'John' });
      expect(user.id).toBeDefined();
    });

    it('should read user', async () => {
      const created = await db.insert('users', { name: 'John' });
      const read = await db.findById('users', created.id);
      expect(read.name).toBe('John');
    });
  });

  describe('Constraints', () => {
    it('should enforce NOT NULL', async () => {
      await expect(
        db.insert('users', { name: null })
      ).rejects.toThrow();
    });
  });

  describe('Performance', () => {
    it('should find user quickly', async () => {
      // Insert many, query performance
    });
  });
});
```

---

## Conclusão

Testes de BD são essenciais para:

- ✅ Validar integridade de dados
- ✅ Detectar problemas de schema
- ✅ Confiança em operações críticas
- ✅ Documentar comportamento esperado

**Pontos-chave:**

| Aspecto | Recomendação |
|---------|-------------|
| **Isolamento** | BD nova por teste |
| **Ambiente** | SQLite in-memory para speed, Testcontainers para realismo |
| **Escopo** | Testar queries, CRUD, relacionamentos, constraints |
| **Performance** | Incluir testes de performance |
| **Migrações** | Testar schema changes |
| **Limpeza** | Restaurar estado entre testes |

Testes de BD fornecem confiança que dados são persisted corretamente e acessíveis conforme esperado.
