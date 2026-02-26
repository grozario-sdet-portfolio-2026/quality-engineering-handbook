# Unit Testing Best Practices

## 1. Test Behavior, Not Implementation

Escrever testes focados no comportamento esperado, não em detalhes de implementação.

**Princípio:**
- Teste o "o que" e "quando", não o "como"
- Refatorações seguras não devem quebrar testes
- Testes devem resistir a mudanças internas

**Exemplo:**

```javascript
// ❌ Ruim: testando implementação
it('should call setEmail method', () => {
  const user = new User();
  spyOn(user, 'setEmail');
  user.setEmail('john@example.com');
  expect(user.setEmail).toHaveBeenCalled();
});

// ✅ Bom: testando comportamento
it('should update user email when setEmail is called', () => {
  const user = new User();
  user.setEmail('john@example.com');
  expect(user.getEmail()).toBe('john@example.com');
});
```

**Benefícios:**
- Testes mais resilientes
- Refatorações seguras
- Foco no valor entregue

---

## 2. Follow the AAA Pattern

Estruturar testes em três seções: **Arrange**, **Act**, **Assert**.

**Princípio:**
- Deixa claro o que está sendo testado
- Fácil de ler e manter
- Estrutura consistente

**Exemplo:**

```javascript
it('should calculate discount for premium users', () => {
  // Arrange - preparar dados
  const user = new User({ type: 'premium', spent: 1000 });
  const order = new Order({ total: 100 });

  // Act - executar ação
  const discount = calculateDiscount(user, order);

  // Assert - validar resultado
  expect(discount).toBe(15); // 15% para premium
});
```

**Benefícios:**
- Estrutura clara
- Fácil identificar cenário, ação, resultado
- Manutenção simplificada

---

## 3. Use Descriptive Test Names

Nomes que descrevem cenário, ação e resultado esperado.

**Padrão:**
`should [expected behavior] when [condition/trigger]`

**Exemplo:**

```javascript
// ❌ Ruim
it('works', () => {});
it('email validation', () => {});

// ✅ Bom
it('should throw error when email is empty', () => {});
it('should return discount for orders over 100 when user is premium', () => {});
it('should calculate total including tax for international orders', () => {});
```

**Benefícios:**
- Documentação viva
- Fácil entender intent sem ler código
- Debugging simplificado

---

## 4. Keep Tests Isolated and Independent

Cada teste deve ser executável isoladamente sem depender de outros.

**Princípio:**
- Nenhuma dependência entre testes
- Ordem de execução não importa
- Cleanup apropriado

**Exemplo:**

```javascript
// ❌ Ruim: testes dependentes
let user;

it('test 1', () => {
  user = new User('John');
  expect(user.name).toBe('John');
});

it('test 2', () => {
  // Depende de test 1 ter rodado
  expect(user.name).toBe('John');
});

// ✅ Bom: testes independentes
it('test 1', () => {
  const user = new User('John');
  expect(user.name).toBe('John');
});

it('test 2', () => {
  const user = new User('Jane');
  expect(user.name).toBe('Jane');
});
```

**Benefícios:**
- Ordem de execução não importa
- Corridas paralelas possíveis
- Falhas isoladas e fáceis de debugar

---

## 5. Test Error Cases and Edge Cases

Testar não apenas o "caminho feliz" mas também erros e limites.

**Exemplo:**

```javascript
describe('calculateAge', () => {
  // Caminho feliz
  it('should calculate age correctly', () => {
    expect(calculateAge('1990-01-01')).toBe(34);
  });

  // Casos de erro
  it('should throw error when date is invalid', () => {
    expect(() => calculateAge('invalid')).toThrow();
  });

  it('should throw error when date is in future', () => {
    expect(() => calculateAge('2030-01-01')).toThrow();
  });

  // Limites
  it('should handle age 0 (born today)', () => {
    const today = new Date().toISOString().split('T')[0];
    expect(calculateAge(today)).toBe(0);
  });

  it('should handle very old ages', () => {
    expect(calculateAge('1900-01-01')).toBe(124);
  });
});
```

**Benefícios:**
- Coverage genuína
- Confiabilidade aumentada
- Documentação de limites

---

## 6. Use Factories and Builders for Test Data

Criar dados de teste de forma consistente e fácil de manter.

**Exemplo:**

```javascript
// ❌ Ruim: dados espalhados
it('test 1', () => {
  const user = { id: 1, name: 'John', email: 'john@example.com', age: 30 };
  // ...
});

it('test 2', () => {
  const user = { id: 2, name: 'Jane', email: 'jane@example.com', age: 25 };
  // ...
});

// ✅ Bom: factory function
const createUser = (overrides = {}) => ({
  id: 1,
  name: 'John',
  email: 'john@example.com',
  age: 30,
  ...overrides
});

it('test 1', () => {
  const user = createUser();
  // ...
});

it('test 2', () => {
  const user = createUser({ name: 'Jane', age: 25 });
  // ...
});
```

**Benefícios:**
- Fácil reutilizar dados
- Mudanças centralizadas
- Menos duplicação

---

## 7. Mock Only External Dependencies

Mockar apenas dependências externas, não lógica interna.

**Princípio:**
- Mocke: API calls, BD, filesystems, relógio
- Não mocke: lógica de negócio, funções puras
- Integre componentes internos quando possível

**Exemplo:**

```javascript
// ✅ Bom: mockar API externa
it('should fetch user from API', async () => {
  const mockApi = jest.spyOn(api, 'getUser')
    .mockResolvedValue({ id: 1, name: 'John' });

  const user = await userService.getUser(1);

  expect(user.name).toBe('John');
  mockApi.mockRestore();
});

// Não mockar lógica interna
it('should calculate discount correctly', () => {
  const price = 100;
  const discount = discountService.calculate(price, 'PREMIUM');
  expect(discount).toBe(15); // Test real logic
});
```

**Benefícios:**
- Testes validam integração real
- Detecta problemas reais
- Confianç maior no código

---

## 8. Use test.each for Multiple Similar Cases

Consolidar múltiplos testes em um único `test.each` quando testam mesma lógica.

**Exemplo:**

```javascript
// ❌ Ruim: múltiplos testes repetitivos
it('should sum 1+2', () => {
  expect(sum(1, 2)).toBe(3);
});
it('should sum 2+1', () => {
  expect(sum(2, 1)).toBe(3);
});
it('should sum 5+10', () => {
  expect(sum(5, 10)).toBe(15);
});

// ✅ Bom: consolidado em test.each
it.each([
  [1, 2, 3],
  [2, 1, 3],
  [5, 10, 15],
  [0, 0, 0],
  [-5, 5, 0]
])('should sum %i + %i = %i', (a, b, expected) => {
  expect(sum(a, b)).toBe(expected);
});
```

**Benefícios:**
- Menos duplicação
- Fácil adicionar casos
- Output mais limpo
- Manutenção centralizada

---

## 9. Keep Tests Simple and Focused

Um teste deve validar um comportamento específico.

**Princípio:**
- Uma assertion principal por teste
- Ou múltiplas assertions relacionadas
- Evitar testes complexos

**Exemplo:**

```javascript
// ❌ Ruim: teste faz muita coisa
it('should process user signup', () => {
  const user = signUp('john@example.com', 'password123');
  expect(user.email).toBe('john@example.com');
  expect(user.isActive).toBe(false);
  expect(user.emailVerified).toBe(false);
  expect(user.role).toBe('USER');
  expect(user.createdAt).toBeDefined();
  sendEmail(user.email);
  expect(emailQueue.length).toBe(1);
});

// ✅ Bom: testes focados
it('should create user with correct data', () => {
  const user = signUp('john@example.com', 'password123');
  expect(user).toEqual({
    email: 'john@example.com',
    isActive: false,
    emailVerified: false,
    role: 'USER'
  });
});

it('should send verification email after signup', () => {
  signUp('john@example.com', 'password123');
  expect(emailQueue.length).toBe(1);
  expect(emailQueue[0].to).toBe('john@example.com');
});
```

**Benefícios:**
- Fácil entender o que testa
- Falhas específicas
- Debugging simplificado

---

## 10. Use Strong and Specific Assertions

Validações específicas, não genéricas.

**Exemplo:**

```javascript
// ❌ Ruim: validações fraca
it('should calculate total', () => {
  const total = calculateTotal([10, 20, 30]);
  expect(total).toBeDefined(); // Muito fraco
  expect(total).toEqual(expect.any(Number)); // Genérico
});

// ✅ Bom: validações específicas
it('should calculate total correctly', () => {
  const total = calculateTotal([10, 20, 30]);
  expect(total).toBe(60); // Valor exato
  expect(total).toBeGreaterThan(0);
  expect(total).toBeLessThanOrEqual(100);
});

it('should validate email format', () => {
  const email = 'user@example.com';
  expect(email).toMatch(/^[\w\.-]+@[\w\.-]+\.\w+$/);
});
```

**Benefícios:**
- Maior confiabilidade
- Detecta mais erros
- Valores bem definidos

---

## 11. Clean Up After Tests

Restaurar estado original após cada teste.

**Exemplo:**

```javascript
// ✅ Bom: limpeza de mocks
describe('UserService', () => {
  afterEach(() => {
    jest.restoreAllMocks();
  });

  it('test 1', () => {
    jest.spyOn(api, 'getUser').mockResolvedValue(...);
    // test
  });

  it('test 2', () => {
    // Mocks anteriores foram limpos
  });
});

// Ou com beforeEach/afterEach
describe('Database', () => {
  beforeEach(() => {
    database.connect();
  });

  afterEach(() => {
    database.disconnect();
  });
});
```

**Benefícios:**
- Testes não interferem uns nos outros
- Estado consistente
- Execução paralela segura

---

## 12. Write Tests as You Code (TDD)

Escrever testes simultaneamente ou antes da implementação.

**Fluxo Red-Green-Refactor:**

```javascript
// 1. Red: escrever teste que falha
it('should calculate tax correctly', () => {
  expect(calculateTax(100, 'BR')).toBe(17); // Falha
});

// 2. Green: implementação mínima
function calculateTax(amount, country) {
  return amount * 0.17;
}

// 3. Refactor: melhorar código mantendo testes verdes
const TAX_RATES = {
  BR: 0.17,
  US: 0.10
};

function calculateTax(amount, country) {
  return amount * TAX_RATES[country];
}
```

**Benefícios:**
- Design melhor desde início
- Menos retrabalho
- Documentação simultânea
- Refatorações mais seguras

---

## 13. Test Boundary Conditions

Validar limites e valores extremos.

**Exemplo:**

```javascript
describe('validateAge', () => {
  it('should accept valid ages (18-100)', () => {
    expect(validateAge(18)).toBe(true);
    expect(validateAge(50)).toBe(true);
    expect(validateAge(100)).toBe(true);
  });

  it('should reject age below 18', () => {
    expect(validateAge(17)).toBe(false);
    expect(validateAge(0)).toBe(false);
    expect(validateAge(-5)).toBe(false);
  });

  it('should reject age above 100', () => {
    expect(validateAge(101)).toBe(false);
    expect(validateAge(150)).toBe(false);
  });

  it('should handle edge cases', () => {
    expect(validateAge(null)).toBe(false);
    expect(validateAge(undefined)).toBe(false);
    expect(validateAge('30')).toBe(true); // Conversão
  });
});
```

**Benefícios:**
- Cobertura completa
- Evita off-by-one errors
- Maior confiabilidade

---

## 14. Use Meaningful Setup and Teardown

Estruturar beforeEach/afterEach para clareza.

**Exemplo:**

```javascript
// ✅ Bom: setup claro e minimal
describe('OrderService', () => {
  let orderService;
  let mockPaymentGateway;

  beforeEach(() => {
    // Setup - só o necessário
    mockPaymentGateway = {
      process: jest.fn().mockResolvedValue({ success: true })
    };
    orderService = new OrderService(mockPaymentGateway);
  });

  afterEach(() => {
    // Cleanup
    jest.restoreAllMocks();
  });

  it('should process order successfully', () => {
    const order = orderService.create({ items: [1, 2] });
    expect(mockPaymentGateway.process).toHaveBeenCalled();
  });
});
```

**Benefícios:**
- Setup transparente
- Menos surpresas
- Fácil de manter

---

## 15. Avoid Brittle Tests

Testes que quebram com mudanças menores desnecessárias.

**Exemplo:**

```javascript
// ❌ Brittle: assertiona estrutura interna
it('should return user object', () => {
  const user = getUser(1);
  expect(Object.keys(user)).toEqual(['id', 'name', 'email']);
});
// Quebra se adicionar um campo novo

// ✅ Robusto: testa valores importantes
it('should return user with required fields', () => {
  const user = getUser(1);
  expect(user).toHaveProperty('id', 1);
  expect(user).toHaveProperty('name');
  expect(user).toHaveProperty('email');
});

// Ainda melhor: comportamento
it('should fetch user by id', () => {
  const user = getUser(1);
  expect(user.id).toBe(1);
  expect(user.name).toBeDefined();
});
```

**Benefícios:**
- Testes mais resilientes
- Refatorações seguras
- Menos manutenção

---

## 16. Group Related Tests with describe

Organizar testes em blocos lógicos.

**Exemplo:**

```javascript
describe('UserService', () => {
  describe('#create', () => {
    it('should create user with valid data', () => {});
    it('should throw error with invalid email', () => {});
    it('should hash password before saving', () => {});
  });

  describe('#update', () => {
    it('should update user email', () => {});
    it('should not allow email update without verification', () => {});
  });

  describe('#delete', () => {
    it('should soft delete user', () => {});
    it('should preserve user data for 30 days', () => {});
  });
});
```

**Benefícios:**
- Organização clara
- Fácil navegar
- Agrupa contexto relacionado

---

## 17. Review and Refactor Tests

Testes também precisam de manutenção.

**Princípio:**
- Revisar testes regularmente
- Refatorar testes legáveis
- Remover testes duplicados
- Atualizar conforme código evolui

**Exemplo:**

```javascript
// Refatorar para clareza
const createOrderWithItems = (items) => 
  new Order({ items, total: items.reduce((sum, item) => sum + item.price, 0) });

it.each([
  [
    [{ price: 10 }, { price: 20 }],
    30
  ]
])('should calculate total', (items, expected) => {
  const order = createOrderWithItems(items);
  expect(order.total).toBe(expected);
});
```

**Benefícios:**
- Testes manteníveis
- Evita technical debt em testes
- Qualidade consistente

---

## 18. Test Error Handling Explicitly

Validar tratamento de erros de forma específica.

**Exemplo:**

```javascript
// ✅ Bom: validado explicitamente
describe('UserService', () => {
  it('should throw ValidationError with message when email is invalid', () => {
    expect(() => {
      userService.create({ email: 'invalid' });
    }).toThrow(ValidationError);

    // Validar mensagem
    expect(() => {
      userService.create({ email: 'invalid' });
    }).toThrow('Email must be valid');
  });

  it('should throw NotFoundError when user does not exist', () => {
    expect(() => {
      userService.getById(999);
    }).toThrow(NotFoundError);
  });
});
```

**Benefícios:**
- Comportamento de erro validado
- Mensagens diagnósticas claras
- Debugging facilitado

---

## 19. Use Snapshots Wisely

Snapshots para estruturas complexas, não triviais.

**Princípio:**
- Use para objetos/componentes complexos
- Não use para valores simples atirados
- Revise mudanças sempre

**Exemplo:**

```javascript
// ✅ Bom: snapshot para HTML complex
it('should render user profile', () => {
  const profile = renderProfile(user);
  expect(profile).toMatchSnapshot();
});

// ❌ Ruim: snapshot desnecessário
it('should create user', () => {
  const user = createUser('John');
  expect(user).toMatchSnapshot();
});
// Melhor seria:
it('should create user', () => {
  const user = createUser('John');
  expect(user.name).toBe('John');
});
```

**Benefícios:**
- Detecta mudanças inesperadas
- Quando bem usado, economiza tempo

---

## 20. Measure and Monitor Test Quality

Acompanhar métricas de qualidade de testes.

**Indicadores importantes:**

- **Code Coverage**: % de linhas testadas (almeje 80%+)
- **Mutation Score**: % de mutações detectadas
- **Flaky Tests**: Testes que falham irregularmente
- **Test Execution Time**: Velocidade da suíte
- **Defect Rate**: Bugs chegando em produção

**Exemplo:**

```javascript
// Executar com coverage report
// package.json
{
  "scripts": {
    "test:coverage": "jest --coverage"
  }
}

// Investigar testes flakey
jest --detectOpenHandles // Detecta memory leaks
jest --runInBand         // Executa sequencial (debug flakey)
```

**Benefícios:**
- Qualidade evidenciada
- Melhoria contínua
- Identificação de problemas

---

## Conclusão

Boas práticas em testes unitários são baseadas em:

- **Clareza**: código legível e intenção clara
- **Isolamento**: testes independentes
- **Confiabilidade**: assertions fortes
- **Manutenibilidade**: estrutura e organização
- **Eficiência**: tempo e recursos

Aplicar essas práticas resulta em:
- Maior confiança no código
- Refatorações seguras
- Debugging mais rápido
- Sistemas mais confiáveis
- Time mais produtivo
