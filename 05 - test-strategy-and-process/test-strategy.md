# Estratégia de Testes

## Visão Geral

Uma estratégia de testes eficaz é aquela que integra qualidade em todas as fases do desenvolvimento, desde o planejamento até a produção. Não se trata apenas de executar testes em um momento específico, mas sim de tornar a qualidade uma responsabilidade compartilhada por todos os membros do time.

A estratégia de testes de uma organização define:

- **O que** será testado e em que escala
- **Quando** os testes serão executados
- **Quem** é responsável pela validação
- **Como** manter a qualidade ao longo do tempo

Uma boa estratégia aumenta a confiança nas entregas, reduz o custo de correção de defeitos e acelera o feedback.

---

## Princípios Norteadores

### Pirâmide de Testes

A Pirâmide de Testes é um dos pilares fundamentais da nossa estratégia. Confira a documentação completa em [Pirâmide de Testes](../01%20-%20foundations/test-pyramid.md).

Em resumo, a pirâmide propõe:

- **Base (Muitos):** Testes unitários rápidos e baratos
- **Meio (Moderado):** Testes de integração
- **Topo (Poucos):** Testes E2E, mais complexos e lentos

Essa distribuição garante feedback rápido acompanhado de validações abrangentes.

### Shift-Left

Shift-Left é uma estratégia que antecipa atividades de validação para fases iniciais do desenvolvimento, reduzindo o custo de correção de defeitos. Confira a documentação completa em [Shift-Left Testing](../01%20-%20foundations/shift-left.md).

Princípios chave do Shift-Left:

- **Prevenção é mais eficiente que detecção** – Defeitos identificados cedo custam menos
- **Feedback rápido** – Testes automatizados desde o início
- **Qualidade na arquitetura** – Design testável desde a concepção

Combinando Pirâmide de Testes com Shift-Left, criamos uma estratégia robusta onde:

- Qualidade é contemplada desde o refinamento
- Testes são executados continuamente
- Defeitos são identificados o mais cedo possível

**Nota importante:** Embora a Pirâmide e Shift-Left sejam nossos nortes, a estratégia é flexível. Dependendo do contexto, outros tipos de organização de testes podem ser aplicados, desde que o objetivo final de qualidade seja mantido.

---

## Processo Ideal de Testes

Um processo ideal de testes é composto por múltiplas etapas, onde qualidade é responsabilidade compartilhada. Abaixo apresentamos cada fase do ciclo:

### 1. Planning (Planejamento)

**O que acontece:**
- Testes são discutidos **desde o início** da criação da atividade
- O time identifica cenários críticos, edge cases e riscos
- Qualidade é considerada na estimativa de esforço

**Responsabilidades:**
- **PO/PM:** Define requisitos claros e critérios de aceite
- **Devs/QAs:** Propõem estratégias de teste
- **Time:** Alinha expectativas sobre qualidade

**Saída esperada:**
- Atividade com requisitos, critérios de aceite e estratégia de testes claros
- Estimativa que inclui esforço de testes

---

### 2. Refinamento

**O que acontece:**
- O time analisa a atividade em detalhe
- Autonomia para propor **tipos de testes** mais adequados
- Validação de pontos de **DOD (Definition of Done)** e **DOR (Definition of Ready)**

**Responsabilidades:**
- **Time:** Propõe cenários de teste, tipos (unitário, integração, etc.)
- **QA/DevOps:** Valida estratégia e viabilidade
- **Todos:** Confirmam se DOD e DOR estão descritos no card

**Saída esperada:**
- Card com detalhes de teste bem definidos
- DOD e DOR descritos e consensuados
- Time alinhado com a estratégia

---

### 3. Coding (Desenvolvimento)

**O que acontece:**
- Desenvolvedores criam os testes necessários conforme a estratégia definida
- Testes são commited junto com o código
- Cobertura alinhada com critérios

**Responsabilidades:**
- **Devs:** Implementam funcionalidade e testes
- **Todos:** Segue a estratégia de teste definida no refinamento

**Saída esperada:**
- Código com testes implementados
- Testes passando localmente

---

### 4. Code Review

**O que acontece:**
- Atividade passa por revisão de **2 ou mais membros do time**
- Validação de qualidade do código e testes
- Feedback técnico e estratégico

**Responsabilidades:**
- **Reviewers:** Validam lógica, testes, padrões
- **Autor:** Responde feedbacks e itera

**Saída esperada:**
- Código aprovado por múltiplos pares
- Testes revisados e validados

---

### 5. Pull Request (PR) - PR Checker

**O que acontece:**
- **PR Checker** valida se funcionalidades estão **devidamente testadas**
- Verifica se nenhuma outra funcionalidade foi quebrada
- Executa **análise de código:** linters, SonarQube, ferramentas de qualidade
- Testes automatizados são executados

**Responsabilidades:**
- **PR Checker:** Validação técnica completa
- **Pipeline:** Execução de ferramentas de qualidade

**Critérios de aprovação:**
- ✅ Todos os testes passando
- ✅ Cobertura adequada (conforme padrão do time)
- ✅ Sem erros de lint/SonarQube
- ✅ Documentação atualizada se necessário

**Saída esperada:**
- PR aprovado e pronto para merge
- Confiança de que não há regressões

---

### 6. Ambientes Internos: DEV

**O que acontece:**
- Antes da atividade chegar em DEV, a **stack inteira é executada** (mesmo processo do PR checker)
- **Cross Validation:** 1 ou mais pessoas que **não participaram da atividade** validam manualmente
- **Design Validation:** Equipe de design valida se layout e experiência estão conforme requisitos

**Responsabilidades:**
- **QA/Dev externo:** Cross validation funcional
- **Design/PM:** Design validation
- **Developer:** Correção de problemas encontrados

**Possíveis resultados:**
- ✅ Nenhum problema → continua para TST
- ❌ Problemas encontrados → criar card de bug (Jira ou ferramenta similar) e continuar em dev

**Saída esperada:**
- Atividade validada funcionalmente
- Layout e experiência alinhados com requisitos

---

### 7. Ambientes Internos: TST

**O que acontece:**
- Antes de chegar em TST, a **stack de testes é executada novamente**
- **Cross Validation em TST:** Membros do time que **não criaram nem validaram em DEV** realizam validação
- Para atividades grandes, Cross Validation é **substituído por Teste de Regressão/Exploratório/Experiência**
- **Todos os membros do time participam:** devs, QAs, PM, TM, design

**Responsabilidades:**
- **QA/Dev externo:** Validação cruzada
- **Todos (em grandes atividades):** Teste exploratório compartilhado
- **Developer:** Correção de problemas

**Possíveis resultados:**
- ✅ Nenhum problema → pronta para produção
- ❌ Problemas encontrados → criar card de bug e retornar para dev/correção

**Saída esperada:**
- Atividade completamente validada
- Time confiante na qualidade

---

### 8. Produção e Observabilidade

**O que acontece:**
- Atividade é liberada para produção
- **Observabilidade ativa:** Membros designados monitoram a atividade

**Responsabilidades:**
- **Observability Team:** Monitora em tempo real:
  - 📊 Métricas de downtime
  - 📋 Logs de erro
  - 🔍 Dados no banco
  - 📈 Performance
  - ❌ Erros críticos

**Ações reativas:**
- Se problemas detectados → rollback ou hotfix conforme impacto
- Criar card de bug para análise pós-mortem se necessário

**Saída esperada:**
- Atividade em produção funcionando conforme esperado
- Time reativo a problemas

---

## Fluxograma do Processo Completo

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       ESTRATÉGIA DE TESTES - FLUXO COMPLETO                 │
└─────────────────────────────────────────────────────────────────────────────┘

                                 PLANNING
                         (Testes desde o início)
                                   │
                                   ▼
                              REFINEMENT
                      (Validar DOD/DOR, tipos de testes)
                                   │
                                   ▼
                                CODING
                         (Dev cria funcionalidade + testes)
                                   │
                                   ▼
                              CODE REVIEW
                       (2+ membros do time validam)
                                   │
                                   ▼
                            ┌──────────────┐
                            │  PR CHECKER  │
                            │ (Pipeline)   │
                            └──────────────┘
                                   │
                    ┌──────────────┼──────────────┐
                    │              │              │
                 TESTS         COVERAGE       LINT/SONARQUBE
                PASSANDO       ADEQUADA        SEM ERROS
                    │              │              │
                    └──────────────┴──────────────┘
                                   │
                         ✅ Pronto para DEV?
                                   │
                    ┌──────────────┴──────────────┐
                    │                             │
                   NÃO                           SIM
                    │                             │
                    ▼                             ▼
              ┌──────────────┐          ┌─────────────────────┐
              │  Corrigir    │          │  AMBIENTES INTERNOS │
              │   Problemas  │          │       (DEV)          │
              └──────────┬───┘          └────────────┬─────────┘
                         │                          │
                         └──────────────┬───────────┘
                                        │
                        Stack de testes executada novamente
                                        │
                        Cross Validation + Design Validation
                        (Pessoas que não participaram)
                                        │
                             ✅ Tudo OK em DEV?
                                        │
                    ┌───────────────────┴──────────────────┐
                    │                                      │
                   NÃO                                     SIM
                    │                                      │
                    ▼                                      ▼
              ┌──────────────┐                    ┌─────────────────────┐
              │  Criar Card  │                    │  AMBIENTES INTERNOS │
              │  de Bug      │                    │       (TST)          │
              │ + Retornar   │                    └────────────┬─────────┘
              │  para Dev    │                                 │
              └──────────┬───┘                 Stack de testes executada novamente
                         │                                 │
                         │                    ┌────────────┴────────────┐
                         │                    │                         │
                         │           Cross Validation        Teste Exploratório/
                         │      (Pessoas não envolvidas)    Regressão (Grande escala)
                         │                    │             (Todo time participa)
                         │                    │                         │
                         │                    └────────────┬────────────┘
                         │                                 │
                         │                      ✅ Tudo OK em TST?
                         │                                 │
                         │                    ┌────────────┴────────────┐
                         │                    │                         │
                         │                   NÃO                       SIM
                         │                    │                         │
                         │                    ▼                         ▼
                         │              ┌──────────────┐        ┌─────────────────┐
                         │              │  Criar Card  │        │   PRODUÇÃO      │
                         └─────────────▶│  de Bug      │        │ + OBSERVABILIDADE
                                        │ + Retornar   │        └────────┬────────┘
                                        │  para Dev    │                 │
                                        └──────────────┘      Monitorar em tempo real:
                                                             • Downtime
                                         Ou                  • Logs/Erros
                                                             • Dados BD
                                         Retornar            • Performance
                                         para TST se        • Erros críticos
                                         correção rápida          │
                                                             ✅ Sem problemas?
                                                                │
                                                    ┌───────────┴────────────┐
                                                    │                        │
                                                   SIM                      NÃO
                                                    │                        │
                                                    ▼                        ▼
                                              ┌──────────┐           ┌──────────────┐
                                              │ SUCESSO  │           │ Rollback ou  │
                                              │   ✅     │           │ Hotfix +     │
                                              │ Activity │           │ Investigação │
                                              │Complete  │           └──────────────┘
                                              └──────────┘
```

---

## Engajamento do Time em Qualidade

### O Papel de Cada Um

É extremamente importante para a estratégia de qualidade que **todo o time esteja engajado**, pois somente um QA não consegue ser responsável por todas as facetas de um processo tão amplo e complexo.

#### Desenvolvedores

- Escrevem testes unitários e de integração
- Responsáveis por cobertura adequada
- Identificam cenários de risco durante desenvolvimento
- Participam de code reviews
- Realizam validação cruzada em DEV

#### QAs

- Definem estratégia de teste
- Realizam testes exploratórios
- Validam critérios de aceite
- Executam testes de regressão
- Monitoram métricas de qualidade

#### Product Owner / Manager

- Define critérios de aceite claros
- Participa do refinamento de testes
- Valida features em DEV e TST
- Prioriza bugs críticos
- Acompanha métricas de qualidade

#### Tech Lead / Architect

- Define padrões de teste
- Garante testabilidade do design
- Valida qualidade arquitetural
- Realiza code reviews técnicos

#### Designer

- Valida design em DEV
- Verifica aderência aos wireframes/protótipos
- Valida experiência do usuário
- Participa de testes exploratórios em grandes atividades

#### Todo o Time (coletivamente)

- Compromisso com qualidade
- Participação em testes exploratórios para grandes features
- Feedback contínuo sobre processo
- Propriedade compartilhada da qualidade

---

## Qualidade em Tudo

A qualidade **não é uma fase**, é um princípio. Precisa estar presente em:

### Métricas do Time

- Taxa de defeitos encontrados por fase
- Cobertura de testes
- Tempo médio de correção de bugs
- Lead time (do planejamento à produção)
- Densidade de defeitos

### Reuniões

- **Planning:** Discussão de estratégia de teste
- **Refinement:** Validação de DOD/DOR com foco em qualidade
- **Retrospectiva:** Análise de falhas e melhorias de processo
- **Demo:** Validação de qualidade com stakeholders

### Daily Standup

- Mencionar bloqueadores de qualidade
- Alertar sobre defeitos críticos encontrados
- Comunicar progresso de testes

### Pull Requests

- Comentários específicos sobre qualidade
- Sugestões de melhorias de teste

### Production Monitoring

- Alerts de problemas em tempo real
- Análise de logs e erros
- Acompanhamento de métricas

---

## Conclusão

Uma estratégia de testes eficaz é resultado de:

1. **Processo bem definido** – Com responsabilidades claras em cada etapa
2. **Uso de melhores práticas** – Pirâmide de Testes e Shift-Left como guias
3. **Engajamento do time** – Qualidade como responsabilidade compartilhada
4. **Cultura de qualidade** – Integrada em tudo que o time faz
5. **Observabilidade e feedback** – Contínuo acompanhamento de métricas

Quando essas dimensões se alinham, qualidade deixa de ser um custo e passa a ser um facilitador de velocidade.
