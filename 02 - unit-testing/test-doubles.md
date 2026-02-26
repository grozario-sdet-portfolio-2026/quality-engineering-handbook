# Test Doubles

## Origem e Conceito

Test Doubles é um termo cunhado por Gerard Meszaros para descrever objetos substitutos utilizados em testes para substituir dependências reais.

Assim como "stunt double" (dublê) substitui um ator em cenas perigosas, test doubles substituem componentes reais em testes para isolar o código testado.

**Definição:** Test doubles são objetos que substituem dependências reais de forma controlada durante testes.

---

## Por Que Usar Test Doubles?

### Problemas que resolvem:

1. **Dependências lentas** - BD, chamadas HTTP
2. **Dependências não determinísticas** - Relógio, números aleatórios
3. **Dependências indisponíveis** - APIs externas em manutenção
4. **Efeitos colaterais** - Enviar email, deletar arquivo
5. **Difíceis de testar** - Exceções, timeouts
6. **Acoplamento** - Reduzir dependência entre componentes

### Benefícios:

- ✅ Testes mais rápidos
- ✅ Testes determinísticos
- ✅ Fácil simular erros
- ✅ Isolamento de unidades
- ✅ Sem efeitos colaterais

---

## Tipos de Test Doubles

### 1. Dummy

Objeto que é passado mas **nunca é realmente usado**.

**Características:**
- Preenchimento obrigatório de parâmetros
- Sem comportamento implementado
- Nunca chamado durante o teste

**Exemplo:**

```javascript
class OrderService {
  constructor(paymentGateway, logger) {
    this.payment = paymentGateway;
    this.logger = logger;
  }

  createOrder(items) {
    // logger nunca é usado neste caso
    return { items, total: items.length * 10 };
  }
}

// Teste com dummy
it('should calculate order total', () => {
  const dummyLogger = {}; // Dummy - nunca será usado
  const service = new OrderService(paymentGateway, dummyLogger);
  
  const order = service.createOrder([{ price: 10 }]);
  expect(order.total).toBe(10);
});
```

**Quando usar:**
- Parâmetro obrigatório mas não relevante ao teste
- Reduzir setup desnecessário

---

### 2. Stub

Objeto que retorna **respostas pré-configuradas** quando chamado.

**Características:**
- Retorna dados fixos
- Não valida como foi chamado
- Substitui comportamento externo

**Exemplo:**

```javascript
class UserService {
  constructor(userRepository) {
    this.repo = userRepository;
  }

  getUser(id) {
    return this.repo.findById(id);
  }

  getUserEmail(id) {
    const user = this.repo.findById(id);
    return user ? user.email : null;
  }
}

// Teste com stub
it('should return user email', () => {
  // Stub: configura respostas pré-definidas
  const stubRepository = {
    findById: (id) => ({ id: 1, email: 'john@example.com' })
  };

  const service = new UserService(stubRepository);
  const email = service.getUserEmail(1);

  expect(email).toBe('john@example.com');
});

// Com Jest
it('should return user email (Jest mock)', () => {
  const mockRepo = { findById: jest.fn().mockReturnValue({ id: 1, email: 'john@example.com' }) };
  const service = new UserService(mockRepo);
  
  expect(service.getUserEmail(1)).toBe('john@example.com');
});
```

**Características:**
- Fácil de configurar
- Não valida uso
- Substitui chamadas externas

**Quando usar:**
- Substituir BD, APIs externas
- Simular dados complexos
- Cenários é difícil reproduzir

---

### 3. Spy

Objeto que **registra (espia) chamadas** enquanto passa para implementação real ou stub.

**Características:**
- Envolve objeto real
- Registra chamadas e parâmetros
- Permite asserções sobre comportamento
- Pode validar sem asserção

**Exemplo:**

```javascript
class EmailService {
  sendEmail(to, subject, body) {
    // Implementação real
    console.log(`Email enviado para ${to}`);
    return true;
  }
}

class NotificationService {
  constructor(emailService) {
    this.email = emailService;
  }

  notifyUser(user, message) {
    return this.email.sendEmail(user.email, 'Notification', message);
  }
}

// Teste com spy
it('should send email notification', () => {
  const emailService = new EmailService();
  const spyEmail = jest.spyOn(emailService, 'sendEmail');

  const service = new NotificationService(emailService);
  service.notifyUser({ email: 'john@example.com' }, 'Hello');

  // Spy permite validar chamada
  expect(spyEmail).toHaveBeenCalledWith('john@example.com', 'Notification', 'Hello');
  expect(spyEmail).toHaveBeenCalledTimes(1);

  spyEmail.mockRestore();
});

// Spy sem mock (apenas registro)
it('should call sendEmail once', () => {
  const sendEmailSpy = jest.spyOn(emailService, 'sendEmail');
  
  // ... executa código
  
  expect(sendEmailSpy).toHaveBeenCalled();
  sendEmailSpy.mockRestore();
});
```

**Quando usar:**
- Validar que método foi chamado
- Verificar parâmetros passados
- Contar chamadas
- Combinar registro + stub

---

### 4. Mock

Objeto que valida **comportamento esperado** (chamadas, parâmetros).

**Características:**
- Pré-configura expectativas
- Falha se comportamento não ocorrer como esperado
- Substitui implementação
- Foca em interações

**Exemplo:**

```javascript
class OrderProcessor {
  constructor(paymentGateway, emailService) {
    this.payment = paymentGateway;
    this.email = emailService;
  }

  processOrder(order) {
    const payment = this.payment.charge(order.total);
    if (payment.success) {
      this.email.sendConfirmation(order.id);
      return { status: 'completed' };
    }
    return { status: 'failed' };
  }
}

// Teste com mock
it('should charge payment and send confirmation', () => {
  // Mock: espera que métodos sejam chamados
  const mockPayment = {
    charge: jest.fn().mockReturnValue({ success: true })
  };
  const mockEmail = {
    sendConfirmation: jest.fn()
  };

  const processor = new OrderProcessor(mockPayment, mockEmail);
  const result = processor.processOrder({ id: 123, total: 100 });

  // Validar interações esperadas
  expect(mockPayment.charge).toHaveBeenCalledWith(100);
  expect(mockEmail.sendConfirmation).toHaveBeenCalledWith(123);
  expect(result.status).toBe('completed');
});

// Mock com verificação de ordem
it('should charge before sending email', () => {
  const callOrder = [];

  const mockPayment = {
    charge: jest.fn(() => { callOrder.push('charge'); return { success: true }; })
  };
  const mockEmail = {
    sendConfirmation: jest.fn(() => { callOrder.push('email'); })
  };

  const processor = new OrderProcessor(mockPayment, mockEmail);
  processor.processOrder({ id: 123, total: 100 });

  expect(callOrder).toEqual(['charge', 'email']);
});
```

**Quando usar:**
- Validar comportamento de interações
- Verificar ordem de chamadas
- Garantir que métodos foram chamados
- Falhar se comportamento esperado não ocorrer

---

### 5. Fake

Implementação **funcional mas simplificada** da dependência real.

**Características:**
- Implementação real funcionando
- Mais simples que versão real
- Determinístico
- Sem efeitos colaterais
- Mais complexo que stub

**Exemplo:**

```javascript
// Implementação real (lenta)
class RealUserRepository {
  async findById(id) {
    const response = await fetch(`/api/users/${id}`);
    return response.json();
  }
}

// Fake: implementação funcional simplificada
class FakeUserRepository {
  constructor() {
    this.users = [
      { id: 1, name: 'John', email: 'john@example.com' },
      { id: 2, name: 'Jane', email: 'jane@example.com' }
    ];
  }

  async findById(id) {
    return this.users.find(u => u.id === id);
  }

  async save(user) {
    const index = this.users.findIndex(u => u.id === user.id);
    if (index >= 0) {
      this.users[index] = user;
    } else {
      this.users.push(user);
    }
    return user;
  }
}

class UserService {
  constructor(repo) {
    this.repo = repo;
  }

  async getUser(id) {
    return this.repo.findById(id);
  }

  async updateEmail(id, email) {
    const user = await this.repo.findById(id);
    if (user) {
      user.email = email;
      await this.repo.save(user);
    }
    return user;
  }
}

// Teste com fake
it('should update user email', async () => {
  const fakeRepo = new FakeUserRepository();
  const service = new UserService(fakeRepo);

  const user = await service.updateEmail(1, 'newemail@example.com');

  expect(user.email).toBe('newemail@example.com');
});

// Benefício: roda rápido, sem BD real
it('should persist user updates', async () => {
  const fakeRepo = new FakeUserRepository();
  const service = new UserService(fakeRepo);

  await service.updateEmail(1, 'new@example.com');
  const updated = await service.getUser(1);

  expect(updated.email).toBe('new@example.com');
});
```

**Quando usar:**
- Persistência (in-memory DB)
- Lógica complexa em stub seria repetitiva
- Testes de integração leve
- Performance crítica

---

## Comparação Visual

| Aspecto | Dummy | Stub | Spy | Mock | Fake |
|--------|-------|------|-----|------|------|
| **Implementacao** | Vazio | Retorna valor | Registra + real/stub | Valida chamadas | Funcional |
| **Comportamento** | Nenhum | Fixo | Variável | Verificado | Lógica real |
| **Valida chamadas?** | ❌ | ❌ | ✅ | ✅ | ❌ |
| **Implementação real** | Não | Não | Parcial | Não | Sim (simplificada) |
| **Complexidade** | ⭐ | ⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Uso** | Preenchimento | Valores externos | Validações | Interações | Integração |

---

## Ferramentas Test Doubles em JavaScript

### 1. Jest

Ferramenta mais popular, built-in mocking.

```javascript
// Mock function
const mockFn = jest.fn();
mockFn.mockReturnValue('value');
mockFn.mockResolvedValue({ data: 'async' });
mockFn.mockRejectedValue(new Error('error'));

// Mock module
jest.mock('./api', () => ({
  fetchUser: jest.fn().mockResolvedValue({ id: 1 })
}));

// Spy
jest.spyOn(obj, 'method');
```

### 2. Sinon.js

Biblioteca especializada em test doubles.

```javascript
// Stub
const stub = sinon.stub(obj, 'method').returns(value);

// Spy
const spy = sinon.spy(obj, 'method');

// Mock
const mock = sinon.mock(obj);
mock.expects('method').once().returns(value);
```

### 3. Vitest

Alternativa moderna ao Jest, API similar.

```javascript
import { describe, it, vi, expect } from 'vitest';

const mockFn = vi.fn().mockReturnValue('value');
vi.spyOn(obj, 'method');
```

---

## Anti-Patterns com Test Doubles

### 1. Over-Mocking

Mockar tudo, incluindo lógica interna.

```javascript
// ❌ Ruim
it('should call method', () => {
  const mockAdd = jest.fn().mockReturnValue(3);
  expect(mockAdd(1, 2)).toBe(3);
  expect(mockAdd).toHaveBeenCalled();
  // Não testa lógica real
});

// ✅ Bom
it('should calculate sum', () => {
  expect(add(1, 2)).toBe(3); // Testa função real
});
```

### 2. Mock Trop Específico

Mock que é frágil a mudanças.

```javascript
// ❌ Brittle
it('test', () => {
  const mock = jest.fn();
  mock.mockReturnValueOnce(1).mockReturnValueOnce(2);
  // Quebra se ordem mudar
});

// ✅ Robusto
it('test', () => {
  const mock = jest.fn().mockReturnValue(1);
  // Retorna sempre mesmo valor
});
```

### 3. Não Limpar Mocks

Deixar mocks ativos contamina testes.

```javascript
// ❌ Problema
jest.spyOn(api, 'fetch').mockReturnValue({ data: 1 });
// Não restaura

// ✅ Correto
afterEach(() => {
  jest.restoreAllMocks();
});
```

### 4. Mockar Lógica de Negócio

Mockar o que deveria ser testado.

```javascript
// ❌ Ruim
it('should validate email', () => {
  const mockValidate = jest.fn().mockReturnValue(true);
  expect(mockValidate('test@example.com')).toBe(true);
  // Não valida lógica real
});

// ✅ Bom
it('should validate valid email', () => {
  expect(validateEmail('test@example.com')).toBe(true);
  expect(validateEmail('invalid')).toBe(false);
});
```

---

## Boas Práticas com Test Doubles

### 1. Mock Apenas Dependências Externas

```javascript
// ✅ Mocke
jest.mock('axios'); // API externa
jest.mock('fs');    // Filesystem
jest.mock('redis'); // DB externa

// ❌ Não mocke
// Lógica interna, cálculos, validações
```

### 2. Usar Builder Pattern

Facilitar criação de test doubles complexos.

```javascript
class MockUserBuilder {
  constructor() {
    this.user = { id: 1, name: 'John', email: 'john@example.com' };
  }

  withId(id) {
    this.user.id = id;
    return this;
  }

  withEmail(email) {
    this.user.email = email;
    return this;
  }

  build() {
    return { ...this.user };
  }
}

// Uso fluente
const mockUser = new MockUserBuilder()
  .withId(5)
  .withEmail('custom@example.com')
  .build();
```

### 3. Usar Fixtures Compartilhadas

Reutilizar test doubles entre testes.

```javascript
describe('UserService', () => {
  let mockRepository;
  let mockEmail;

  beforeEach(() => {
    mockRepository = {
      findById: jest.fn(),
      save: jest.fn()
    };
    mockEmail = {
      send: jest.fn()
    };
  });

  it('test 1', () => { /* usa mockRepository */ });
  it('test 2', () => { /* usa mockEmail */ });
});
```

### 4. Documentar Comportamento Esperado

```javascript
/**
 * Mock simula API que retorna user em 100ms
 * - GET /users/:id → { id, name, email }
 * - Erro se id < 0
 * - Timeout após 5s
 */
const createMockApi = () => ({
  getUser: jest.fn(async (id) => {
    if (id < 0) throw new Error('Invalid ID');
    return { id, name: 'John' };
  })
});
```

---

## Quando Não Usar Test Doubles

### 1. Lógica Pura

Testar direto, sem mocks.

```javascript
// ✅ Direto
it('should sum', () => {
  expect(sum(1, 2)).toBe(3);
});
```

### 2. Componentes Simples

Mock adiciona complexidade desnecessária.

### 3. Integração Real Necessária

Às vezes é melhor integrar que mockar.

```javascript
// ✅ Integração testando contrato real
it('repository should persist data', () => {
  const realDb = new Database(':memory:');
  const repo = new UserRepository(realDb);
  
  repo.save({ id: 1, name: 'John' });
  expect(repo.findById(1).name).toBe('John');
});
```

---

## Conclusão

Test Doubles são ferramentas poderosas para:

- ✅ Isolar unidades sob teste
- ✅ Simular comportamentos externos
- ✅ Validar interações
- ✅ Tornar testes rápidos e determinísticos

**Resumo de uso:**

| Situação | Use |
|----------|-----|
| Parâmetro não relevante | Dummy |
| Sistema externo lento | Stub ou Fake |
| Validar se chamou | Spy ou Mock |
| Validar parâmetros | Mock |
| Lógica em-memória | Fake |

O segredo é usar test doubles **estrategicamente**, não cegamente.
