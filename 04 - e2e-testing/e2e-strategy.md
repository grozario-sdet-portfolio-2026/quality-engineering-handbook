# End-to-End Testing Strategy

## Origem e Conceito

Testes End-to-End (E2E) validam fluxos completos de usuário através de toda aplicação, simulando interações reais.

Diferente de testes unitários e integração que testam partes isoladas, E2E testa o sistema como um todo do ponto de vista do usuário.

**Definição:** Testes E2E validam jornadas completas de usuário em ambiente que simula produção, incluindo UI, API, BD e dependências externas.

---

## Por Que Testes E2E?

### Problemas que resolvem:

1. **Fluxos quebrados** - Função individual funciona, mas fluxo não
2. **Integração completa** - Não apenas partes, sistema inteiro
3. **Problemas de UI** - Clicks, navegação, renderização
4. **Regressões** - Código antigo quebrando funcionalidades
5. **Confiança em deploys** - Validar antes de produção
6. **Experiência do usuário** - Não apenas lógica, mas usabilidade
7. **Cenários críticos** - Jornadas de negócio essenciais

### Benefícios:

- ✅ Valida sistema completo
- ✅ Detecta problemas de integração complex
- ✅ Teste realista do usuário
- ✅ Confiança em produção
- ✅ Documentação de fluxos críticos

### Limitações:

- ❌ Lento
- ❌ Frágil a mudanças de UI
- ❌ Difícil debugar falhas
- ❌ Caro em tempo de execução
- ❌ Flakey se não bem estruturado

---

## Escopo de Testes E2E

### O que testar em E2E:

- ✅ Jornadas críticas de negócio
- ✅ Fluxos de usuário completos
- ✅ Interações UI-API-BD
- ✅ Casos de erro principal
- ✅ Autenticação/autorização
- ✅ Integração com 3º partes

### O que NÃO testar em E2E:

- ❌ Validações triviais
- ❌ Lógica pura (math, strings)
- ❌ Todos os edge cases
- ❌ Performance micro
- ❌ Tudo que é coberto por unitário/integração

---

## Tipos de Testes E2E

### 1. Testes de Fluxo de Usuário (Happy Path)

Cenário ideal onde tudo funciona.

**Exemplo:**

```javascript
describe('User Signup E2E', () => {
  it('should complete signup flow', async () => {
    // 1. Navegar para página
    await page.goto('http://localhost:3000/signup');

    // 2. Preencher formulário
    await page.fill('input[name="name"]', 'John Doe');
    await page.fill('input[name="email"]', 'john@example.com');
    await page.fill('input[name="password"]', 'SecurePass123');

    // 3. Submeter
    await page.click('button[type="submit"]');

    // 4. Validar sucesso
    await page.waitForNavigation();
    expect(page.url()).toContain('/dashboard');
    
    // 5. Validar BD
    const user = await db.query('SELECT * FROM users WHERE email = ?', ['john@example.com']);
    expect(user).toHaveLength(1);
  });
});
```

**Características:**
- Testa fluxo principal
- Valida do início ao fim
- Importante para confiança

---

### 2. Testes de Erro e Validação

Cenários onde algo dá errado.

**Exemplo:**

```javascript
describe('Form Validation E2E', () => {
  it('should show error for invalid email', async () => {
    await page.goto('http://localhost:3000/signup');
    await page.fill('input[name="email"]', 'invalid-email');
    await page.click('button[type="submit"]');

    const error = await page.textContent('.error-message');
    expect(error).toContain('Invalid email');
  });

  it('should show error for duplicate email', async () => {
    // Setup: user já existe
    await db.insert('users', { email: 'existing@example.com' });

    await page.goto('http://localhost:3000/signup');
    await page.fill('input[name="email"]', 'existing@example.com');
    await page.click('button[type="submit"]');

    const error = await page.textContent('.error-message');
    expect(error).toContain('Email already exists');
  });

  it('should prevent submission with empty fields', async () => {
    await page.goto('http://localhost:3000/signup');
    
    const button = await page.$('button[type="submit"]');
    const isDisabled = await button.getAttribute('disabled');
    
    expect(isDisabled).toBeTruthy();
  });
});
```

---

### 3. Testes de Integração UI-API

Validar que UI e API trabalham juntas.

**Exemplo:**

```javascript
describe('Order Creation E2E', () => {
  it('should create order and update inventory', async () => {
    // 1. Login
    await page.goto('http://localhost:3000/login');
    await page.fill('input[name="email"]', 'user@example.com');
    await page.fill('input[name="password"]', 'password');
    await page.click('button[type="submit"]');
    await page.waitForNavigation();

    // 2. Navegar para produto
    await page.goto('http://localhost:3000/products/laptop-123');

    // 3. Validar stock display
    const stock = await page.textContent('[data-testid="stock"]');
    expect(stock).toContain('10 in stock');

    // 4. Adicionar ao carrinho
    await page.click('button:has-text("Add to Cart")');
    
    const cartCount = await page.textContent('[data-testid="cart-count"]');
    expect(cartCount).toBe('1');

    // 5. Ir para checkout
    await page.click('[data-testid="cart"]');
    await page.click('button:has-text("Checkout")');

    // 6. Submeter order
    await page.click('button:has-text("Place Order")');
    await page.waitForNavigation();

    // 7. Validar confirmação
    expect(page.url()).toContain('/order-confirmation');
    const orderNumber = await page.textContent('[data-testid="order-number"]');
    expect(orderNumber).toBeDefined();

    // 8. Validar BD: order criada
    const order = await db.query('SELECT * FROM orders ORDER BY id DESC LIMIT 1');
    expect(order[0].status).toBe('pending');

    // 9. Validar BD: inventory atualizado
    const product = await db.query('SELECT * FROM products WHERE id = ?', ['laptop-123']);
    expect(product[0].stock).toBe(9);
  });
});
```

---

### 4. Testes de Autenticação/Autorização

Validar segurança e acesso.

**Exemplo:**

```javascript
describe('Auth E2E', () => {
  it('should not access dashboard without login', async () => {
    await page.goto('http://localhost:3000/dashboard');
    // Redireciona para login
    expect(page.url()).toContain('/login');
  });

  it('should login and persist session', async () => {
    await page.goto('http://localhost:3000/login');
    await page.fill('input[name="email"]', 'user@example.com');
    await page.fill('input[name="password"]', 'password');
    await page.click('button[type="submit"]');
    await page.waitForNavigation();

    expect(page.url()).toContain('/dashboard');

    // Refresh page - session persiste
    await page.reload();
    expect(page.url()).toContain('/dashboard');
  });

  it('should prevent admin access to regular user', async () => {
    // Login como user regular
    await loginAs('user@example.com');

    // Tentar acessar admin
    await page.goto('http://localhost:3000/admin');

    // Sem acesso
    expect(page.url()).not.toContain('/admin');
    expect(await page.textContent('body')).toContain('Access Denied');
  });
});
```

---

### 5. Testes de Jornada Multipasso

Fluxos com múltiplas etapas.

**Exemplo:**

```javascript
describe('Payment Flow E2E', () => {
  it('should complete full payment journey', async () => {
    // Step 1: Add product to cart
    await goToProduct('laptop-123');
    await page.click('button:has-text("Add to Cart")');

    // Step 2: Apply coupon
    await page.goto('http://localhost:3000/cart');
    await page.fill('input[name="coupon"]', 'SAVE10');
    await page.click('button:has-text("Apply")');
    
    const discount = await page.textContent('[data-testid="discount"]');
    expect(discount).toContain('-$99.90');

    // Step 3: Checkout
    await page.click('button:has-text("Checkout")');

    // Step 4: Shipping
    await page.fill('input[name="address"]', '123 Main St');
    await page.selectOption('select[name="shipping"]', 'express');

    // Step 5: Payment
    await page.fill('input[name="cardNumber"]', '4111111111111111');
    await page.fill('input[name="cvv"]', '123');
    
    // Step 6: Confirm
    await page.click('button:has-text("Pay")');
    await page.waitForNavigation();

    // Validar confirmação
    expect(page.url()).toContain('/order-confirmation');
  });
});
```

---

## Estratégias de Implementação

### 1. Pyramid Approach (Recomendado)

Poucos E2E, muitos testes inferiores.

```
  E2E (5-10 testes)
  ↓
  Integration (30-40 testes)
  ↓
  Unit (70-80 testes)
```

**Benefícios:**
- ✅ Rápido (maioria é rápida)
- ✅ Confiável (menos flaky)
- ✅ Fácil manter
- ✅ Coverage alta

---

### 2. Critical Path Testing

E2E apenas para fluxos críticos de negócio.

**Exemplo:**

```javascript
describe('E2E Critical Paths', () => {
  // CP1: User signup
  it('critical: user should signup', async () => { /**/ });

  // CP2: User login
  it('critical: user should login', async () => { /**/ });

  // CP3: Purchase order
  it('critical: user should place order', async () => { /**/ });

  // CP4: Refund
  it('critical: user should get refund', async () => { /**/ });

  // Não testamos: theme switching, language change, etc
});
```

**Vantagem:**
- ✅ Foco no que importa
- ✅ Menos testes, mais valor
- ✅ Rápidos

---

### 3. Smoke Testing

Testes rápidos validando sistema basic rodando.

```javascript
describe('Smoke Tests E2E', () => {
  it('should load homepage', async () => {
    await page.goto('http://localhost:3000');
    expect(page.url()).toContain('localhost:3000');
  });

  it('should have navigation', async () => {
    const nav = await page.$('nav');
    expect(nav).toBeTruthy();
  });

  it('should search products', async () => {
    await page.fill('input[type="search"]', 'laptop');
    await page.press('input', 'Enter');
    
    const results = await page.$$('.product-card');
    expect(results.length).toBeGreaterThan(0);
  });
});
```

---

## Ambientes de Teste E2E

### 1. Local Environment

Rodando localmente.

```bash
# Start app locally
npm start

# Run tests against local
npm run test:e2e -- --base-url http://localhost:3000
```

**Vantagens:**
- ✅ Debug fácil
- ✅ Rápido iteração

**Desvantagens:**
- ❌ Não representa prod
- ❌ Variações entre máquinas

---

### 2. Staging Environment

Ambiente que copia produção.

```bash
# Deploy to staging
npm run deploy:staging

# Run tests
npm run test:e2e -- --base-url https://staging.example.com
```

**Vantagens:**
- ✅ Próximo de produção
- ✅ Compartilhado pelo time
- ✅ Tests determinísticos

**Desvantagens:**
- ❌ Mais lento que local
- ❌ Pode afetar outros testes

---

### 3. Docker Environment

Containers simulando stack completa.

```yaml
# docker-compose.test.yml
version: '3'
services:
  app:
    build: .
    ports:
      - "3000:3000"
  
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: test

  redis:
    image: redis:7
```

```bash
docker-compose -f docker-compose.test.yml up
npm run test:e2e
```

---

### 4. CI/CD Environment

Em pipeline de CI.

```yaml
# GitHub Actions
name: E2E Tests

on: [push, pull_request]

jobs:
  e2e:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Start services
        run: docker-compose -f docker-compose.test.yml up -d
      
      - name: Wait for services
        run: npm run wait:services
      
      - name: Run E2E tests
        run: npm run test:e2e
      
      - name: Upload videos
        if: failure()
        uses: actions/upload-artifact@v2
        with:
          name: cypress-videos
          path: cypress/videos
```

---

## Anti-Patterns em Testes E2E

### 1. Testar Tudo em E2E

Duplicar cobertura de unitário/integração.

```javascript
// ❌ Ruim: E2E testando lógica trivial
it('should calculate discount', async () => {
  await page.fill('input[name="total"]', '100');
  const discount = await page.textContent('[data-testid="discount"]');
  expect(discount).toBe('10'); // Math, deveria ser unitário
});

// ✅ Bom: E2E testando fluxo
it('should apply discount on checkout', async () => {
  await addProductToCart('laptop');
  await page.click('[data-testid="apply-coupon"]');
  await page.fill('input[name="coupon"]', 'SAVE10');
  const total = await page.textContent('[data-testid="total"]');
  expect(total).toContain('$899.90'); // Total correto
});
```

---

### 2. Dados Compartilhados Entre Testes

Estado compartilhado causa fragilidade.

```javascript
// ❌ Problema
let userId;

it('test 1: create user', async () => {
  // ...
  userId = 123; // Salva estado global
});

it('test 2: delete user', async () => {
  // Depende de test 1 ter rodado
  await deleteUser(userId);
});

// ✅ Isolado
it('test 1: create user', async () => {
  const userId = await createTestUser();
  // ...
  await cleanupUser(userId);
});

it('test 2: delete user', async () => {
  const userId = await createTestUser();
  await deleteUser(userId);
  // ...
});
```

---

### 3. Testes Muito Frágeis

Acoplados a detalhes de UI.

```javascript
// ❌ Frágil: class específica
it('should show success', async () => {
  await page.click('button.primary-btn-large-blue');
  const msg = await page.$('.success-msg-container-v2');
});

// ✅ Robusto: data-testid
it('should show success', async () => {
  await page.click('[data-testid="submit-button"]');
  const msg = await page.textContent('[data-testid="success-message"]');
  expect(msg).toContain('Success');
});
```

---

### 4. Sem Waits Apropriados

Click e esperar imediato sem aguardar.

```javascript
// ❌ Problema: não espera
it('should load results', async () => {
  await page.fill('input[name="search"]', 'laptop');
  await page.press('input', 'Enter');
  const results = await page.$$('.result'); // Pode não carregar
});

// ✅ Com espera
it('should load results', async () => {
  await page.fill('input[name="search"]', 'laptop');
  await page.press('input', 'Enter');
  
  // Espera elemento aparecer
  await page.waitForSelector('.result');
  
  const results = await page.$$('.result');
  expect(results.length).toBeGreaterThan(0);
});
```

---

### 5. Resetar Estado

Deixar testes sujando BD.

```javascript
// ❌ Não limpa
afterAll(() => {
  /* Nothing */
});

// ✅ Cleanup
afterEach(async () => {
  // Limpar dados criados
  await db.query('DELETE FROM users WHERE email LIKE ?', ['%test%']);
  await db.query('DELETE FROM orders');
});
```

---

## Best Practices em Testes E2E

### 1. Usar Page Object Model (POM)

Encapsular UI sem duplication.

```javascript
// pages/LoginPage.js
class LoginPage {
  constructor(page) {
    this.page = page;
    this.emailInput = 'input[name="email"]';
    this.passwordInput = 'input[name="password"]';
    this.submitButton = 'button[type="submit"]';
  }

  async goto() {
    await this.page.goto('http://localhost:3000/login');
  }

  async login(email, password) {
    await this.page.fill(this.emailInput, email);
    await this.page.fill(this.passwordInput, password);
    await this.page.click(this.submitButton);
    await this.page.waitForNavigation();
  }
}

// Uso em teste
describe('Login E2E', () => {
  it('should login successfully', async () => {
    const loginPage = new LoginPage(page);
    await loginPage.goto();
    await loginPage.login('user@example.com', 'password');
    
    expect(page.url()).toContain('/dashboard');
  });
});
```

**Benefícios:**
- ✅ Sem duplicação
- ✅ Fácil manutenção
- ✅ Reutilizável

---

### 2. Usar Test Data Builders

Preparar dados consistentemente.

```javascript
class TestDataBuilder {
  static async createUser(overrides = {}) {
    const user = {
      name: 'John Doe',
      email: `user-${Date.now()}@example.com`,
      password: 'Password123',
      status: 'active',
      ...overrides
    };

    // Criar no BD
    await db.insert('users', user);
    return user;
  }

  static async createOrder(userId, overrides = {}) {
    const order = {
      user_id: userId,
      total: 100,
      status: 'pending',
      ...overrides
    };

    await db.insert('orders', order);
    return order;
  }
}

// Uso
it('should fulfill order', async () => {
  const user = await TestDataBuilder.createUser();
  const order = await TestDataBuilder.createOrder(user.id);

  // Test...
});
```

---

### 3. Usar data-testid

Selectors específicos para testes.

```html
<!-- HTML -->
<button class="btn btn-primary btn-large" data-testid="submit-button">
  Submit
</button>
```

```javascript
// Teste
await page.click('[data-testid="submit-button"]');
```

**Benefícios:**
- ✅ Não quebra com CSS
- ✅ Intenção clara
- ✅ Fácil manutenção

---

### 4. Testes Focados e Rápidos

Cada teste valida UMA coisa.

```javascript
// ❌ Muito longo
it('should complete entire flow', async () => {
  // Signup
  // Login
  // Create product
  // Update product
  // Delete product
});

// ✅ Focado
it('should complete signup', async () => {
  // Apenas signup
});

it('should create product', async () => {
  // Apenas create (com user já logado)
});
```

---

### 5. Usar Fixtures de Dados

Dados pré-preparados.

```javascript
// fixtures/users.json
{
  "admin": {
    "email": "admin@example.com",
    "password": "AdminPassword123",
    "role": "admin"
  },
  "user": {
    "email": "user@example.com",
    "password": "UserPassword123",
    "role": "user"
  }
}

// Uso
const fixtures = require('../fixtures/users.json');

beforeEach(async () => {
  await db.insert('users', fixtures.user);
});

it('should login as user', async () => {
  await loginPage.login(fixtures.user.email, fixtures.user.password);
});
```

---

### 6. Capturar Screenshots em Falhas

Debugar mais fácil.

```javascript
afterEach(async () => {
  if (test.failed) {
    await page.screenshot({ path: `screenshots/${test.title}.png` });
  }
});
```

---

### 7. Validar Visualmente Changes

Screenshots para regressão visual.

```javascript
it('checkout page layout unchanged', async () => {
  await page.goto('http://localhost:3000/checkout');
  
  // Screenshots por componente
  const header = await page.$('header');
  await header.screenshot({ path: 'checkout-header.png' });
});
```

---

### 8. Separar por Feature/User Journey

Organização clara.

```
tests/e2e/
  auth/
    login.spec.js
    signup.spec.js
    logout.spec.js
  checkout/
    add-to-cart.spec.js
    apply-coupon.spec.js
    payment.spec.js
  admin/
    manage-products.spec.js
```

---

## Ferramentas de Testes E2E

### Cypress

Moderno, debugging excelente.

```javascript
describe('E2E Tests', () => {
  it('should login', () => {
    cy.visit('http://localhost:3000/login');
    cy.get('input[name="email"]').type('user@example.com');
    cy.get('input[name="password"]').type('password');
    cy.get('button[type="submit"]').click();
    cy.url().should('include', '/dashboard');
  });
});
```

**Pros:**
- ✅ Time-travel debugging
- ✅ Excellent documentation
- ✅ Great DX

**Cons:**
- ❌ Só browser (não mobile)
- ❌ JavaScript only

---

### Playwright

Rápido, suporte multi-browser.

```javascript
const { browser, page } = require('@playwright/test');

test('should login', async () => {
  const browser = await chromium.launch();
  const page = await browser.newPage();
  
  await page.goto('http://localhost:3000/login');
  await page.fill('input[name="email"]', 'user@example.com');
  await page.fill('input[name="password"]', 'password');
  await page.click('button[type="submit"]');
  
  await expect(page).toHaveURL(/.*dashboard/);
  await browser.close();
});
```

**Pros:**
- ✅ Multi-browser (Chrome, Firefox, Safari)
- ✅ Fast
- ✅ Mobile testing

**Cons:**
- ❌ Menos vibrante que Cypress

---

### WebdriverIO

Padrão W3C, linguagem agnóstica.

```javascript
describe('Login', () => {
  it('should login', () => {
    browser.url('http://localhost:3000/login');
    $('input[name="email"]').setValue('user@example.com');
    $('input[name="password"]').setValue('password');
    $('button[type="submit"]').click();
    
    expect(browser).toHaveUrlContaining('/dashboard');
  });
});
```

---

### Selenium

Clássico, suporte amplo.

```javascript
const driver = new Builder()
  .forBrowser('chrome')
  .build();

await driver.get('http://localhost:3000/login');
await driver.findElement(By.name('email')).sendKeys('user@example.com');
// ...
```

---

## Estrutura de Testes E2E Recomendada

```javascript
describe('E2E - Checkout Flow', () => {
  let page;
  let testUser;

  beforeAll(async () => {
    browser = await chromium.launch();
    page = await browser.newPage();
  });

  beforeEach(async () => {
    // Setup: criar dados
    testUser = await TestDataBuilder.createUser();
    await cleanupTestData();
  });

  afterEach(async () => {
    // Cleanup: apagar dados
    await db.query('DELETE FROM users WHERE id = ?', [testUser.id]);
  });

  afterAll(async () => {
    await browser.close();
  });

  describe('Happy Path', () => {
    it('should complete checkout', async () => {
      // Login
      const loginPage = new LoginPage(page);
      await loginPage.goto();
      await loginPage.login(testUser.email, testUser.password);

      // Add to cart
      const productPage = new ProductPage(page);
      await productPage.goto('laptop-123');
      await productPage.addToCart();

      // Checkout
      const checkoutPage = new CheckoutPage(page);
      await checkoutPage.goto();
      await checkoutPage.fillShipping({ address: '123 Main' });
      await checkoutPage.fillPayment({ card: '4111111111111111' });
      await checkoutPage.submit();

      // Verify
      expect(page.url()).toContain('/order-confirmation');
    });
  });

  describe('Error Cases', () => {
    it('should show error for invalid card', async () => {
      // ...
      await checkoutPage.fillPayment({ card: '1234567890' });
      const error = await page.textContent('[data-testid="card-error"]');
      expect(error).toContain('Invalid card');
    });
  });
});
```

---

## Conclusão

Testes E2E são essenciais para:

- ✅ Validar jornadas completas de usuário
- ✅ Detectar problemas de integração
- ✅ Confiança antes de produção
- ✅ Experiência real do usuário

**Pontos-chave:**

| Aspecto | Recomendação |
|---------|-------------|
| **Escopo** | Apenas fluxos críticos (5-10 testes) |
| **Estratégia** | Critical path + smoke tests |
| **Padrão** | Page Object Model |
| **Dados** | Test builders, isolados por teste |
| **Ferramenta** | Cypress (dev) ou Playwright (CI/CD) |
| **Estrutura** | Data-testid, fixtures, cleanup |
| **Ambiente** | Staging ou Docker |
| **Integração** | CI/CD em cada push |

Testes E2E completam a pirâmide ao validar que o sistema inteiro funciona do ponto de vista do usuário.
