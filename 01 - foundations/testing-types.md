# Testing Types and Technologies

## Visão Geral

Diferentes tipos de teste atendem a objetivos distintos dentro de uma estratégia de qualidade.  
A escolha adequada depende de fatores como risco, arquitetura, criticidade de negócio e estágio do ciclo de desenvolvimento.

Abaixo estão os principais tipos de teste utilizados em ambientes modernos, seus objetivos e tecnologias comumente adotadas.

---

## Testes Unitários

Validam unidades isoladas de código, como funções e métodos, sem dependências externas reais.

**Objetivos:**
- Garantir regras de negócio
- Permitir refatorações seguras
- Oferecer feedback rápido

**Tecnologias comuns:**
- Jest
- Mocha + Chai

Essas ferramentas permitem execução rápida, mocks e assertions robustas.

---

## Testes de Integração

Validam a comunicação entre módulos, serviços ou camadas da aplicação.

**Objetivos:**
- Garantir contratos internos
- Validar persistência e integrações reais
- Detectar falhas de interface

**Tecnologias comuns:**
- Jest
- Mocha + Chai

Podem utilizar banco de dados em memória, containers ou ambientes controlados para simular integrações reais.

---

## Testes de Componentes

Muito utilizados em aplicações front-end modernas.

Validam componentes isolados com dependências controladas.

**Objetivos:**
- Garantir comportamento correto da interface
- Validar renderização condicional
- Reduzir dependência excessiva de E2E

**Tecnologias comuns:**
- Jest
- Vue Testing Utils
- React Testing Library

Essas bibliotecas permitem testar comportamento sem acoplamento à implementação interna.

---

## Testes de Carga (Load Testing)

Avaliam o comportamento do sistema sob diferentes níveis de carga.

**Objetivos:**
- Identificar gargalos
- Avaliar escalabilidade
- Medir tempo de resposta
- Validar limites operacionais

**Tecnologias comuns:**
- k6

Ferramentas de load testing permitem simular múltiplos usuários simultâneos e coletar métricas de performance.

---

## Testes de Mutação

Avaliam a efetividade da suíte de testes introduzindo alterações intencionais no código.

**Objetivos:**
- Medir qualidade dos testes
- Identificar cenários não cobertos
- Reduzir falsa sensação de cobertura

**Tecnologias comuns:**
- Stryker

Testes de mutação ajudam a garantir que os testes realmente validam comportamentos críticos.

---

## Testes Manuais

Complementam a automação e são fundamentais em determinados contextos.

Incluem:

- Testes exploratórios
- Testes regressivos manuais
- Testes de experiência
- Validação visual (UI/UX)

**Objetivos:**
- Identificar comportamentos inesperados
- Avaliar usabilidade
- Validar percepção do usuário

São especialmente relevantes para cenários complexos ou subjetivos.

---

## Testes de Segurança

Avaliam vulnerabilidades e falhas de proteção do sistema.

Incluem validações contra:

- Cross-Site Scripting (XSS)
- SQL Injection
- Falhas de autenticação
- Exposição indevida de dados

**Objetivos:**
- Reduzir riscos de exploração
- Proteger dados sensíveis
- Garantir conformidade com padrões de segurança

Podem ser realizados manualmente ou com ferramentas automatizadas integradas ao pipeline.

---

## Testes de Contrato

Validam acordos entre serviços, especialmente em arquiteturas distribuídas e microsserviços.

**Objetivos:**
- Garantir compatibilidade entre produtor e consumidor
- Evitar que mudanças quebrem integrações
- Reduzir dependência de ambientes compartilhados

**Tecnologias comuns:**
- Jest
- Mocha + Chai
- Pact

Testes de contrato permitem validar expectativas sem necessidade de integração completa em tempo real.

---

## Considerações Estratégicas

A adoção de diferentes tipos de teste deve considerar:

- Criticidade da funcionalidade
- Complexidade da arquitetura
- Risco operacional
- Custo de falha
- Tempo de feedback esperado

Uma estratégia madura combina múltiplos tipos de teste de forma complementar, evitando dependência excessiva de uma única abordagem.

---

## Conclusão

A diversidade de tipos de teste reflete a complexidade dos sistemas modernos.

A combinação adequada entre testes unitários, integração, componentes, carga, mutação, segurança e contrato permite equilibrar velocidade, confiabilidade e redução de risco.

A escolha das tecnologias deve priorizar integração com o ecossistema da aplicação, facilidade de manutenção e eficiência operacional.