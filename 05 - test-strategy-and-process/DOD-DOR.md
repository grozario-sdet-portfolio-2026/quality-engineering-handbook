# Definition of Done (DOD) e Definition of Ready (DOR)

## Introdução

Definition of Done (DOD) e Definition of Ready (DOR) são duas práticas fundamentais de um time ágil e orientado a qualidade.

Enquanto **DOR define quando uma atividade está pronta para começar**, **DOD define quando uma atividade está pronta para ir para produção**.

Essas definições são o **contrato entre o time de engenharia e o negócio**, garantindo que:

- ✅ Requisitos são claros antes de começar (DOR)
- ✅ Atividades são completas antes de finalizar (DOD)
- ✅ Qualidade é objetivo, não negociável
- ✅ Surpresas em produção são minimizadas

---

## Por Que DOD e DOR?

### Problemas que Resolvem

#### Sem DOR:

- 🔴 Dev começa sem clareza de requisitos
- 🔴 Meio do desenvolvimento mudam requisitos
- 🔴 Retrabalho constante
- 🔴 Estimativas sempre erradas

#### Sem DOD:

- 🔴 Atividades são aproximadas para "pronto"
- 🔴 Testes incompletos
- 🔴 Bugs chegam em produção
- 🔴 Não há critério objetivo para finalizar

#### Com DOR e DOD:

- 🟢 Atividades começam claras e estruturadas
- 🟢 Fim é objetivo e bem definido
- 🟢 Qualidade é não-negociável
- 🟢 Time trabalha com confiança

### Impacto nas Métricas

```
Métrica                    Sem DOD/DOR    Com DOD/DOR    Melhoria
─────────────────────────────────────────────────────────────
Lead Time                  8 dias         3 dias         62% ↓
Taxa de Retrabalho         35%            8%             77% ↓
Bugs em Produção/mês       12             2              83% ↓
Satisfação do Time         6/10           9/10           50% ↑
Produtividade              70%            95%            36% ↑
```

---

## Definition of Ready (DOR)

### Propósito

**DOR** estabelece o conjunto de critérios que uma atividade deve atender **antes de entrar em desenvolvimento**.

Garante que o time começa com:
- Requisitos claros
- Critérios de aceite bem definidos
- Estratégia de teste alinhada
- Estimativas realistas

### Princípios do DOR

#### 1. Clareza de Requisitos

O que exatamente precisa ser feito? Não deve haver ambiguidade.

**Exemplo ruim:**
```
Título: "Melhorar busca de usuários"
Descrição: "A busca não funciona bem"
```

**Exemplo bom:**
```
Título: "Implementar busca de usuários com filtros avançados"
Descrição: "
Permitir que usuários busquem outros usuários por:
- Nome (partial match, case-insensitive)
- Email (exact match)
- Departamento (múltipla seleção)
- Data de criação (range)

A busca deve retornar máximo 50 resultados.
Resultados devem estar paginados.
"
```

#### 2. Critérios de Aceite Objetivos

"Pronto" é mensurável, não subjetivo.

**Exemplo ruim:**
```
- [ ] Implementar busca
- [ ] Deixar bonito
- [ ] Testar bem
```

**Exemplo bom:**
```
- [ ] Form de busca renderiza com campos: Nome, Email, Departamento, Data
- [ ] Busca por nome retorna usuários com match parcial (case-insensitive)
- [ ] Busca por email retorna match exato quando preenchido
- [ ] Filtro de departamento permite múltipla seleção
- [ ] Filtro de data funciona com range (de-até)
- [ ] Resultados são paginados em grupos de 10
- [ ] Máximo 50 resultados retornados
- [ ] Tempo de resposta < 1s em caso normal
- [ ] Mensagem clara quando nenhum resultado encontrado
```

#### 3. Estimativa de Esforço Considerando Testes

Testes **não são bônus**, são parte do esforço. A estimativa deve incluir:

- Implementação da funcionalidade (X pontos)
- Testes unitários (Y pontos)
- Testes de integração (Z pontos)
- Code review (A pontos)

**Exemplo:**
```
Story: Implementar busca com filtros
Estimativa breakdown:
- Implementação: 5 pontos
- Testes unitários: 2 pontos
- Testes integração: 2 pontos
- Code review: 1 ponto
TOTAL: 10 pontos (não 5 + testes depois)
```

#### 4. Pontos de DOD e DOR Descritos no Card

O card deve deixar claro:
- Quando será considerado "pronto" (DOD)
- O que faltava para "começar" (DOR)

#### 5. Aceitação do Time

O time (não só PO/PM) precisa estar alinhado:
- Devs entendem o requisito?
- QAs identificaram cenários de teste?
- Design aprovou?

### Checklist de DOR

Use esse checklist no refinamento:

**Requisitos:**
- [ ] Título claro e descritivo
- [ ] Descrição detalha o "o que" e "por que"
- [ ] User story está bem formada (se aplicável)
- [ ] Critérios de aceite são SMART (Específicos, Mensuráveis, Atingíveis, Relevantes, Temporal)
- [ ] Wireframes/protótipos anexados (se necessário)

**Testes e Qualidade:**
- [ ] Estratégia de teste definida (tipos: unitário, integração, e2e)
- [ ] Cenários positivos identificados
- [ ] Cenários negativos/edge cases identificados
- [ ] Ambientes testados identificados (local, dev, tst)

**Estimativa:**
- [ ] Estimativa inclui código, testes e review
- [ ] Story size é adequado (não muito grande)
- [ ] Dependências externas identificadas

**Alinhamento:**
- [ ] PO/PM alinhado
- [ ] Team lead/tech lead alinhado
- [ ] QA/Teste alinhado
- [ ] Design alinhado (se aplicável)
- [ ] DevOps alinhado (se mudança infraestrutura)

**Pronto:**
- [ ] Nenhuma questão em aberto
- [ ] Time unânime ("Podemos começar?")
- [ ] Card movido para "Ready" no quadro

---

## Definition of Done (DOD)

### Propósito

**DOD** estabelece o conjunto de critérios que uma atividade deve atender **antes de ser considerada completa** e pronta para produção.

Garante que:
- Funcionalidade está implementada conforme requisitos
- Testes são abrangentes
- Código passa por verificações de qualidade
- Documentação está atualizada
- Nada foi esquecido

### Princípios do DOD

#### 1. Funcionalidade Completa

Todos os critérios de aceite foram atendidos.

- ✅ Funcionalidade implementada
- ✅ Todos os critérios de aceite passam
- ✅ Edge cases tratados
- ✅ Comportamento em erro está definido

#### 2. Testes Abrangentes

Código está coberto por testes em múltiplos níveis.

**Testes Unitários:**
- [ ] Cobertura mínima 80% da lógica nova
- [ ] Testes cobrem caminho feliz
- [ ] Testes cobrem caminhos de erro
- [ ] Testes passam localmente

**Testes de Integração:**
- [ ] Se há integração externa, há teste de integração
- [ ] Se há acesso a BD, há teste com BD de teste
- [ ] Contratos com APIs externas validados

**Testes E2E (se aplicável):**
- [ ] Fluxo crítico testado end-to-end
- [ ] Testes não são flaky (não falham aleatoriamente)

#### 3. Code Review Aprovado

- ✅ 2 ou mais membros revisaram
- ✅ Todos os comentários foram respondidos
- ✅ Padrões do projeto foram seguidos
- ✅ Nenhuma flag vermelha técnica

#### 4. Análise de Código Passou

- ✅ Linter sem erros
- ✅ SonarQube com qualidade gate aprovada
- ✅ Sem vulnerabilidades críticas/altas
- ✅ Sem código duplicado excessivo

#### 5. Documentação Atualizada

- ✅ README atualizado se necessário
- ✅ Código complexo comentado
- ✅ API documentada (se expõe API)
- ✅ Changelog atualizado
- ✅ Design patterns/decisions documentadas

#### 6. Ambiente DEV Validado

- ✅ Stack completa executada com sucesso
- ✅ Funcionalidade validada em DEV por pessoa externa
- ✅ Design validado em DEV
- ✅ Sem não-conformidades críticas

#### 7. Ambiente TST Validado

- ✅ Stack completa executada com sucesso
- ✅ Funcionalidade validada em TST por persona diferente
- [ ] Se grande feature: teste exploratório com todo o time
- ✅ Sem não-conformidades críticas

#### 8. Pronto para Produção

- ✅ Sem hot-fixes pendentes
- ✅ Documentação de deploy/release notes preparada
- ✅ Observabilidade/monitoring configurado
- ✅ Rollback plan definido (se necessário)

### Checklist Completo de DOD

**Implantação:**
- [ ] Código implementado conforme especificação
- [ ] Feature branch mergeado em main/develop
- [ ] Commit messages descritivas e atômicas

**Testes:**
- [ ] Testes unitários criados (cobertura ≥80%)
- [ ] Testes de integração (se necessário)
- [ ] Testes E2E (se critério aceite)
- [ ] Todos os testes passando
- [ ] Testes não são flaky

**Qualidade:**
- [ ] Code review aprovado por 2+ pessoas
- [ ] Linter passando sem erros
- [ ] SonarQube com qualidade gate OK
- [ ] Sem vulnerabilidades críticas/altas
- [ ] Sem duplicação excessiva de código

**Documentação:**
- [ ] README atualizado
- [ ] JSDoc/TypeDoc adicionado (se necessário)
- [ ] Decisões técnicas documentadas (se comple complexity)
- [ ] Changelog atualizado

**Validação Funcional:**
- [ ] Funcionalidade testada em DEV
- [ ] Design aprovou em DEV
- [ ] Cross validation em DEV aprovada
- [ ] Funcionalidade testada em TST
- [ ] Cross validation ou teste exploratório em TST aprovado

**Preparação Produção:**
- [ ] Observabilidade/monitoring configurado
- [ ] Alerts criados (se necessário)
- [ ] Plano de rollback definido
- [ ] Release notes preparadas
- [ ] Documentação de deployment preparada

**Final:**
- [ ] Nenhuma flag vermelha aberta
- [ ] Time acordou que está "Done"
- [ ] Pronto para merge/deploy

---

## Diferenças Entre DOR e DOD

| Aspecto | DOR | DOD |
|---|---|---|
| **Quando** | Antes de começar | Depois de terminar |
| **Foco** | Clareza | Completude |
| **Responsável** | PO/PM + Team | Desenvolvedor + Time |
| **Objetivo** | Evitar surpresas no meio | Evitar surpresas em produção |
| **Se violado** | Retrabalho constante | Bugs em produção |

**Analogia:**
- **DOR** = Receita clara antes de começar a cozinhar
- **DOD** = Pratos prontos, testados e apresentáveis

---

## Implementando DOD e DOR no Time

### Fase 1: Definição (Semana 1)

1. **Time se reúne** para debater:
   - Qual contextualização é essencial? (DOR)
   - Qual nível de completude é esperado? (DOD)

2. **Cria rascunho** de DOR e DOD

3. **Refina em refinamento** com PO/PM

### Fase 2: Teste (Sprints 2-4)

1. **Aplica DOR** em refinamento
   - Marca cards "Ready" quando DOR é atingido
   - Cards sem DOR retornam para "Backlog"

2. **Aplica DOD** em acceptance
   - Cards só saem do sprint se DOD é 100%
   - Rastreia quantos cards violam DOD

### Fase 3: Refinamento (Sprints 5+)

1. **Retrospectiva** mensalmente sobre DOR/DOD
   - Está funcionando?
   - Necessita ajustes?
   - Novos critérios emergiram?

2. **Evolui** conforme team matura

### Fase 4: Integração em Processo

1. **Automatiza validações** (linter, coverage, security scans)
2. **Dashboard de métricas** (% de DOD compliance)
3. **Cultura permanente** ("É DOD?")

---

## Sinais de que DOD/DOR Está Funcionando

✅ Cards começam realmente prontos  
✅ Menos retrabalho durante sprint  
✅ Fewer bugs em produção  
✅ Lead time reduz  
✅ Previsibilidade melhora  
✅ Time tem confiança no que entrega  

---

## Sinais de que DOD/DOR Está Falhando

❌ Cards entram "Backlog" frequentemente (DOR ruim)  
❌ Sprint é caótica com mudanças de requisitos  
❌ Cards saem do sprint incompletos  
❌ Bugs aparecem em produção (DOD fraco)  
❌ Team "gamifica" checklist sem valor real  
❌ DOD é visto como burocracia  

---

## Exemplos Reais de DOD por Tipo de Atividade

### Feature de Negócio Simples

**Exemplo:** Adicionar campo de observações em formulário

**DOD:**
- [ ] Campo renderiza no formulário
- [ ] Dados salvos em BD
- [ ] Validação de tamanho máximo (500 chars)
- [ ] Teste unitário do validador
- [ ] Teste de integração do save
- [ ] Code review aprovado
- [ ] Funciona em DEV e TST
- [ ] Documentação atualizada

### Feature Complexa com Integração

**Exemplo:** Integração com gateway de pagamento

**DOD:**
- [ ] Endpoints de pagamento implementados
- [ ] Autenticação com gateway configurada
- [ ] Testes unitários (cobertura 90%+)
- [ ] Testes de integração com gateway mock
- [ ] Testes E2E do fluxo completo
- [ ] Tratamento de erros de timeout/falha
- [ ] Retry logic implementado
- [ ] Observabilidade/logging configurado
- [ ] Code review 2+ pessoas
- [ ] SonarQube OK
- [ ] Sem vulnerabilidades
- [ ] Validado em DEV com cross-validation
- [ ] Validado em TST com teste exploratório
- [ ] Documentação de deployment
- [ ] Runbook de troubleshooting criado

### Refatoração

**Exemplo:** Refatorar componente React

**DOD:**
- [ ] Comportamento externo não mudou
- [ ] Testes unitários criados (se não havia)
- [ ] Cobertura aumentou ou se manteve
- [ ] Performance igual ou melhor (benchmark)
- [ ] Testes passando
- [ ] Code review aprovado
- [ ] Funcionando igual em DEV/TST
- [ ] Decision document se complexity alta

---

## Possíveis Resistências e Como Lidar

### "DOD é muito rigoroso, vai atrasar"

**Resposta:**
Sim, mas economiza muito mais tempo depois:
- Antes: 5 dias dev + 3 dias bug fix = 8 dias
- Depois: 8 dias dev (com DOD) + 0 dias bug fix = 8 dias
- Plus: Sem bugs em produção

### "Nem sempre temos time para cross validation"

**Resposta:**
- Então rotaciona/distribui
- Ou faz pair programming (protege design)
- Ou reduz scope se recursos limitados

### "Isso é para times grandes, não para startups"

**Resposta:**
- Mesmo para times pequenos (ou **especialmente**)
- Pode ser adaptado (menos rigoroso)
- ROI é ainda maior em times pequenos

### "Temos pouco tempo, isso é overhead"

**Resposta:**
- Overhead de 10% hoje = economia de 50% amanhã
- Bugs em produção = reputação perdida
- Trade o que achar menos importante, mas não abra mão da qualidade

---

## Conclusão

**DOD e DOR são investimentos na qualidade e previsibilidade:**

- **DOR** garante que trabalho começa informado
- **DOD** garante que trabalho termina completo

Sem elas, o time trabalha com incerteza e produz defeitos.

Com elas, o time trabalha com clareza e confiança.

A escolha é simples: qualidade é mais cara no início ou no fim?
