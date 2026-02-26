# Unit Testing Anti-Patterns

## 1. Testing Implementation Details Instead of Behavior

Testes acoplados à implementação interna, não ao comportamento esperado.

**Problema:**
- Testes quebram com refatorações seguras
- Alta manutenção
- Difícil de entender a intenção do teste

**Exemplo ruim:**
```javascript
it('should call setName method', () => {
  const obj = new User();
  spyOn(obj, 'setName');
  obj.setName('John');
  expect(obj.setName).toHaveBeenCalled();
});
```

**Melhor:**
```javascript
it('should update user name', () => {
  const user = new User();
  user.setName('John');
  expect(user.getName()).toBe('John');
});
```

---

## 2. Excessive Mocking and Stubbing

Mockar tudo, perdendo valor de teste de integração.

**Problema:**
- Testes passam mas código quebra na prática
- Falso senso de segurança
- Difícil de detectar problemas reais

**Solução:**
- Mock apenas dependências externas (API, DB, etc)
- Teste comportamento real quando possível
- Use testes de integração para validar contratos

---

## 3. Not Isolating Units

Testes que dependem de múltiplas unidades, fugindo do escopo unitário.

**Problema:**
- Falhas em cascata
- Difícil debugar qual componente falhou
- Lento feedback

**Solução:**
- Um teste deve testar uma única responsabilidade
- Injete dependências para isolamento

---

## 4. Shared Test Fixtures

Compartilhar setup entre múltiplos testes causa acoplamento.

**Problema:**
```javascript
let user;

beforeEach(() => {
  user = { name: 'John', email: 'john@example.com' };
});

it('test 1', () => { /* modifica user */ });
it('test 2', () => { /* depende das modificações */ });
```

**Solução:**
- Setup localizado ou factory functions
- Testes independentes

---

## 5. Assertion Overkill

Múltiplas assertions dificultam identificar qual falhou.

**Problema:**
```javascript
it('should process user', () => {
  const user = processUser(data);
  expect(user.name).toBe('John');
  expect(user.email).toBe('john@example.com');
  expect(user.age).toBe(30);
  expect(user.status).toBe('active');
  expect(user.credits).toBe(100);
});
```

**Solução:**
- Um assertion principal por teste
- Ou assertions relacionadas apenas

---

## 6. Testing Trivial Code

Testar getters/setters simples sem lógica.

**Problema:**
- Desperdício de tempo
- Sem valor real
- Falsa sensação de cobertura

**Solução:**
- Foco em código com lógica
- Lógica condicional, validações, regras de negócio

---

## 7. Not Validating Error Cases

Testes apenas do "caminho feliz".

**Problema:**
```javascript
it('should calculate total', () => {
  expect(calculateTotal([1, 2, 3])).toBe(6);
});
// Falta: e se amount é null? String? Negativo?
```

**Solução:**
- Testar casos de erro
- Exceções
- Valores inválidos
- Limites

---

## 8. Weak Test Names

Nomes genéricos ou pouco descritivos.

**Problema:**
```javascript
it('should work', () => {});
it('test function', () => {});
```

**Solução:**
- Descrever cenário, ação, resultado
```javascript
it('should throw error when email is empty', () => {});
it('should calculate discount for premium users', () => {});
```

---

## 9. Test Fixtures That Are Too Complex

Setup elaborado dificulta compreensão do teste.

**Problema:**
- Lógica complexa no before/after
- Difícil entender o que está sendo testado
- Difícil manutenção

**Solução:**
- Builder patterns
- Factory functions
- Dados simples no teste

---

## 10. Ignoring or Skipping Tests

Manter testes desabilitados sem razão.

**Problema:**
```javascript
xit('should validate email', () => {}); // Esquecido
```

**Solução:**
- Deletar testes inúteis
- Corrigir os que falham
- Documentar skip com razão clara

---

## 11. Testing Multiple Behaviors in One Test

"God tests" que testam múltiplas funcionalidades.

**Problema:**
- Difícil de debugar
- Falha em um comportamento quebra todo o teste

**Solução:**
- Um comportamento por teste
- Separar em múltiplos testes

---

## 12. Random Data in Tests

Usar dados aleatórios sem controle.

**Problema:**
```javascript
it('should process user', () => {
  const user = generateRandomUser();
  expect(processUser(user)).toBeDefined();
});
```

**Solução:**
- Dados determinísticos
- User fixtures conhecidos
- Propriedades builder quando necessário

---

## 13. Not Testing Edge Cases

Ignorar limites e valores extremos.

**Solução:**
- Testar 0, valores negativos, null, undefined
- String vazia, array vazio
- Valores máximos/mínimos
- off-by-one errors

---

## 14. Tightly Coupled Tests

Ordem de execução importa ou testes interferem uns nos outros.

**Problema:**
- Testes passam isolados mas falham em suite
- Não são verdadeiramente independentes

**Solução:**
- Cada teste deve ser executável isoladamente
- Limpeza própria (teardown)
- Sem dependência de estado global

---

## 15. Testing Only for Coverage Metrics

Criar testes apenas para aumentar percentual de cobertura sem valor real.

**Problema:**
```javascript
it('should create user', () => {
  const user = new User('John', 30);
  expect(user).toBeDefined(); // Teste inútil
});
```

**Consequências:**
- Falsa sensação de segurança
- Testes que não apanham erros reais
- Manutenção desnecessária
- Métrica de qualidade enganosa

**Solução:**
- Focar em cenários significativos
- Cobertura é consequência, não objetivo
- Qualidade > quantidade de testes

---

## 16. Not Using test.each for Similar Scenarios

Ter múltiplos testes separados quando deveriam ser consolidados em um `test.each`.

**Problema:**
```javascript
it('should sum 1+2', () => {
  expect(sum(1, 2)).toBe(3);
});

it('should sum 2+1', () => {
  expect(sum(2, 1)).toBe(3);
});

it('should sum 2+2', () => {
  expect(sum(2, 2)).toBe(4);
});

it('should sum 5+3', () => {
  expect(sum(5, 3)).toBe(8);
});
```

**Consequências:**
- Muitos testes repetindo a mesma lógica
- Difícil manutenção (alterar em 4 lugares)
- Output verboso e poluído
- Pouco legível

**Solução:**
```javascript
it.each([
  [1, 2, 3],
  [2, 1, 3],
  [2, 2, 4],
  [5, 3, 8]
])('should sum %i + %i = %i', (a, b, expected) => {
  expect(sum(a, b)).toBe(expected);
});
```

**Benefícios:**
- Código conciso
- Fácil adicionar novos casos
- Menos duplicação
- Melhor readabilidade

---

## 17. Unit Tests Dependent on Environment Variables

Testes unitários que dependem de configuração externa.

**Problema:**
```javascript
it('should connect to database', () => {
  const dbUrl = process.env.DATABASE_URL;
  const connection = connect(dbUrl);
  expect(connection).toBeDefined();
});
```

**Consequências:**
- Teste falha em ambiente diferente
- Difícil reproduzir localmente
- Não é verdadeiramente unitário

**Solução:**
- Mockar variáveis de ambiente
- Injete configuração como parâmetro
- Use valores fixos conhecidos

```javascript
it('should connect to database', () => {
  const connection = connect('mongodb://localhost:27017/test');
  expect(connection).toBeDefined();
});
```

---

## 18. Not Cleaning Up Mocks Between Tests

Deixar mocks ativos que afetam testes subsequentes.

**Problema:**
```javascript
it('test 1', () => {
  jest.spyOn(api, 'fetch').mockResolvedValue({ data: 'test' });
  // Falta restore()
});

it('test 2', () => {
  // Mock anterior ainda está ativo!
  const result = api.fetch();
  expect(result).toBe({ data: 'test' }); // Comportamento inesperado
});
```

**Consequências:**
- Testes interferem uns nos outros
- Comportamento não determinístico
- Falhas aleatórias

**Solução:**
```javascript
afterEach(() => {
  jest.restoreAllMocks();
});
```

Ou usar `jest.mock()` que é automaticamente limpo.

---

## 19. Blindly Accepting AI-Generated Tests

Não revisar ou questionar testes gerados por IA.

**Problema:**
- IA pode gerar testes que passam mas não validam nada real
- Testes acoplados à implementação
- Sem valor de negócio
- Manutenção técnica sem benefício

**Exemplo ruim:**
```javascript
// Gerado por IA - aceito sem crítica
it('should call function', () => {
  spyOn(obj, 'method');
  obj.method();
  expect(obj.method).toHaveBeenCalled();
});
```

**Solução:**
- Sempre revisar testes gerados
- Questionar: o que este teste valida?
- Testar comportamento, não implementação
- Rejeitar testes sem valor
- Refatorar conforme necessário

**Bom:**
```javascript
// Revisado e melhorado
it('should increase user balance when deposit is made', () => {
  const user = new Account(100);
  user.deposit(50);
  expect(user.getBalance()).toBe(150);
});
```

---

## 20. Using Weak Validations

Usar validações genéricas e fracas ao invés de assertions específicas.

**Problema:**
```javascript
it('should create user', () => {
  const user = createUser({ name: 'John', email: 'john@example.com' });
  expect(user).toBeDefined(); // Muito fraco
});

it('should calculate total', () => {
  const total = calculateTotal([1, 2, 3]);
  expect(total).toEqual(expect.any(Number)); // Genérico demais
});

it('should fetch user email', () => {
  const email = getUserEmail(1);
  expect(email).toEqual(expect.any(String)); // Qualquer string?
});
```

**Consequências:**
- Teste passa com valores inesperados
- `any(Number)` aceita -999999, Infinity, NaN
- `any(String)` aceita string vazia, "error", etc
- Não valida regras de negócio
- Falso senso de segurança

**Solução:**
```javascript
it('should create user with valid data', () => {
  const user = createUser({ name: 'John', email: 'john@example.com' });
  expect(user).toEqual({
    id: expect.any(Number),
    name: 'John',
    email: 'john@example.com',
    createdAt: expect.any(Date)
  });
});

it('should calculate total correctly', () => {
  const total = calculateTotal([1, 2, 3]);
  expect(total).toBe(6); // Valor específico
  expect(total).toBeGreaterThan(0);
});

it('should return valid user email', () => {
  const email = getUserEmail(1);
  expect(email).toMatch(/^[\w\.-]+@[\w\.-]+\.\w+$/); // Validação de email
  expect(email).not.toHaveLength(0);
});
```

**Boas Práticas:**
- Validar valores específicos, não apenas tipos
- Usar regex para emails, URLs, etc
- Validar limites e regras de negócio
- `toBe()` ao invés de `toEqual(expect.any())`

---



Anti-patterns em testes unitários reduzem a efetividade, aumentam manutenção e levam a falso senso de segurança.

A base está em testes:
- Focados e independentes
- Testando comportamento, não implementação
- Claros e descritivos
- Com valor de negócio real
- Resultados determinísticos e reproduzíveis
