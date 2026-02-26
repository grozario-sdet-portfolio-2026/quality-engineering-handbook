# Feature Flags

## Origem e Conceito

Feature Flags (ou Feature Toggles) são mecanismos que permitem habilitar ou desabilitar recursos de uma aplicação sem necessidade de redeploy.

Originado em práticas de desenvolvimento contínuo (continuous delivery), feature flags desacoplam **deploy de código** de **release de funcionalidade**, permitindo entregas seguras e controle granular.

**Definição:** Uma Feature Flag é uma técnica que permite ativar ou desativar funcionalidades em tempo de execução, sem alterar código ou fazer novo deploy.

---

## Por Que Feature Flags?

### Problemas que resolvem:

1. **Deploy arriscado** - Deploy de nova feature = risco de quebra
2. **Rollback custoso** - Precisa fazer redeploy para desfazer
3. **Work-in-progress em main** - Código incompleto na produção
4. **Teste em produção limitado** - Não consegue testar com usuários reais
5. **Coordenação complexa** - Múltiplas equipes esperando go/nogo
6. **Feedback lento** - Só sabe se funciona quando usuário usa
7. **Blue-Green Deploy difícil** - Precisa de infraestrutura complexa

### Benefícios:

- ✅ Deploy seguro (feature desativada por padrão)
- ✅ Rollback instantâneo (desativar flag)
- ✅ Teste gradual (alguns usuários)
- ✅ Feedback de usuários reais
- ✅ Coordenação simples
- ✅ Continuous deployment com confiança

### Impacto em Números

```
Métrica                          Sem Flags    Com Flags    Melhoria
──────────────────────────────────────────────────────────────────
Deploy frequency                 1x/semana    10x/dia      70x ↑
Time to rollback                 2 horas      10 seg       720x ↓
Production incidents/mês          ~5          ~0.5        90% ↓
Customer impact per incident     1000 users  10-50 users  95% ↓
Lead time para release           2 semanas   1 dia        93% ↓
Testing done pre-production      85%         40%          (shift validation)
```

---

## Tipos de Feature Flags

### 1. Release Toggles

Habilitar/desabilitar features em desenvolvimento.

**Uso:**
- Feature em WIP pode ir para main branch
- Código incompleto não afeta produção
- Pode continuar desenvolvendo incrementalmente

**Exemplo:**

```javascript
// Feature em desenvolvimento
if (featureFlags.newPaymentSystem) {
  return useNewPaymentFlow();
} else {
  return useOldPaymentFlow();
}

// Server-side
const featureFlags = {
  newPaymentSystem: false  // TBD, desativado por padrão
};
```

**Ciclo de Vida:**

```
Sprint 1: Developer cria feature com flag = false
          ├─ Escreve testes (todos testam old flow)
          ├─ PR enviada
          ├─ Tudo passa
          └─ Merge para main (feature silenciosa)

Sprint 2: Developer continua desenvolvimento
          ├─ Escreve mais testes
          ├─ Completa feature
          └─ Merge com flag pronta

Sprint 3: Feature completa e testada
          ├─ Product manager habilita flag = true
          ├─ 10% de usuários veem nova feature
          └─ Monitora métricas
          
Tudo OK?  ├─ Gradualmente aumenta % usuários
          └─ Eventualmente deleta flag (obsoleta)

Problema? ├─ Desativa flag = false
          └─ Usuário volta ao estado anterior
```

---

### 2. Business Toggles

Controlar características de negócio (pricing, features premium).

**Uso:**
- Habilitar/desabilitar features por cliente
- A/B testing
- Ofertas limitadas por tempo

**Exemplo:**

```javascript
// Feature para usuários premium
if (featureFlags.premiumAnalytics && user.plan === 'premium') {
  return showAdvancedAnalytics();
} else {
  return showBasicAnalytics();
}

// Server-side
const featureFlags = {
  premiumAnalytics: true,
  
  // Apenas para clientes selecionados
  newDashboard: user.id in ['user123', 'user456']
};
```

**Cenário:**

```
Empresa A: premiumAnalytics = true
Empresa B: premiumAnalytics = false (test)
Empresa C: premiumAnalytics = false (different tier)
```

---

### 3. Experiment Toggles (A/B Testing)

Rotear usuários para diferentes versões.

**Uso:**
- A/B testing
- Feature experimentation
- Validar mudanças com amostra

**Exemplo:**

```javascript
// Rotear usuário para variante A ou B
const variant = getVariant(user.id, 'checkoutFlow');

if (variant === 'A') {
  return checkoutFlowA();  // Design original
} else if (variant === 'B') {
  return checkoutFlowB();  // Design novo
}

// Server-side implementação
function getVariant(userId, experiment) {
  // Consistent: mesmo usuário sempre vê mesma variante
  const bucket = userId.hashCode() % 100;
  
  if (experiment === 'checkoutFlow') {
    if (bucket < 50) return 'A';
    if (bucket < 80) return 'B';
    // 20% sem feature (control group)
  }
}
```

**Análise:**

```
Variante A (Original):    50% traffic, 2.5% conversion
Variante B (Novo design): 30% traffic, 3.1% conversion
Control:                  20% traffic, 2.4% conversion

Resultado: B statistically significantly better ✅
Decision: Promover B para 100%
```

---

### 4. Ops Toggles

Controlar comportamento operacional (caching, retry logic).

**Uso:**
- Habilitar/desabilitar cache
- Circuit breakers
- Rate limiting
- Fallback strategies

**Exemplo:**

```javascript
// Cache habilitável em runtime
if (featureFlags.enableRedisCache) {
  return cache.get(key) || 
         db.fetch(key).then(v => cache.set(key, v));
} else {
  return db.fetch(key);  // Sem cache
}

// Retry logic
const maxRetries = featureFlags.aggressiveRetry ? 5 : 2;
await retry(() => apiCall(), { maxRetries });

// Server-side
const featureFlags = {
  enableRedisCache: true,
  aggressiveRetry: false,  // Desativar se API instável
  enableCircuitBreaker: true
};
```

---

### 5. Permission Toggles

Habilitar funcionalidades por grupo de usuários.

**Uso:**
- Admin features
- Beta testing
- Gradual rollout

**Exemplo:**

```javascript
// Feature para admins
if (user.role === 'admin' && featureFlags.adminPanel) {
  return renderAdminPanel();
}

// Feature para beta testers
const isBetaTester = BETA_TESTERS.includes(user.id);
if (isBetaTester && featureFlags.betaFeatures) {
  return renderBetaFeatures();
}
```

---

## Arquitetura de Feature Flags

### Implementação Simples (In-Memory)

```javascript
// flags.js
const flags = {
  newPayment: false,
  premiumAnalytics: true,
  betaDashboard: false
};

export function isFeatureEnabled(flagName) {
  return flags[flagName] === true;
}

// usage.js
if (isFeatureEnabled('newPayment')) {
  processNewPayment();
} else {
  processOldPayment();
}
```

**Limitações:**
- Precisa redeploy para mudar flag
- Não é escalável para múltiplos ambientes

---

### Implementação Escalável (Service Externo)

```javascript
// Usar serviço dedicado (LaunchDarkly, Unleash, etc)

import { LDClient } from '@launchdarkly/node-server-sdk';

const client = new LDClient('sdk-key');

async function isFeatureEnabled(flagName, user) {
  // Busca flag remotamente
  const flagValue = await client.variation(
    flagName,
    { key: user.id, email: user.email },
    false  // default: false
  );
  
  return flagValue;
}

// usage.js
if (await isFeatureEnabled('newPayment', user)) {
  processNewPayment();
} else {
  processOldPayment();
}
```

**Fluxo:**

```
Aplicação                LaunchDarkly Service
    │                           │
    ├─ isFeatureEnabled()       │
    ├──────────────────────────>│
    │   {flag: 'newPayment',
    │    user: {id: 123, ...}}   │
    │                           │
    │  Consulta config:         │
    │  ├─ Flag habilitada?      │
    │  ├─ User qualifica?       │
    │  ├─ Porcentagem rollout?  │
    │  ├─ Experiments?          │
    │                           │
    │<──────────────────────────┤
    │   { variation: true }     │
    │                           │
    ├─ return true             │
    │
    └─ Execute feature
```

---

## Padrões Práticos

### 1. Gradual Rollout

Liberar feature lentamente para detectar problemas.

```javascript
// LaunchDarkly: variação percentual
{
  "newPayment": {
    "on": true,
    "variations": [false, true],
    "rules": [{
      "percentage": 10  // 10% de usuários
    }]
  }
}

// Timeline
Dia 1:  10% usuários → monitora 4 horas
Dia 2:  25% usuários → monitora 4 horas
Dia 3:  50% usuários → monitora 1 dia
Dia 4: 100% usuários → monitora 2 dias
Dia 5: Desativar flag (obsoleta)
```

**Monitoramento:**

```
Métrica                10%        25%        50%       100%
─────────────────────────────────────────────────────────
Erro rate          0.01% ✅   0.02% ✅    0.05% ⚠️   0.05% ⚠️
Latência P99       150ms ✅   155ms ✅    200ms ⚠️   210ms ⚠️
Taxa conversão     2.5% ✅    2.51% ✅    2.49% ✅   2.48% ✅
CPU                45% ✅     48% ✅      52% ⚠️    58% ⚠️
```

---

### 2. Ring Deployment

Liberar em círculos concêntricos: dev → staging → canary → prod.

```
Dev Environment:
└─ Feature ON (devs trabalham com nova feature)

Staging:
└─ Feature ON (testes completos)

Production Canary (1% servers):
├─ Feature ON para 5% usuários
└─ 1-2 horas monitoramento

Production Stable (99% servers):
├─ Feature ON para 25% usuários
└─ 1 dia monitoramento

Production Full (100% servers):
└─ Feature ON para 100% usuários
```

---

### 3. Dark Launch

Rodar nova feature junto com antiga sem mostrar para usuário.

```javascript
// Rodar feature nova, descartar resultado
async function saveData(data) {
  // Path antigo (resultado é usado)
  const oldResult = await saveToOldSystem(data);
  
  // Path novo (resultado descartado, apenas testamos)
  if (featureFlags.darkLaunchNewSystem) {
    try {
      await saveToNewSystem(data);
      metrics.recordSuccess('newSystem');
    } catch (e) {
      metrics.recordError('newSystem', e);
    }
  }
  
  return oldResult;  // Usuário nunca vê novo sistema
}

// Após confiante que novo sistema funciona:
// Trocar para usar resultado do novo sistema
```

**Timeline:**

```
Semana 1-2: Dark launch
  ├─ Novo sistema rodar em BG
  ├─ Monitora erros (none expected)
  └─ Compara resultados com antigo

Semana 2-3: Validação
  ├─ Verificar se ambos retornam mesmo resultado
  ├─ Testes com dados reais passando
  └─ Segurança aumentada

Semana 3-4: Shadow launch (% usuários)
  ├─ 5% usuários veem novo sistema (real)
  ├─ Resto ainda vê antigo como fallback
  └─ Monitora métricas reais

Semana 4: Full rollout
  ├─ 100% usuários em novo sistema
  └─ Remover código antigo
```

---

### 4. Feature Licensing

Controlar features por tipo de licença.

```javascript
const userPlan = user.subscription.plan;

const featureAccess = {
  'free': {
    analytics: true,
    export: false,
    api: false,
    customBranding: false
  },
  'pro': {
    analytics: true,
    export: true,
    api: false,
    customBranding: false
  },
  'enterprise': {
    analytics: true,
    export: true,
    api: true,
    customBranding: true
  }
};

const planFeatures = featureAccess[userPlan];

if (planFeatures.export) {
  showExportButton();
}
```

---

## Integração com Testes

### Testar Feature Flags

```javascript
describe('Feature Flags', () => {
  // Mock feature flag
  beforeEach(() => {
    mockFeatureFlag('newPayment', true);
  });

  it('should show new payment button when flag ON', () => {
    const { getByText } = render(<CheckoutPage />);
    expect(getByText('New Payment Method')).toBeInTheDocument();
  });

  it('should show old payment button when flag OFF', () => {
    mockFeatureFlag('newPayment', false);
    const { getByText } = render(<CheckoutPage />);
    expect(getByText('Legacy Payment')).toBeInTheDocument();
  });
});

// Teste ambos os caminhos
describe.each([
  ['newPayment', true],
  ['newPayment', false]
])('Checkout with %s = %s', (flag, value) => {
  beforeEach(() => mockFeatureFlag(flag, value));

  it('should process checkout', () => {
    // teste que funciona com ambos valores
  });
});
```

---

### Test Coverage com Flags

```javascript
// Matriz de teste: cobertura de todos os paths

Coverage Map:
├─ newPayment = ON
│  ├─ Test 1: Happy path ✅
│  ├─ Test 2: Error handling ✅
│  └─ Test 3: Edge cases ✅
│
└─ newPayment = OFF
   ├─ Test 4: Happy path ✅
   ├─ Test 5: Error handling ✅
   └─ Test 6: Edge cases ✅

Total coverage: 6/6 ramos testados
```

---

## Boas Práticas

### 1. Flag por Feature, não por Trecho

```javascript
// ❌ Flags espalhadas
if (featureFlags.paymentEnabled) {
  validatePayment();
}

if (featureFlags.paymentEnabled) {
  processPayment();
}

if (featureFlags.paymentEnabled) {
  sendReceipt();
}

// ✅ Flag centralizado
function processCheckout(cart) {
  if (!featureFlags.newPaymentSystem) {
    return legacyCheckout(cart);
  }
  
  const payment = validatePayment(cart);
  processPayment(payment);
  sendReceipt(payment);
}
```

---

### 2. Cleanup Pós-Deploy

Remover flags obsoletas.

```javascript
// After feature rolling out 100% for 2 weeks:
// DELETE: featureFlags.newPayment

// Antes de limpar:
// 1. Garantir 100% em produção
// 2. Remover todas as configurações no server
// 3. Remover du código-fonte
// 4. Atualizar documentação

// Checklist de limpeza
□ Feature está 100% ativo em produção
□ Zero erros nos últimos 7 dias
□ Código antigo pode ser removido
□ Flags deletadas do config service
□ Tests atualizados
□ Documentação atualizada
```

---

### 3. Naming Convention

Nomes claros e descritivos.

```javascript
// ❌ Confuso
const flags = {
  f1: true,
  enabled_new_stuff: true,
  MAGIC_FLAG_2024: false
};

// ✅ Claro
const flags = {
  newPaymentSystem: true,      // Release toggle
  premiumUserAnalytics: true,  // Business toggle
  checkoutExperiment: false,   // Experiment toggle
  redisCache: true             // Ops toggle
};
```

---

### 4. Documentação

Manter registro de each flag.

```markdown
# Feature Flags Registry

## newPaymentSystem
- **Type:** Release Toggle
- **Owner:** PaymentTeam
- **Created:** 2024-01-15
- **Status:** In Rollout (25% traffic)
- **Plan:** Reach 100% by 2024-02-15
- **Cleanup Date:** 2024-03-01
- **Monitoring:** [Dashboard Link]
- **Rollback Plan:** Disable flag immediately
- **Related PRs:** #1234, #1245

## premiumUserAnalytics
- **Type:** Business Toggle
- **Owner:** ProductTeam
- **Created:** 2023-12-01
- **Status:** Stable (100% traffic)
- **Permanent:** YES (parte da business model)
- **Related Docs:** [Pricing](pricing.md)
```

---

## Monitoramento e Métricas

### Observabilidade de Flags

```javascript
const metrics = {
  // Cobertura
  activeFlags: 5,
  expiredFlags: 0,        // Flags que deveriam estar limpas
  orphanedFlags: 2,       // Criadas mas nunca usadas
  
  // Performance
  flagLatency: '2ms',     // Tempo para avaliar flag
  cacheHitRate: '99.5%',  // Quantas vezes foi cached
  
  // Rollout
  newPaymentTraffic: '25%',
  newPaymentErrors: '0.02%',
  newPaymentLatency: '150ms',
  
  // Qualidade
  flagChangeFrequency: 'daily',
  unplannedRollbacks: 0,
  timeSinceLastCleanup: '15 days'
};
```

---

## Ferramentas Populares

### Serviços Gerenciados

| Tool | Características |
|------|-----------------|
| **LaunchDarkly** | Completo, A/B testing, analytics |
| **Unleash** | Open source, self-hosted, simples |
| **Split.io** | Especializado em experimentation |
| **Optimizely** | Enterprise, completo |
| **Flagsmith** | Open source, ótimo para MVP |

### Implementação Caseira

```javascript
// flags-service.js - Implementação simples
const flags = {
  newPayment: { value: false, rolloutPercentage: 0 },
  premiumAnalytics: { value: true, rolloutPercentage: 100 }
};

export async function evaluateFlag(flagName, context) {
  const flag = flags[flagName];
  if (!flag) return false;
  
  // Check rollout percentage
  if (flag.rolloutPercentage < 100) {
    const hash = hashUserId(context.userId);
    const userBucket = hash % 100;
    return userBucket < flag.rolloutPercentage;
  }
  
  return flag.value;
}

function hashUserId(userId) {
  // Consistent hashing: mesmo userId sempre mesmo bucket
  return userId.split('').reduce((h, c) => h + c.charCodeAt(0), 0);
}
```

---

## Conexão com Outras Practices

### Com Test Pipelines

Feature flags permitem deploy seguro sem bloquear pipeline:

```
Deploy com Feature Flag OFF → Tudo passa
  ├─ Sem impacto em produção
  ├─ Código novo dormindo
  └─ Sem riscos

Quando pronto:
  └─ Apenas ativar flag (não precisa redeploy)
```

---

### Com Shift-Left

Feature flags + Shift-Left = qualidade máxima:

```
Durante desenvolvimento:
├─ Feature flag = OFF (não afeta produção)
├─ TDD (testes guiam desenvolvimento)
├─ Code review (qualidade do código)
└─ PR validações (linter, coverage, etc)

No deploy:
├─ Feature flag = OFF (seguro)
├─ Monitor dark launch
└─ Depois ON para alguns
```

---

### Com Definition of Done

Adicionar ao checklist:

```
Definition of Done
✅ Code review aprovado
✅ Testes com 80%+ cobertura
✅ Feature flag implementada
✅ Dark launch testado
✅ Monitoring configurado
✅ Runbook de rollback pronto
✅ Documentação atualizada
```

---

## Conclusão

Feature Flags são ferramentas poderosas para Continuous Delivery com confiança. Permitem:

- ✅ Deploy seguro e frequente
- ✅ Rollback instantâneo
- ✅ Teste com usuários reais
- ✅ Coordenação simplificada
- ✅ Continuous experimentation

Combine com Test Pipelines e Pull Request Testing para ciclo de desenvolvimento robusto e rápido.
