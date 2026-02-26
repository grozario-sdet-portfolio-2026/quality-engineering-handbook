# Ownership

## Origem e Conceito

Ownership em engenharia refere-se ao senso de responsabilidade e autonomia que um time ou indivíduo tem sobre um produto, sistema ou componente.

Ownership não é apenas um título ou uma atribuição, é um **mindset** onde a pessoa sente que aquele código/produto é "seu" e se preocupa com qualidade, manutenção e melhoria contínua.

**Definição:** Ownership é o modelo organizacional onde times e indivíduos têm clara responsabilidade, autoridade e autonomia sobre partes específicas do sistema, incluindo qualidade, manutenção e evolution.

---

## Por Que Ownership?

### Problemas que a Falta Cria

#### Sem Ownership Claro

```
Cenário: Produção com bug crítico

Bug encontrado em payment.js

Dev A: "Não sou eu, eu toquei em algo relacionado"
Dev B: "Mas foi Dev A quem mexeu último"
Dev C: "Eu não mexi nisso"

Resultado:
├─ Ninguém assume
├─ Ninguém iniciativa para fix
├─ Culpa é jogada
├─ Bug fica horas sem fix
└─ Clientes afetados por mais tempo

MTTR: 4+ horas 😞
```

#### Com Ownership Claro

```
Cenário: Produção com bug crítico

Bug encontrado em payment.js

Dev A: "Esse é meu componente, eu faço um hotfix now"
├─ Avalia problema rápido
├─ Faz fix em 10 minutos
├─ Pessoa X revisa (super rápido)
├─ Deploy em 5 minutos
└─ Alerta cliente em 20 minutos

MTTR: 20 minutos ✅
Post-mortem: Dev A planeja como evitar futuramente
```

### Impacto em Números

```
Métrica                        Sem Ownership    Com Ownership    Melhoria
──────────────────────────────────────────────────────────────────────
MTTR                           4+ horas         20 minutos       92% ↓
Bugs não atribuídos            40%              0%               100% ↓
Tempo para investigação        2-3h             5 min            95% ↓
Team accountability            LOW              HIGH             -
Code maintainability           Baixa            Alta             +
Proatividade em melhoria       Baixa            Alta             +
```

---

## Tipos de Ownership

### 1. Feature Ownership (Squad)

Squad é dono de uma feature ou conjunto de features.

```
Squad: Pagamento
├─ Dono: Tech Lead (Dev Senior)
├─ Membros: 4 Devs, 1 QA
├─ Responsável por:
│  ├─ Qualidade do código
│  ├─ Performance
│  ├─ Escalabilidade
│  ├─ Manutenção
│  ├─ On-call (incidents)
│  └─ Evolução contínua
│
├─ Autoridade:
│  ├─ Decidir tecnologia
│  ├─ Design de arquitetura
│  ├─ Prioridade de bugs
│  └─ Refatoração
│
└─ Autonomia:
   ├─ Planejamento
   ├─ Execução
   ├─ Deploy (com gates)
   └─ Decisões técnicas

Modelo: Squads multidisciplinares
Escalabilidade: ~5-9 pessoas por squad
Benefício: Contexto completo, ciclos curtos
```

---

### 2. Component Ownership (Especialista)

Pessoa é especialista em um componente crítico.

```
Componente: Payment Gateway Integration
├─ Dono: Dev Sênior (especialista)
├─ Responsabilidades:
│  ├─ Qualidade do código
│  ├─ Documentação
│  ├─ Revisão de PRs
│  ├─ Decisões arquiteturais
│  ├─ Escalabilidade
│  └─ Suporte a outros devs
│
├─ Duty: On-call para esse componente
│
└─ Evolução: Mentora outros na área

Benefício: Profundo conhecimento
Risco: Single point of failure (mitigar com compartilhamento)
```

---

### 3. Process Ownership

Pessoa é dona de um processo de qualidade.

```
Processo: CI/CD Pipeline
├─ Dono: DevOps/Platform Eng
├─ Responsável por:
│  ├─ Saúde do pipeline
│  ├─ Performance (feedback loops)
│  ├─ Confiabilidade (uptime)
│  ├─ Escalabilidade
│  └─ Documentação
│
├─ Usuários: Todos os devs
└─ Goal: Rápido, confiável, bem documentado

Exemplo:
├─ Test Pipeline dono: QA Lead
├─ Security dono: SecOps
├─ Performance dono: Performance Eng
└─ Reliability dono: SRE
```

---

### 4. Quality Ownership (Shared)

Qualidade é de **todos**, mas há clareza de papéis.

```
❌ ERRADO:
"Qualidade é responsabilidade de QA"
Developer: "Não é meu trabalho"

✅ CORRETO:
"Qualidade é de todos. Deixe-me descrever papéis:"

Developer:
├─ Escreve testes unitários (80%+ coverage)
├─ Responsável por qualidade do código
├─ Considera casos de borda
├─ Code review rigoroso
└─ Production support (on-call)

QA:
├─ Define estratégia de teste
├─ E2E testing
├─ Validação de cenários reais
├─ Performance e segurança
└─ Teste exploratório

Product:
├─ Define requisitos claros
├─ Aceita features
├─ Feedback de usuários
└─ Priorização de bugs

Resultado: Qualidade é realmente compartilhada
```

---

## Implementando Ownership

### Passo 1: Definir Responsabilidades

```
Inventário de Componentes/Features:
├─ Payment System
│  ├─ Dono: Dev A (Tech Lead)
│  ├─ Backup: Dev B
│  └─ RACI: A=Responsible, B=Consulted
│
├─ Notification Service
│  ├─ Dono: Dev C
│  └─ Backup: Dev D
│
├─ Auth System
│  ├─ Dono: Dev E
│  └─ Backup: Dev F
│
└─ Frontend Components
   ├─ Dono: Dev G (Frontend Lead)
   └─ Backup: Dev H (Frontend)

Documentar em: Wiki, Confluence, ou GitHub
```

---

### Passo 2: Clareza de Autoridade

```
DEV OWNER pode:
✅ Decidir como implementar
✅ Revisar PRs
✅ Priorizar bugs
✅ Refatorar conforme necessário
✅ Tomar decisões técnicas
✅ Rejeitar changes ruins

DEV OWNER NÃO pode:
❌ Ignorar processo de deploy
❌ Colocar código ruim em produção
❌ Ignorar code review
❌ Ignorar testes
❌ Decisões de negócio (PM faz isso)
❌ Contrata/demite (Manager)
```

---

### Passo 3: On-Call Rotation

```
On-Call: Pessoa responsável por alertas em produção

Estrutura:
├─ 1 pessoa on-call por sprint
├─ Disponível 24/7 para seu componente
├─ Próxima na fila se a primeira não responde
├─ Resposta SLA: < 5 minutos para CRITICAL
│
├─ Compensação:
│  ├─ Flexibilidade horária (não rígido 9-5)
│  ├─ Folga no dia seguinte se chamado à noite
│  ├─ Bônus/compensation (se houver)
│  └─ Load balanceado (não mesmo dev sempre)
│
└─ Benefício: Incentiva qualidade
   (Se código é seu, você quer que funcione)
```

### Passo 4: Autonomia com Guardrails

```
Autonomia: Dev pode fazer mudanças sem pedir permissão
Guardrails: Mas deve seguir standards

Exemplo:
Dev (Payment Lead) pode:
├─ ✅ Refatorar código interno
├─ ✅ Adicionar tipos/segurança
├─ ✅ Otimizar performance
├─ ✅ Adicionar testes

Dev (Payment Lead) não pode:
├─ ❌ Mudar API público sem avisar usuarios
├─ ❌ Quebrar contrato de outro serviço
├─ ❌ Ignorar testes
├─ ❌ Deploy sem code review

Processo:
├─ Self-service: Mudanças internas
└─ Colaborativo: Mudanças que afetam outros
```

---

## Ownership em Diferentes Escalas

### Nível 1: Indivíduo

```
Dev Junior é dono de:
├─ Seu PR / código que escreve
├─ Testes que escreve
├─ Código review que faz
└─ Aprendizado

Responsabilidade: PESSOAL
├─ Escrever bom código
├─ Pedir ajuda quando precisa
├─ Aprender com feedback
└─ Responsável mas com mentorship
```

---

### Nível 2: Feature/Squad

```
Squad de 5 pessoas é dona de:
├─ Feature X
├─ Qualidade dela
├─ Performance dela
├─ Manutenção dela
├─ Escalabilidade dela
└─ On-call para ela

Estrutura:
├─ Tech Lead (decision maker)
├─ Senior Dev (code quality)
├─ Mid/Junior Devs (hands-on)
├─ QA (strategy)
└─ Possível PO dedicado

Benefício: Full context, rápido ciclo
Risco: Silos entre squads
```

---

### Nível 3: Platform/Infra

```
Platform Team é dona de:
├─ CI/CD pipeline
├─ Infraestrutura
├─ Test infrastructure
├─ Deployment systems
├─ Monitoring
└─ Escalabilidade

Usuários: Todos os outros squads

Balanceamento:
- Muita autonomia aos squads
- Platform fornece ferramentas
- Squads usam ferramentas
- Ambos investem em qualidade
```

---

### Nível 4: Organização

```
Organização é dona de:
├─ Qualidade geral
├─ Security
├─ Performance
├─ Scalability
├─ Confiança de usuários
└─ Reputação

Como?
├─ Standards & guidelines
├─ Quality gates
├─ Security scanning
├─ Performance budgets
├─ SLOs (Service Level Objectives)

Exemplo SLO:
├─ Availability: 99.95%
├─ P95 Latency: < 200ms
├─ Error rate: < 0.1%
└─ Defect leakage: < 5%
```

---

## Mitigando Riscos de Ownership

### Problema: Single Point of Failure

```
Cenário: Dev A é único owner de componente crítico

Riscos:
├─ Se Dev A sai, conhecimento vai com ele
├─ Se Dev A fica doente, componente desatendido
├─ Se Dev A fica sobrecarregado, qualidade cai
└─ Turnover crítico

Solução:

1. Conhecimento Compartilhado
   ├─ Documentação clara
   ├─ Code reviews por outros
   ├─ Pair programming periodicamente
   └─ Knowledge sharing sessions

2. Backup Owner
   ├─ Dev B é backup
   ├─ Recebe 20% das PRs
   ├─ Supt on-call ocasionalmente
   └─ Conhecimento crescente

3. Gradual Transition
   ├─ Dev A treina Dev B
   ├─ Dev B aumenta responsabilidade
   ├─ Dev A se torna advisor
   └─ Smooth transition

Timeline: 2-3 meses
```

### Problema: Developer Burnout

```
Cenário: Owner fico sobrecarregado

Prevenção:

1. Load Balancing
   ├─ Não colocar um dev em múltiplos criticals
   ├─ Rodízio de on-call
   ├─ Backlog adequado
   └─ Não é eternidade

2. Support System
   ├─ Mentorship (não fazer sozinho)
   ├─ Peer review (validação)
   ├─ Escalation path (quando impossível)
   └─ Feedback regular

3. Recognition
   ├─ Reconhecer o esforço
   ├─ Compensação (se on-call)
   ├─ Growth opportunity
   └─ Career development
```

---

## Ownership e Qualidade

### Como Ownership Melhora Qualidade

```
Correlação Direta:

Owner que se importa:
├─ ✅ Escreve testes rigorosos
├─ ✅ Code review cuida
├─ ✅ Otimiza performance
├─ ✅ Documenta
├─ ✅ Planeja arquitetura bem
├─ ✅ Pensa em scaling
└─ ✅ Quer que código última os anos

Result: Código de alta qualidade

Metrics:
├─ Bugs own component: 40% menos
├─ MTTR own component: 70% mais rápido
├─ Refatoração: 3x mais
└─ Satisfação pessoal: Muito mais alta
```

---

### Ownership e DoD/DOR

```
Definition of Done deve incluir:

"Código está pronto para produção se:"
├─ ✅ Owner revisou (não apenas dev genérico)
├─ ✅ Testes cobrem cenários
├─ ✅ Performance aceitável (ou melhor)
├─ ✅ Documentação atualizada
├─ ✅ Produção está pronta para receber
└─ ✅ Owner se importa com qualidade

Definition of Ready:

"Feature está pronta se:"
├─ ✅ Owner entendeu requisito
├─ ✅ Estratégia de teste definida com Owner
├─ ✅ Owner identificou riscos
├─ ✅ Owner tem capacity
└─ ✅ Critérios de aceite claros pra Owner
```

---

## Referências Relacionadas

```
Ownership conecta com:

02 - Unit Testing:
└─ Owner é responsável por cobertura

04 - E2E Testing:
├─ Owner define strategy
└─ Owner responsável pelo test suite

05 - Test Strategy & Quality Gates:
├─ Owner define DoD
├─ Owner respeita quality gates
└─ Owner enforcement

06 - CI/CD & Automation:
├─ Owner usa pipeline
├─ Owner respeita gates
└─ Owner on-call

07 - Metrics & Maturity:
├─ DORA metrics: Owner responsável
├─ Quality metrics: Owner accountable
└─ Maturity: Owner evolui

08 - Dev-QA Collaboration:
└─ Owner trabalha com QA como parceiros
```

---

## Boas Práticas

### 1. Ownership Não é Isolação

```
❌ ERRADO:
Owner: "Esse é meu código, não mexe"

✅ CORRETO:
Owner: "Esse é meu código. Vou cuidar da qualidade, mas fico feliz
         com PRs de outros. Preciso revisar antes de merge."
```

---

### 2. Ownership Recompensa Qualidade

```
Antes:
├─ Dev A escreve código rápido (sem testes)
├─ Dev B escreve com cuidado (com testes)
└─ Ambos ganham igual

Depois (com on-call):
├─ Dev A (código ruim) → on-call frequente → Burnout 😞
├─ Dev B (código bom) → on-call raro → Happy 😊
└─ Incentivo alinhado com qualidade
```

---

### 3. Ownership + Collaboration

```
Owner não significa trabalhar sozinho

Owner:
├─ ✅ Responsável
├─ ✅ Mas colabora com time
├─ ✅ Pede ajuda quando precisa
├─ ✅ Treina outros
├─ ✅ Compartilha conhecimento
└─ ✅ Não é single point of failure
```

---

## Conclusão

Ownership não é apenas atribuir responsabilidade.

É criar um **ambiente onde pessoas se importam** com qualidade porque:

- ✅ Sabem que é seu
- ✅ Têm autonomia para fazer certo
- ✅ Têm autoridade para decisões
- ✅ São recompensadas por qualidade (on-call raro)
- ✅ São reconhecidas (expertise, opportunity)
- ✅ Têm suporte (mentoring, collaboration)

Resultado: **Código melhor, Qualidade melhor, Time mais feliz.**
