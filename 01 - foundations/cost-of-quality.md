# Cost of Quality (CoQ)

## Origem do Conceito

Cost of Quality (CoQ) é um modelo originado na gestão da qualidade industrial, amplamente difundido por práticas de engenharia e melhoria contínua.

O conceito estabelece que qualidade não é gratuita — ela possui custos associados tanto à prevenção quanto à falha.

No contexto de software, o modelo ajuda a compreender o impacto financeiro de defeitos, retrabalho e estratégias de validação.

---

## Conceito

Cost of Quality representa a soma dos custos envolvidos para garantir qualidade e dos custos gerados pela ausência dela.

O modelo é dividido em duas grandes categorias:

1. Custos de Conformidade
2. Custos de Não Conformidade

A relação entre essas categorias determina a maturidade do processo de engenharia.

---

## Custos de Conformidade

São investimentos realizados para evitar defeitos.

Dividem-se em:

### 1. Custos de Prevenção

Atividades realizadas para evitar que falhas ocorram:

- Refinamento detalhado de requisitos
- Arquitetura orientada à testabilidade
- Test-Driven Development
- Code review estruturado
- Automação de testes
- Treinamentos técnicos

Prevenção reduz significativamente o custo futuro de correção.

---

### 2. Custos de Avaliação

Atividades voltadas à detecção de defeitos antes da entrega:

- Execução de testes automatizados
- Testes exploratórios
- Auditorias técnicas
- Análises estáticas
- Validação em pipeline

Esses custos são previsíveis e controláveis.

---

## Custos de Não Conformidade

São custos gerados por falhas que não foram prevenidas ou detectadas.

Dividem-se em:

### 1. Falhas Internas

Defeitos identificados antes da entrega ao usuário:

- Retrabalho
- Correção de bugs
- Reprocessamento
- Reabertura de tarefas

Embora menos graves que falhas externas, impactam prazo e produtividade.

---

### 2. Falhas Externas

Defeitos identificados após o release:

- Incidentes em produção
- Hotfix emergencial
- Rollback de deploy
- Perda de receita
- Danos à reputação
- Multas contratuais

Falhas externas representam o maior custo dentro do modelo.

---

## Relação Custo vs Momento da Detecção

Quanto mais tarde um defeito é identificado, maior seu custo de correção.

Correção em:
- Requisito → baixo impacto
- Desenvolvimento → impacto moderado
- Produção → alto impacto técnico e financeiro

Esse princípio reforça estratégias como Shift-Left e automação contínua.

---

## Interpretação Estratégica

Investir em prevenção aumenta custos de conformidade no curto prazo, mas reduz drasticamente custos de não conformidade no médio e longo prazo.

Sistemas maduros apresentam:

- Maior investimento em prevenção
- Pipeline estável
- Baixa taxa de incidentes
- Previsibilidade operacional

Sistemas imaturos apresentam:

- Baixo investimento preventivo
- Alto volume de retrabalho
- Incidentes frequentes
- Operação reativa

---

## Integração com Engenharia de Software

Cost of Quality deve influenciar decisões como:

- Nível de automação
- Estratégia de testes
- Definição de quality gates
- Priorização de débito técnico
- Adoção de práticas como TDD

Qualidade deixa de ser apenas técnica e passa a ser decisão econômica.

---

## Métricas Relacionadas

Indicadores úteis para análise de CoQ:

- Defect Leakage
- MTTR (Mean Time to Recovery)
- Taxa de retrabalho
- Percentual de hotfix pós-release
- Custo médio por incidente
- Lead Time de correção

A mensuração desses indicadores permite avaliar o equilíbrio entre prevenção e falha.

---

## Anti-Patterns Comuns

### Reduzir investimento em prevenção para acelerar entregas

Pode gerar aumento exponencial de falhas externas.

---

### Focar apenas em cobertura de testes

Cobertura elevada não garante redução de falhas se não houver foco em risco e regra de negócio.

---

### Tratar incidentes como eventos isolados

Falhas recorrentes indicam problema estrutural no modelo de qualidade.

---

## Benefícios da Análise de Cost of Quality

- Melhor tomada de decisão técnica
- Redução de incidentes em produção
- Maior previsibilidade financeira
- Justificativa estratégica para investimento em qualidade
- Sustentabilidade da engenharia

---

## Conclusão

Cost of Quality demonstra que qualidade não é custo adicional, mas investimento estratégico.

Ao equilibrar corretamente prevenção, avaliação e gestão de falhas, é possível reduzir riscos, aumentar confiabilidade e manter sustentabilidade técnica e financeira no longo prazo.