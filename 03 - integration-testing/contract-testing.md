# Contract Testing

## Origem e Conceito

Contract Testing é uma técnica que valida contratos entre serviços, garantindo que produtor (provider) e consumidor (consumer) estejam alinhados em requisições e respostas.

Originado na comunidade de microsserviços, evita que mudanças em um serviço quebrem outro sem necessidade de testes E2E complexos.

**Definição:** Contract testing valida que a interface entre dois serviços (requisição/resposta) atende às expectativas de ambas as partes.

---

## Por Que Contract Testing?

### Problemas que resolvem:

1. **Breaking changes** - Provider muda API, consumer quebra
2. **Mudanças não documentadas** - Changes não comunicados entre times
3. **Ambiguidade de contrato** - Incerteza sobre formato de dados
4. **Acoplamento invisível** - Dependências não óbvias
5. **Testes E2E complexos** - Múltiplos serviços rodar para validar
6. **Deploy descoordenado** - Consumer/Provider fora de sincronia
7. **Regressão em produção** - Contract quebrado detectado tarde

### Benefícios:

- ✅ Valida compatibilidade sem rodar tudo
- ✅ Detecta breaking changes cedo
- ✅ Segurança em deploys independentes
- ✅ Documentação viva de contratos
- ✅ Melhor comunicação entre times

---

## Conceitos Fundamentais

### Consumer (Consumidor)

Serviço que **chama** outro serviço.

```javascript
// Consumidor: UserService
class UserService {
  constructor(userApiUrl) {
    this.apiUrl = userApiUrl;
  }

  async getUser(id) {
    // Espera resposta: { id, name, email }
    const response = await fetch(`${this.apiUrl}/users/${id}`);
    return response.json();
  }
}
```

---

### Provider (Provedor)

Serviço que **fornece** a API.

```javascript
// Provedor: UserAPI
app.get('/users/:id', (req, res) => {
  // Fornece resposta: { id, name, email }
  res.json({ id: 1, name: 'John', email: 'john@example.com' });
});
```

---

### Contract

Acordo entre consumer e provider.

```javascript
// Contract: "GET /users/:id retorna { id, name, email }"
const contract = {
  request: {
    method: 'GET',
    path: '/users/1'
  },
  response: {
    status: 200,
    body: {
      id: 1,
      name: 'John',
      email: 'john@example.com'
    }
  }
};
```

---

## Arquitetura de Contract Testing

### Fluxo:

```
1. Consumer escreve teste esperando contrato
   ↓
2. Consumer gera arquivo de contrato
   ↓
3. Provider valida seu código contra contrato
   ↓
4. Provider verifica se implementação bate contrato
   ↓
5. ✅ Ou ❌ Contrato validado/quebrado
```

---

## Tipos de Contract Testing

### 1. Pact (Baseado em Exemplo)

Cria contratos a partir de exemplos de requisição/resposta.

**Exemplo - Consumer:**

```javascript
const { ConsumerPact } = require('@pact-foundation/pact');

describe('UserService Consumer Contract', () => {
  const userServicePact = new ConsumerPact({
    consumer: 'UserService',
    provider: 'UserAPI',
    port: 8080,
    logLevel: 'INFO'
  });

  it('should get user by id', async () => {
    // Define expectativa de contrato
    await userServicePact
      .addInteraction({
        state: 'user 1 exists',
        uponReceiving: 'a request for user 1',
        withRequest: {
          method: 'GET',
          path: '/users/1',
          headers: { 'Accept': 'application/json' }
        },
        willRespondWith: {
          status: 200,
          headers: { 'Content-Type': 'application/json' },
          body: {
            id: 1,
            name: 'John',
            email: 'john@example.com'
          }
        }
      })
      .then(async () => {
        // Consumer testa contra mock
        const userService = new UserService('http://localhost:8080');
        const user = await userService.getUser(1);

        expect(user.id).toBe(1);
        expect(user.name).toBe('John');
      });
  });

  // Gera arquivo de contrato
  afterAll(async () => {
    await userServicePact.finalize();
  });
});
```

**Exemplo - Provider:**

```javascript
const { VerifyPacts } = require('@pact-foundation/pact');

describe('UserAPI Provider Contract Verification', () => {
  it('should fulfill consumer contracts', async () => {
    // Verifica se provider cumpre contratos do consumer
    const verifyPacts = new VerifyPacts({
      providerBaseUrl: 'http://localhost:3000',
      pactFiles: ['pacts/UserService-UserAPI.json'],
      stateHandlers: {
        'user 1 exists': () => {
          // Setup: garantir que user 1 existe no provider
          return setupTestData();
        }
      }
    });

    await verifyPacts.verifyAll();
  });
});
```

**Características:**
- ✅ Contratos baseados em exemplos reais
- ✅ Suporta múltiplos consumers
- ✅ Broker para compartilhar contratos
- ❌ Requer sincronização de versões
- ❌ Setup mais complexo

---

### 2. OpenAPI/Swagger

Contrato baseado em especificação OpenAPI.

**Exemplo:**

```yaml
# openapi.yml
openapi: 3.0.0
info:
  title: User API
  version: 1.0.0

paths:
  /users/{id}:
    get:
      summary: Get user by id
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: integer
      responses:
        200:
          description: User found
          content:
            application/json:
              schema:
                type: object
                properties:
                  id:
                    type: integer
                  name:
                    type: string
                  email:
                    type: string
                required: [id, name, email]
```

**Teste Consumer:**

```javascript
describe('UserService with OpenAPI Contract', () => {
  it('should match user API contract', async () => {
    const userService = new UserService('http://localhost:3000');
    const user = await userService.getUser(1);

    // Valida contra OpenAPI schema
    const spec = require('./openapi.yml');
    const schema = spec.paths['/users/{id}'].get.responses[200];

    expect(ajv.validate(schema, user)).toBe(true);
  });
});
```

**Características:**
- ✅ Especificação padronizada
- ✅ Documentação e testes juntos
- ✅ Ferramentas maduras
- ❌ Menos flexível que Pact
- ❌ Requer manutenção de spec

---

### 3. Spring Cloud Contract

Contrato escrito em Groovy/YAML.

```groovy
// contracts/UserAPI.groovy
Contract.make {
  description "should get user by id"
  
  request {
    method GET()
    url "/users/1"
  }
  
  response {
    status 200
    body(
      id: 1,
      name: "John",
      email: "john@example.com"
    )
  }
}
```

**Características:**
- ✅ Integrado com Spring
- ✅ Gera stubs automaticamente
- ❌ Ecossistema Java-cêntrico

---

### 4. Teste Manual de Contrato

Abordagem simples sem framework.

```javascript
describe('Contract: GET /users/:id', () => {
  it('consumer expects correct response format', async () => {
    const response = await fetch('http://localhost:3000/users/1');
    const user = JSON.parse(await response.text());

    // Contrato
    expect(user).toHaveProperty('id', expect.any(Number));
    expect(user).toHaveProperty('name', expect.any(String));
    expect(user).toHaveProperty('email', expect.any(String));
    expect(response.status).toBe(200);
  });
});
```

**Características:**
- ✅ Simples, sem dependências
- ❌ Sem compartilhamento formal
- ❌ Difícil escalar

---

## Estratégias de Implementação

### 1. Consumer-Driven Contracts (CDC)

Consumer define contrato, provider valida.

**Fluxo:**

```
1. Consumer escreve teste esperando formato
2. Consumer gera/publica contrato
3. Provider valida código contra contrato
4. Provider notifica se quebrou
```

**Vantagens:**
- ✅ Foco no que consumer realmente precisa
- ✅ Evita over-engineering

**Exemplo:**

```javascript
// 1. Consumer escreve teste
describe('UserService Contract', () => {
  it('should get user', async () => {
    const user = await userService.getUser(1);
    
    // Consumer define expectativa
    expect(user).toEqual({
      id: expect.any(Number),
      name: expect.any(String),
      email: expect.any(String)
      // Não espera 'phone', 'address', etc
    });
  });
});

// 2. Contrato publicado (arquivo Pact)
// 3. Provider testa: se adicional 'phone' obrigatório, quebra
```

---

### 2. Provider-Driven Contracts

Provider define contrato, consumer adota.

**Fluxo:**

```
1. Provider publica OpenAPI/Swagger
2. Consumer lê e consome conforme spec
3. Provider notifica mudanças
```

**Vantagens:**
- ✅ Spec única de verdade
- ✅ Documentação centralizada

---

### 3. Bilateral Contracts

Ambos definem contrato junto.

```javascript
// Contrato acordado entre times
const userContract = {
  endpoint: 'GET /users/{id}',
  request: {
    parameters: {
      id: { type: 'integer', required: true }
    }
  },
  response: {
    status: 200,
    body: {
      id: { type: 'integer' },
      name: { type: 'string' },
      email: { type: 'string', format: 'email' }
    }
  }
};
```

---

## Implementando Pact

### Setup Consumer

```javascript
const { ConsumerPact } = require('@pact-foundation/pact');
const path = require('path');

const pact = new ConsumerPact({
  consumer: 'UserService',
  provider: 'UserAPI',
  port: 8080,
  dir: path.resolve(process.cwd(), 'pacts'),
  logLevel: 'INFO'
});

beforeAll(() => pact.setup());
afterAll(() => pact.finalize());

describe('UserService', () => {
  it('should get user', async () => {
    await pact
      .addInteraction({
        state: 'user 1 exists',
        uponReceiving: 'GET request for user 1',
        withRequest: {
          method: 'GET',
          path: '/users/1'
        },
        willRespondWith: {
          status: 200,
          body: {
            id: 1,
            name: 'John',
            email: 'john@example.com'
          }
        }
      })
      .then(async () => {
        const service = new UserService('http://localhost:8080');
        const user = await service.getUser(1);
        expect(user.name).toBe('John');
      });
  });
});
```

### Setup Provider

```javascript
const { VerifyPacts } = require('@pact-foundation/pact');
const path = require('path');

describe('UserAPI Provider Verification', () => {
  it('should verify pacts', async () => {
    const verifier = new VerifyPacts({
      providerBaseUrl: 'http://localhost:3000',
      pactFiles: [path.resolve(process.cwd(), 'pacts/UserService-UserAPI.json')],
      stateHandlers: {
        'user 1 exists': async () => {
          // Setup dados no provider
          await db.insert('users', { id: 1, name: 'John', email: 'john@example.com' });
        }
      },
      logLevel: 'INFO'
    });

    await verifier.verifyAll();
  });
});
```

---

## Anti-Patterns em Contract Testing

### 1. Contratos Muito Genéricos

```javascript
// ❌ Genérico demais
const contract = {
  response: {
    status: expect.any(Number),
    body: expect.any(Object)
  }
};

// ✅ Específico
const contract = {
  response: {
    status: 200,
    body: {
      id: expect.any(Number),
      name: expect.any(String),
      email: expect.stringMatching(/^[\w\.-]+@[\w\.-]+\.\w+$/)
    }
  }
};
```

---

### 2. Não Sincronizar Contratos

Consumer e provider sem comunicação.

```javascript
// ❌ Problema: sem sincronização
// Consumer espera: /users/1 → { id, name }
// Provider retorna: /users/1 → { userId, userName } // Mudança não comunicada

// ✅ Solução: Pact Broker sincroniza
```

---

### 3. Apenas Testar Happy Path

```javascript
// ❌ Falta casos de erro
const contract = {
  successful_response: {
    status: 200,
    body: { id: 1, name: 'John' }
  }
};

// ✅ Incluir erros
const contract = {
  successful_response: { status: 200, body: { id: 1 } },
  not_found: { status: 404, body: { error: 'User not found' } },
  invalid_id: { status: 400, body: { error: 'Invalid ID' } }
};
```

---

### 4. Contratos Muito Acoplados a Implementação

```javascript
// ❌ Detalhe implementação
const contract = {
  response: {
    body: {
      _links: { self: { href: '/users/1' } },
      _embedded: { ...complexStructure }
    }
  }
};

// ✅ Comportamento semântico
const contract = {
  response: {
    body: {
      id: 1,
      name: 'John',
      email: 'john@example.com'
    }
  }
};
```

---

### 5. Não Validar Tipos

```javascript
// ❌ sem validação de tipo
expect(user.id).toBeDefined();

// ✅ Validar tipos
expect(user.id).toEqual(expect.any(Number));
expect(user.email).toMatch(/^[\w\.-]+@[\w\.-]+\.\w+$/);
```

---

## Best Practices em Contract Testing

### 1. Começar com Consumer-Driven Contracts

Define que consumer realmente precisa.

```javascript
// Teste primeiro o que consumer precisa
const contract = {
  response: {
    body: {
      id: expect.any(Number),
      name: expect.any(String),
      email: expect.any(String)
      // Sem fields extras desnecessários
    }
  }
};
```

---

### 2. Usar Pact Broker

Compartilhar contratos entre times.

```bash
# Setup Pact Broker
docker run -d -p 8080:8080 pactfoundation/pact-broker

# Consumer publica contrato
npm run test:contract -- --publish-contracts

# Provider verifica
npm run test:contract -- --verify
```

---

### 3. Validar Todos os Estados

Diferentes cenários de dados.

```javascript
describe('UserAPI Contract', () => {
  it('should return user when exists', async () => {
    await setupState('user 1 exists');
    // test
  });

  it('should return 404 when not exists', async () => {
    await setupState('user 1 does not exist');
    // test
  });

  it('should validate email format', async () => {
    await setupState('user with invalid email');
    // test
  });
});
```

---

### 4. Documentar Contratos

Deixar claro o que é esperado.

```javascript
/**
 * Contract: GET /users/:id
 * 
 * Request:
 *   - id: integer (required, 1-999999)
 * 
 * Response (200):
 *   - id: integer
 *   - name: string (1-100 chars)
 *   - email: string (valid email format)
 * 
 * Response (404):
 *   - error: string "User not found"
 * 
 * Response (400):
 *   - error: string describing validation error
 */
```

---

### 5. Versionar Contratos

Mudanças devem ser versionadas.

```javascript
// v1 contrato
const contractV1 = {
  response: { id, name, email }
};

// v2 contrato (incompat)
const contractV2 = {
  response: { id, name, email, phone } // Campo adicionado obrigatório
};

// API mantém ambas versões
app.get('/v1/users/:id', (req, res) => { /* ... */ });
app.get('/v2/users/:id', (req, res) => { /* ... */ });
```

---

### 6. Testar Ambos Consumer e Provider

```javascript
// Consumer testa
describe('UserService Consumer', () => {
  it('should parse user response', async () => {
    // Testa parsing
  });
});

// Provider testa
describe('UserAPI Provider', () => {
  it('should return valid user', async () => {
    // Testa formatação
  });
});
```

---

### 7. Integrar ao CI/CD

Contratos validados em cada build.

```yaml
# .github/workflows/contract-test.yml
name: Contract Tests

on: [push, pull_request]

jobs:
  consumer:
    runs-on: ubuntu-latest
    steps:
      - run: npm run test:consumer:contract
      - run: npm run pact:publish

  provider:
    runs-on: ubuntu-latest
    needs: consumer
    steps:
      - run: npm run test:provider:contract
```

---

### 8. Usar Matcher Patterns

Validar formato sem valor exato.

```javascript
// Pact matchers
const { Matchers } = require('@pact-foundation/pact');

const contract = {
  response: {
    body: {
      id: Matchers.number(),
      email: Matchers.regex(/^[\w\.-]+@[\w\.-]+\.\w+$/),
      createdAt: Matchers.iso8601DateTime(),
      balance: Matchers.decimal(100.50)
    }
  }
};
```

---

## Ferramentas de Contract Testing

### Pact

Mais popular, linguagem agnóstica.

```bash
npm install -D @pact-foundation/pact
```

Suporta: JavaScript, Java, Python, Go, Ruby, .NET

---

### OpenAPI / Swagger

Especificação padronizada.

```bash
npm install -D swagger-ui-express swagger-jsdoc ajv
```

---

### Dredd

Valida API contra OpenAPI.

```bash
npm install -D dredd
dredd openapi.yml http://localhost:3000
```

---

### Spring Cloud Contract

Para ecosistema Spring.

```gradle
testCompile 'org.springframework.cloud:spring-cloud-contract-verifier'
```

---

## Caso de Uso Prático

```javascript
// Consumer: OrderService
class OrderService {
  async getInventory(productId) {
    const response = await fetch(`${this.inventoryUrl}/products/${productId}`);
    return response.json();
  }
}

// Provider: InventoryAPI
app.get('/products/:id', (req, res) => {
  res.json({
    id: 1,
    name: 'Laptop',
    stock: 10,
    price: 999.99
  });
});

// Contract Test - Consumer
describe('OrderService Contract', () => {
  const pact = new ConsumerPact({
    consumer: 'OrderService',
    provider: 'InventoryAPI'
  });

  it('should get product inventory', async () => {
    await pact
      .addInteraction({
        uponReceiving: 'GET request for product 1',
        withRequest: { method: 'GET', path: '/products/1' },
        willRespondWith: {
          status: 200,
          body: {
            id: 1,
            name: 'Laptop',
            stock: expect.any(Number),
            price: expect.any(Number)
          }
        }
      })
      .then(async () => {
        const service = new OrderService('http://localhost:8080');
        const product = await service.getInventory(1);
        expect(product.stock).toBeGreaterThanOrEqual(0);
      });
  });
});

// Contract Test - Provider
describe('InventoryAPI Contract Verification', () => {
  it('should verify pacts', async () => {
    const verifier = new VerifyPacts({
      providerBaseUrl: 'http://localhost:3000',
      pactFiles: ['pacts/OrderService-InventoryAPI.json']
    });
    await verifier.verifyAll();
  });
});
```

---

## Conclusão

Contract Testing é essencial para microsserviços:

- ✅ Valida compatibilidade sem E2E complexo
- ✅ Detecta breaking changes cedo
- ✅ Permite deploys independentes
- ✅ Documentação viva de contratos
- ✅ Melhor comunicação entre times

**Pontos-chave:**

| Aspecto | Recomendação |
|---------|-------------|
| **Estratégia** | Consumer-Driven Contracts |
| **Ferramenta** | Pact ou OpenAPI |
| **Compartilhamento** | Pact Broker para sinkronizar |
| **Cobertura** | Happy path + erros |
| **Validação** | Consumer e Provider |
| **CI/CD** | Executar em cada build |

Contract testing forma a camada de integração segura de microsserviços, permitindo evolução e mudanças com confiança.
