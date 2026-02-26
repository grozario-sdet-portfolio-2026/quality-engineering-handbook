# Code Review e Qualidade

## Introdução

Code review é uma prática onde outros membros do time analisam e comentam sobre o código antes que ele seja integrado ao repositório principal. Muito mais do que uma ferramenta de validação técnica, o code review é um investimento em qualidade, conhecimento compartilhado e cultura de qualidade.

Um code review eficaz:

- 🎯 **Previne defeitos** – Identifica problemas antes da produção
- 📚 **Compartilha conhecimento** – Aprende com pares
- 📈 **Melhora design** – Discussão construtiva
- 🔄 **Garante consistência** – Padrões aplicados uniformemente
- 👥 **Constrói confiança** – Validação coletiva

---

## Por Que Code Review?

### Custo vs Benefício

A Lei de Boehm sobre custo de defeitos diz que quanto mais cedo um defeito é identificado, menor seu custo:

| Fase | Custo Relativo |
|------|---|
| Revisão (Code Review) | 1x |
| Teste Unitário | 3-6x |
| Teste Integração | 6-8x |
| Produção | 10-100x |

Um code review que detecta uma falha custa 1 unidade. Esse mesmo defeito custaria 10-100 vezes mais se chegasse à produção.

### Benefícios Além da Qualidade

- **Mentoria informal** – Desenvolvedores júnior aprendem vendo código senior
- **Troca de contexto** – Time fica ciente de mudanças diferentes áreas
- **Documentação viva** – Decisões técnicas ficam registradas
- **Boas práticas** – Padrões são descobertos e reforçados
- **Segurança** – Vulnerabilidades identificadas antes de merge

---

## Princípios de um Bom Code Review

### 1. Objetivo Claro

Todo code review deve ter um propósito definido:

- **Qualidade técnica** – Código está correto?
- **Testes** – Há testes adequados?
- **Design** – Arquitetura faz sentido?
- **Padrões** – Segue guia de estilo?
- **Segurança** – Há vulnerabilidades?

### 2. Respeito e Construtividade

Code review é sobre código, não sobre pessoas.

**Evitar:**
```
❌ "Seu código é ruim"
❌ "Como você não viu isso?"
❌ "Isso é óbvio"
```

**Preferred:**
```
✅ "Esse padrão pode causar race condition. Considerou usar locks?"
✅ "Interessante abordagem. Você pensou em alternativas?"
✅ "Ótima solução! Uma pequena sugestão..."
```

### 3. Foco no Essencial

Nem tudo precisa ser comentado. Priorize:

1. **Crítico** – Bugs, segurança, performance
2. **Importante** – Design, testes, padrões
3. **Menor** – Style, convenções de naming (use linters para isso)

### 4. Contexto é Rei

Entenda o contexto da atividade:

- Qual é o objetivo?
- Quais restrições existem?
- Qual é o prazo?
- É uma refatoração ou feature nova?

```
Exemplo:
PM: "Temos 1 dia para isso"
Dev: "Usei solução rápida por enquanto"
Reviewer: "Entendi. Você criou um card de refatoração futura?"
```

### 5. Conversação, Não Julgamento

Code review é bidirecional:

- Reviewers fazem perguntas, não ditam
- Autores explicam decisões
- Ambos aprendem

### 6. Timeboxing

Code reviews não devem ser interrupções o tempo inteiro.

**Recomendação:**
- Reviewers reservam blocos de tempo para reviews
- Autores não ficam bloqueados esperando (máx 24h)
- Revisitam PRs periodicamente

---

## Tipos de Code Review

### 1. Review de Correção (Bug Fix Review)

**Foco:**
- A correção de fato resolve o problema?
- Há impacto em outras partes?
- Há risco de regressão?

**Checklist:**
- ✅ Bug está descrito claramente
- ✅ Root cause identificada
- ✅ Testes de regressão adicionados
- ✅ Teste específico para o bug passou
- ✅ Documentação atualizada

### 2. Review de Feature Nova

**Foco:**
- Atende aos critérios de aceite?
- Design é escalável?
- Testes cobrem cenários?

**Checklist:**
- ✅ Requisitos cobertos
- ✅ Critérios de aceite listados
- ✅ Testes unitários adequados
- ✅ Testes de integração se necessário
- ✅ DOD/DOR descritos

### 3. Review de Refatoração

**Foco:**
- Mantém comportamento externo?
- Melhora é significativa?
- Testes continuam passando?

**Checklist:**
- ✅ Comportamento não mudou
- ✅ Testes continuam passando
- ✅ Melhoria justificada (performance, legibilidade, etc)
- ✅ Métricas verificadas (complexity, cobertura)

### 4. Review de Documentação

**Foco:**
- Está clara?
- Exemplos funcionam?
- Está atualizada?

**Checklist:**
- ✅ Redação clara e sem jargão excessivo
- ✅ Exemplos testados
- ✅ Screenshots/diagramas atuais
- ✅ Links internos funcionam

---

## O Que Procurar em um Code Review

### 1. Correção Funcional

**Perguntas:**
- O código faz o que deveria fazer?
- Há edge cases não tratados?
- Há lógica incorreta ou incompleta?

**Exemplo:**
```javascript
// ❌ Edge case não tratado
function calculateDiscount(price, percentage) {
  return price * (1 - percentage / 100);
}

// Se percentage = 150, resultado é negativo!

// ✅ Com validação
function calculateDiscount(price, percentage) {
  if (price < 0 || percentage < 0 || percentage > 100) {
    throw new Error('Invalid input');
  }
  return price * (1 - percentage / 100);
}
```

### 2. Testes

**Perguntas:**
- Há testes suficientes?
- Cobrem cenários principais?
- Cobrem edge cases e caminhos de erro?
- Testes fazem sentido?

**Exemplo:**
```javascript
// ❌ Teste fraco
it('should process user', () => {
  const result = processUser({name: 'John'});
  expect(result).toBeDefined();
});

// ✅ Teste completo
it('should process user with valid data', () => {
  const result = processUser({name: 'John', email: 'john@example.com'});
  expect(result.name).toBe('John');
  expect(result.processed).toBe(true);
});

it('should throw error when name is missing', () => {
  expect(() => processUser({email: 'john@example.com'}))
    .toThrow('Name is required');
});
```

### 3. Design e Arquitetura

**Perguntas:**
- A solução segue padrões do projeto?
- Há acoplamento excessivo?
- Responsabilidades estão bem divididas?
- É escalável?

**Exemplo:**
```javascript
// ❌ Baixa coesão, alto acoplamento
class UserManager {
  async createUser(data) {
    // Valida, salva, envia email, chama API, gera relatório...
  }
}

// ✅ Bem estruturado
class UserService {
  async createUser(data) { /* valida e save */ }
}

class NotificationService {
  async sendWelcomeEmail(user) { /* email */ }
}

class ReportingService {
  async generateUserReport(user) { /* relatório */ }
}
```

### 4. Performance

**Perguntas:**
- Há queries N+1?
- Loops desnecessários?
- Alocação excessiva de memória?
- Cálculos repetitivos?

**Exemplo:**
```javascript
// ❌ Performance ruim (N+1)
const users = await User.findAll();
for (const user of users) {
  user.orders = await Order.findByUserId(user.id); // 1 query por usuário!
}

// ✅ Com eager loading
const users = await User.findAll({ include: ['orders'] });
```

### 5. Segurança

**Perguntas:**
- Há injeção SQL?
- Validação de input adequada?
- Secrets não estão expostos?
- Acesso não autorizado é possível?

**Exemplo:**
```javascript
// ❌ Vulnerável
app.get('/user/:id', (req, res) => {
  const user = db.query(`SELECT * FROM users WHERE id = ${req.params.id}`);
  res.json(user);
});

// ✅ Seguro
app.get('/user/:id', (req, res) => {
  const userId = parseInt(req.params.id);
  if (!Number.isInteger(userId)) throw new Error('Invalid ID');
  const user = db.query('SELECT * FROM users WHERE id = ?', [userId]);
  res.json(user);
});
```

### 6. Padrões e Consistência

**Perguntas:**
- Segue o guia de estilo do projeto?
- Nomeação é consistente?
- Estrutura de pastas segue convenção?
- Há duplicação de código?

**Exemplo:**
```javascript
// ❌ Inconsistente
function getUserName(user) { }
const get_user_email = (user) => { }
const getUserAge = user => { }

// ✅ Consistente
function getUserName(user) { }
function getUserEmail(user) { }
function getUserAge(user) { }
```

### 7. Documentação

**Perguntas:**
- Código complexo está documentado?
- Comments explicam o "porquê", não o "o quê"?
- Há JSDoc/TypeDoc onde necessário?
- README foi atualizado?

**Exemplo:**
```javascript
// ❌ Documentação ruim
function calc(a, b) {
  // calcula algo
  return a * b + (a - b);
}

// ✅ Bom
/**
 * Calcula o score de qualidade baseado em métricas
 * @param {number} coverage - Cobertura de testes (0-100)
 * @param {number} complexity - Complexidade ciclomática
 * @returns {number} Score de 0-100
 */
function calculateQualityScore(coverage, complexity) {
  return coverage * 0.7 + (100 - complexity) * 0.3;
}
```

---

## Como Fazer um Bom Code Review

### Estrutura de um Code Review

#### 1. Leitura Inicial

- Leia a descrição da PR
- Entenda o contexto (issue, requisito)
- Verifique mudanças de arquivos

#### 2. Análise de Escopo

- Mudanças fazem sentido para o objetivo?
- Há mudanças não relacionadas?
- Tamanho da PR é apropriado? (< 400 linhas é ideal)

#### 3. Revisão Detalhada

Siga a ordem:

1. **Testes primeiro** – Entenda o que deve funcionar
2. **Interface pública** – Assinaturas, exports
3. **Lógica central** – Implementação principal
4. **Detalhes** – Edge cases, tratamento de erros

#### 4. Feedback Estruturado

Categorize seus comentários:

- **BLOCKER** 🔴 – Deve ser corrigido antes de merge
- **IMPORTANT** 🟠 – Deveria ser corrigido
- **SUGGESTION** 🟡 – Considerem para melhoria
- **PRAISE** 🟢 – Reconhecimento de bom código

**Exemplo:**
```
🔴 BLOCKER: Essa query é N+1. Vai impactar performance.
Sugestão: Use eager loading com include: ['orders']

🟡 SUGGESTION: Você pensou em adicionar cache aqui?
Não é crítico mas poderia melhorar performance.

🟢 PRAISE: Excelente tratamento de erros! Muito defensive.
```

#### 5. Aprovação Condicional

Use as opções disponíveis:

- **Approve** ✅ – Pode fazer merge
- **Request Changes** ❌ – Precisa corrigir
- **Comment** 💬 – Feedback sem bloquear

---

## Impacto do Code Review na Qualidade

### Métricas Importantes

1. **Taxa de defeitos detectados em review**
   - Objetivo: >50% dos bugs detectados antes de produção

2. **Tempo médio de review**
   - Objetivo: 24h (não deixar PRs antigas)

3. **Quantidade de comentários por PR**
   - Objetivo: 2-5 comentários significativos
   - Evitar: 20+ comentários (PR muito grande ou complex)

4. **Taxa de code review vs deploy**
   - Objetivo: 100% das mudanças revisadas antes de merge

5. **Tempo de resolução de feedback**
   - Objetivo: <48h entre feedback e correção

### Exemplo de Impacto

```
Antes do Code Review:
- 30% dos bugs detectados em produção
- Lead Time: 5 dias
- ReWork: 25% das atividades

Depois do Code Review implementado + padrão:
- 70% dos bugs detectados antes de produção
- Lead Time: 2 dias
- ReWork: 5% das atividades

Resultado:
- 66% menos bugs em produção
- 60% mais rápido
- 80% menos retrabalho
```

---

## Cultura de Code Review Saudável

### Mindset Correto

**Para Autores:**
- ✅ Código é do time, não pessoal
- ✅ Feedback é oportunidade de aprender
- ✅ Não precisa estar perfeito na primeira tentativa
- ✅ Pergunte se não entender o feedback

**Para Reviewers:**
- ✅ Assuma boa intenção
- ✅ Seja humilde ("Por que você escolheu essa abordagem?")
- ✅ Reconheça bom código
- ✅ Revise PRs regularmente (não ignore)

### Evitando Problemas Comuns

#### Problema: "Code Review é muito lento"

**Solução:**
- Reserve blocos para revisar (ex: 10h00 e 14h00)
- Use linters e automated checks para reduzir feedback
- Faça PRs menores
- Defina SLA de review (24h)

#### Problema: "Code Review é muito crítico"

**Solução:**
- Estabeleça cultura de respeito
- Separar BLOCKER de SUGGESTION
- Revise comentários antes de enviar
- Faça parecer que está ajudando, não julgando

#### Problema: "Ninguém está fazendo code review"

**Solução:**
- Configure regras no GitHub/GitLab (require reviews)
- Responsabilize: "Quem fez merge?"
- Inclua em métricas e retrospectiva
- Revise PRs em pares para não ficar só em um

#### Problema: "Code review pega muito tempo de senior"

**Solução:**
- Distribua revisions entre o time
- Juniores também revisam (aprendem)
- Use pair programming para grandes features
- Automate com ferramentas

---

## Checklist de Code Review

Use esse checklist para manter consistência:

### Pré-Review
- [ ] PR tem descrição clara?
- [ ] Tamanho é apropriado (< 400 LOC)?
- [ ] Commits são atômicos e com mensagens claras?

### Funcionalidade
- [ ] Código faz o que deveria?
- [ ] Edge cases estão tratados?
- [ ] Erro handling é adequado?
- [ ] Não há comportamento não documentado?

### Testes
- [ ] Há testes suficientes?
- [ ] Testes cobrem caminho feliz e erros?
- [ ] Cobertura aumentou?
- [ ] Testes são confiáveis (não flakies)?

### Design
- [ ] Segue padrões do projeto?
- [ ] Acoplamento é mínimo?
- [ ] Responsabilidades bem divididas?
- [ ] Escalável?

### Performance
- [ ] Sem N+1 queries?
- [ ] Loops desnecessários?
- [ ] Memory leaks?

### Segurança
- [ ] Sem injeção SQL/XSS?
- [ ] Input validado?
- [ ] Sem secrets exposed?
- [ ] Autenticação/autorização OK?

### Qualidade do Código
- [ ] Sem duplicação?
- [ ] Nomeação clara?
- [ ] Funções não são muito grandes?
- [ ] Complexidade ciclomática OK?

### Documentação
- [ ] Código complexo documentado?
- [ ] Comments explicam "porquê"?
- [ ] README atualizado?
- [ ] Changelog atualizado?

### Antes de Aprovar
- [ ] Testes passando?
- [ ] Linters passando?
- [ ] Sem conflitos com main?
- [ ] Pronto para produção?

---

## Conclusão

Code review não é um gatekeeper ou burocracia. É um investimento em:

1. **Qualidade** – Menos bugs em produção
2. **Conhecimento** – Time aprende continuamente
3. **Cultura** – Responsabilidade compartilhada
4. **Confiabilidade** – Validação coletiva

Quando feito bem, code review é um multiplicador de força do time, transformando-se em uma das práticas mais valiosas da engenharia de software.
