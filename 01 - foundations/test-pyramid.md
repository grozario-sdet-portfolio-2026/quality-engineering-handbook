# Pirâmide de Testes

![Pirâmide de Testes](../assets/test-pyramid.jpg)

## Origem do Conceito

O conceito da Pirâmide de Testes foi introduzido por Mike Cohn em seu livro *Succeeding with Agile*. A proposta surgiu como uma estratégia para estruturar testes automatizados de forma sustentável, reduzindo dependência excessiva de testes manuais e suítes E2E lentas.

Posteriormente, o conceito foi amplamente difundido e refinado por Martin Fowler, consolidando-se como uma referência na arquitetura de testes.

---

## Conceito Clássico

A pirâmide tradicional é composta por três camadas principais:

1. Testes Unitários (base)
2. Testes de Integração
3. Testes de Ponta a Ponta (E2E)

Mais do que classificar tipos de teste, a pirâmide define uma **estratégia de distribuição**, baseada em:

- Custo de execução
- Velocidade de feedback
- Estabilidade
- Facilidade de manutenção

Quanto mais alto na pirâmide, maior o custo e menor a velocidade de execução.

---

## Proporção Recomendada

Uma estratégia sustentável costuma seguir a seguinte lógica:

- Muitos testes unitários (rápidos e baratos)
- Quantidade moderada de testes de integração
- Poucos testes E2E (mais lentos e mais frágeis)

Essa distribuição reduz o tempo total de execução da suíte e aumenta a confiabilidade do pipeline.

---

## Camadas da Pirâmide (Abordagem Moderna)

Em aplicações modernas, especialmente web e microsserviços, a pirâmide pode incluir camadas intermediárias como testes de componente.

### 1. Testes Unitários

Validam pequenas unidades de código de forma isolada.

**Objetivos:**
- Garantir regras de negócio
- Permitir refatorações seguras
- Oferecer feedback imediato

**Características:**
- Alta velocidade
- Baixo custo
- Alta confiabilidade

---

### 2. Testes de Componentes

Muito comuns em front-end moderno.

Validam componentes isolados com dependências controladas ou simuladas.

**Objetivos:**
- Garantir comportamento correto da interface
- Reduzir dependência excessiva de E2E

Atuam como camada intermediária entre unitários e integração.

---

### 3. Testes de Integração

Validam a comunicação entre módulos, serviços ou camadas da aplicação.

**Objetivos:**
- Garantir contratos entre sistemas
- Detectar falhas de integração precocemente
- Validar persistência e integrações externas

Possuem custo maior que unitários, mas são fundamentais para sistemas distribuídos.

---

### 4. Testes de Ponta a Ponta (E2E)

Simulam fluxos reais do usuário, validando o sistema como um todo.

**Objetivos:**
- Proteger fluxos críticos de negócio
- Validar integrações reais
- Garantir experiência funcional completa

Devem ser:
- Estratégicos
- Enxutos
- Focados em jornadas críticas

---

## Testes Manuais

Testes manuais não fazem parte da pirâmide original, mas continuam relevantes para:

- Testes exploratórios
- Validação de UX
- Análise visual
- Investigação de comportamentos inesperados

Devem complementar a automação, não substituí-la.

---

## Anti-Pattern: Ice Cream Cone

Um problema recorrente é o chamado “Ice Cream Cone”, descrito por Martin Fowler.

Caracteriza-se por:

- Poucos testes unitários
- Muitos testes E2E
- Forte dependência de testes manuais

Consequências:

- Pipeline lento
- Alta taxa de flaky tests
- Alto custo de manutenção
- Feedback tardio

Esse formato tende a se tornar insustentável conforme o sistema evolui.

---

## Integração com CI/CD

Uma estratégia eficaz considera a execução escalonada da suíte:

- Testes unitários a cada commit
- Testes de integração em pull requests
- Testes E2E em ambientes controlados
- Monitoramento contínuo do tempo total da pipeline

O objetivo é manter feedback rápido sem comprometer cobertura e confiabilidade.

---

## Métricas de Maturidade

Indicadores importantes para avaliar a saúde da estratégia:

- Tempo médio de execução da suíte
- Taxa de flaky tests
- Cobertura de código (com foco qualitativo)
- Tempo de feedback no pipeline
- Taxa de falhas em produção

---

## Conclusão

A Pirâmide de Testes é uma estratégia de engenharia voltada para sustentabilidade, velocidade de feedback e redução de risco.

Sua aplicação adequada permite escalar automação com estabilidade, manter pipelines eficientes e garantir maior confiabilidade nas entregas.