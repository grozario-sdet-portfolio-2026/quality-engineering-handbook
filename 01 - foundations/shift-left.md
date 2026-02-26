# Shift-Left Testing

## Origem do Conceito

Shift-Left é uma estratégia de qualidade que propõe antecipar atividades de validação no ciclo de desenvolvimento de software.

Tradicionalmente, testes eram executados após a implementação completa do sistema. O Shift-Left altera essa dinâmica ao mover atividades de teste, validação e prevenção de defeitos para fases mais iniciais do desenvolvimento.

O objetivo central é reduzir o custo de correção e aumentar a velocidade de feedback.

---

## Conceito

O termo “Shift-Left” representa o deslocamento das atividades de qualidade para a esquerda na linha do tempo do desenvolvimento.

Em vez de validar apenas após a entrega, a qualidade passa a ser incorporada desde:

- Refinamento de requisitos
- Definição de critérios de aceite
- Modelagem de arquitetura
- Escrita do código

A estratégia transforma qualidade em responsabilidade compartilhada, não apenas uma fase do processo.

---

## Princípios Fundamentais

### 1. Prevenção é mais eficiente que detecção

Quanto mais cedo um defeito é identificado, menor seu custo de correção.

Erros detectados:
- Em requisitos → baixo custo
- Durante desenvolvimento → custo moderado
- Em produção → alto impacto financeiro e reputacional

---

### 2. Feedback Rápido

O tempo entre introdução e detecção de falhas deve ser mínimo.

Isso envolve:
- Testes automatizados desde o início
- Execução contínua em pipeline
- Validações locais antes de merge

---

### 3. Qualidade como parte da arquitetura

Shift-Left não é apenas adicionar testes antes.

Envolve:
- Design orientado a testabilidade
- Código desacoplado
- Observabilidade
- Monitoramento desde o início

---

## Práticas Associadas ao Shift-Left

### Refinamento com foco em qualidade

- Definição clara de critérios de aceite
- Identificação antecipada de cenários de borda
- Análise de risco por funcionalidade

---

## Test-Driven Development (TDD)

Test-Driven Development é uma abordagem onde os testes são escritos antes da implementação da funcionalidade.

Segue um ciclo curto e iterativo:

1. Escrever um teste que falha
2. Implementar o código mínimo para fazê-lo passar
3. Refatorar mantendo os testes verdes

Objetivos estratégicos:

- Garantir cobertura desde o início
- Forçar design mais simples e desacoplado
- Reduzir regressões
- Criar documentação executável da regra de negócio

TDD promove alta coesão e melhor testabilidade do código.

---

## Behavior-Driven Development (BDD)

Behavior-Driven Development é uma evolução do TDD com foco em comportamento observável do sistema.

Baseia-se na descrição de funcionalidades em linguagem próxima ao domínio do negócio, geralmente no formato:

Given (Dado)  
When (Quando)  
Then (Então)

Objetivos estratégicos:

- Alinhar negócio, desenvolvimento e qualidade
- Reduzir ambiguidades em requisitos
- Garantir rastreabilidade entre cenários e implementação
- Transformar critérios de aceite em testes automatizados

BDD reforça colaboração e clareza de requisitos dentro da estratégia Shift-Left.

---

## Automação Precoce

- Testes unitários escritos junto ao código
- Testes de contrato para integrações
- Mock de dependências externas
- Validações automatizadas no pipeline

---

## Integração com CI/CD

Uma estratégia Shift-Left eficaz exige integração contínua:

- Execução de testes unitários a cada commit
- Bloqueio de merge em caso de falha
- Análise estática automatizada
- Validação de segurança (SAST) no pipeline

O objetivo é evitar que defeitos avancem para estágios posteriores.

---

## Métricas Relevantes

Indicadores para avaliar maturidade da estratégia:

- Defect Leakage (falhas escapando para produção)
- Lead Time para correção de defeitos
- Tempo médio de feedback no pipeline
- Percentual de cobertura relevante
- Número de falhas detectadas pós-release

---

## Benefícios Estratégicos

- Redução de retrabalho
- Menor custo de correção
- Maior previsibilidade de entrega
- Aumento da confiança em deploys
- Redução de incidentes em produção

---

## Anti-Patterns Comuns

### Testes apenas no final do ciclo

Manter validação apenas na fase final gera:

- Acúmulo de defeitos
- Correções emergenciais
- Atrasos em release

---

### Automação tardia

Automatizar somente após estabilização da feature aumenta custo e reduz efetividade.

---

### Confundir Shift-Left com “testar mais cedo”

Shift-Left não é apenas antecipar testes.

É incorporar qualidade na definição, arquitetura e desenvolvimento.

---

## Conclusão

Shift-Left é uma estratégia orientada à prevenção, velocidade de feedback e redução de risco.

Ao integrar qualidade desde a concepção até a entrega contínua, o processo se torna mais eficiente, sustentável e alinhado aos objetivos de negócio.