# Gerenciamento de Dados de Teste em E2E

## Origem do Conceito

Gerenciamento de dados de teste surgiu como necessidade crítica quando sistemas de software tornaram-se complexos e dependentes de bancos de dados persistentes.

Nos primórdios da automação de testes, equipes enfrentavam problemas recorrentes: testes falhando intermitentemente, interdependências entre testes, acúmulo de dados de teste no banco, impossibilidade de executar testes em paralelo.

A raiz desses problemas era a falta de estratégia clara para criar, isolar e limpar dados de teste. Conforme os sistemas cresceram e as suites de testes se expandiram, o gerenciamento de dados de teste evoluiu de um detalhe técnico para uma prática essencial que impacta diretamente na confiabilidade dos testes.

Hoje, em ambientes de teste E2E com dezenas ou centenas de testes, dados bem gerenciados são tão críticos quanto a automação em si.

---

## Conceito

Gerenciamento de dados de teste é o conjunto de estratégias e práticas para criar, manter, isolar e limpar dados necessários para execução de testes E2E.

Envolve três dimensões principais:

1. **Seeding**: População controlada de dados antes da execução do teste
2. **Isolamento**: Garantir que dados de um teste não afetam outro
3. **Limpeza**: Remoção de dados após execução para manter o ambiente limpo

O objetivo é permitir que cada teste execute em um estado de dados previsível e independente, garantindo execução confiável, rápida e reprodutível.

---

## Por Que Gerenciamento de Dados Importa

### Confiabilidade dos Testes

Testes E2E sem isolamento de dados falham intermitentemente por razões fora do controle da lógica testada.

Quando dois testes compartilham dados:
- Ordem de execução se torna crítica
- Falhas são não-determinísticas
- Debugar torna-se difícil
- Confiança na suite diminui

Isolamento de dados elimina essa classe inteira de problemas.

---

### Velocidade de Execução

Estratégias inadequadas de setup e teardown transformam testes em processos lentos.

Recriar todo o estado do banco para cada teste pode levar segundos. Multiplicado por centenas de testes, isso adiciona minutos ou horas no pipeline.

Estratégias inteligentes de gerenciamento (transações, snapshots, schema isolation) reduzem tempo de setup de segundos para milissegundos.

---

### Manutenibilidade

Dados de teste mal organizados espalhados por toda a suite dificultam manutenção.

Quando equipes não conseguem entender facilmente qual estado de dados um teste espera, modificações se tornam arriscadas. O código de teste fica frágil e acoplado.

Centralizar e documentar estratégia de dados torna testes auto-explicativos e fáceis de modificar.

---

### Paralelização

Sem isolamento de dados, testes não podem rodar em paralelo com segurança.

Ambiente compartilhado gera race conditions. Isso limita velocidade de pipeline e impede aproveitamento moderno de hardware.

Com isolamento apropriado, testes rodando em paralelo não interferem um no outro.

---

## Estratégias de Gerenciamento de Dados

### Estratégia 1: Banco de Dados Por Teste

Cada teste recebe sua própria instância isolada de banco de dados.

Pode ser alcançado através de:
- Containers (TestContainers)
- Clonagem de banco
- Réplicas efêmeras em cloud

**Benefícios:**
- Isolamento total
- Sem race conditions
- Paralelo-seguro

**Custos:**
- Recursos computacionais elevados
- Setup mais lento
- Escalabilidade limitada por hardware

**Quando usar:**
- Testes críticos que modificam múltiplos agregados
- Ambientes com recursos abundantes
- Pequenas suites de testes
- CI/CD com orçamento generoso

---

### Estratégia 2: Schema Por Teste

Cada teste recebe seu próprio schema SQL isolado no mesmo servidor de banco.

Reduz consumo de recursos mantendo isolamento aceitável.

**Benefícios:**
- Bom balanço entre isolamento e eficiência
- Paralelizável
- Recursos moderados

**Custos:**
- Setup ainda requer criação de schema
- Compatibilidade com bancos suportando schemas

**Quando usar:**
- Suites médias (100-500 testes)
- Quando performance é importante
- Ambientes com recursos limitados

---

### Estratégia 3: Rollback de Transação Por Teste

Cada teste executa dentro de uma transação que é revertida após execução.

**Benefícios:**
- Mais rápido (milissegundos de cleanup)
- Nenhum consumo adicional de recursos
- Simples de implementar

**Custos:**
- Limitado a um banco de dados
- Testes devem rodar sequencialmente
- Requer bancos que suportam savepoints

**Quando usar:**
- Suites grandes (500+ testes)
- Performance crítica
- Testes executando sequencialmente
- Ambientes com um único banco

---

### Estratégia 4: Restauração de Snapshot

Testes iniciam de snapshots pré-capturados do banco.

**Benefícios:**
- Muito rápido 
- Funciona bem com cloud databases
- Pouco overhead de execução

**Custos:**
- Esforço inicial de criar snapshots
- Gerenciar múltiplas versões
- Menos flexível para dados dinâmicos

**Quando usar:**
- Testes read-heavy
- Grandes conjuntos de dados necessários
- Cloud databases (RDS, Azure SQL)
- Performance é crítica

---

## Abordagens de Seeding de Dados

### Via SQL Direto

Scripts SQL para popular tabelas.

**Vantagem:** Familiar, simples
**Desvantagem:** Difícil parametrizar, risco de segurança

---

### Via Object Builders

Classes fluentes que constroem objetos de teste com defaults sensatos.

```javascript
const usuario = new UsuarioBuilder()
  .comEmail('test@example.com')
  .comRole('admin')
  .build();
```

**Vantagem:** Type-safe, expressivo
**Desvantagem:** Requer implementação

---

### Via Factory Functions

Funções simples que retornam dados com defaults.

```javascript
const usuario = criarUsuario({ role: 'admin' });
```

**Vantagem:** Simples, rápido
**Desvantagem:** Menos expressivo

---

### Via Fixtures Repositório

Conjuntos pré-definidos de dados para cenários comuns.

```javascript
const fixture = TestFixtures.USUARIO_ADMIN_COM_PEDIDOS;
```

**Vantagem:** Cenários claros, reutilizáveis
**Desvantagem:** Pode se tornar unmaintainable

---

## Anti-Patterns em Gerenciamento de Dados

### Compartilhamento Global de Dados

Múltiplos testes dependendo de dados persistidos em beforeAll().

**Problema:** Ordem de testes importa, sem paralelização possível

---

### Hard-Coded IDs e Valores

Embutir IDs específicos (user_id = 42) que podem não existir.

**Problema:** Testes quebram se dados não existem, ambiente-dependentes

---

### Limpeza Insuficiente

Esquecer de limpar dados entre testes, acumulando lixo no banco.

**Problema:** BD incha, queries degradam, testes ficam lentos

---

### Dependência de Serviços Externos

Usar APIs reais (Stripe, email) para gerar dados de teste.

**Problema:** Testes lentos, não-determinísticos, custosos

---

### Estado Inconsistente

Dados de teste violando regras de negócio ou constraintsdo banco.

**Problema:** Testes validam cenários inválidos, não pegam bugs reais

---

## Boas Práticas em Gerenciamento de Dados

### Usar Builders Para Criação

Centralizar criação de dados em classes com defaults inteligentes reduz repetição e torna testes legíveis.

---

### Implementar Isolamento Real

Cada teste deve ter seu próprio contexto de dados sem compartilhamento.

Mesmo que compartilhe infraestrutura (servidor BD), deve ter espaço isolado (schema, namespace, transação).

---

### Documentar Estrutura de Dados

Manter documentação clara de relacionamentos entre dados esperados por cada teste.

Reduz confusão, facilita modificações, previne quebras inesperadas.

---

### Validar Setup Antes do Teste

Verificar que dados foram criados corretamente antes de executar lógica de teste.

Falhas de setup detectadas cedo economizam debugging.

---

### Paralelizar Criação de Dados

Quando dados são independentes, criá-los em paralelo (Promise.all) acelera setup.

---

### Usar Transações Quando Possível

Se o banco suporta, transações com rollback oferecem melhor performance.

---

### Versionar Dados de Teste

Manter fixtures em version control com histórico de mudanças.

Facilita auditoria e rollback.

---

## Métricas Relevantes

### Tempo de Setup

Quanto tempo leva para preparar dados para um teste.

Meta: < 100ms por teste com transações, < 1s com container-por-teste

---

### Tempo de Cleanup

Quanto tempo leva limpar dados após execução.

Meta: Instantâneo com transações, < 500ms com schemas

---

### Taxa de Flakiness Relacionada a Dados

Percentual de testes falhando devido a estado de dados intermitente.

Meta: 0% (indica isolamento falho)

---

### Capacidade de Paralelização

Quanto da suite pode executar paralelamente sem race conditions.

Meta: 100% com isolamento apropriado

---

## Benefícios Estratégicos

- **Confiabilidade:** Testes com dados isolados não falham intermitentemente
- **Velocidade:** Estratégias eficientes reduzem tempo de pipeline em minutos
- **Manutenibilidade:** Dados bem organizados facilitam modificações
- **Escalabilidade:** Paralelização permite suites maiores sem crescimento linear de tempo
- **Debugabilidade:** Problemas de dados são rastreáveis e reprodutíveis

---

## Conclusão

Gerenciamento de dados de teste é tão crítico quanto a automação em si para suites E2E confiáveis e eficientes.

A escolha correta de estratégia (banco-por-teste, schema, transação ou snapshot) depende contexto: tamanho da suite, disponibilidade de recursos, velocidade requerida, necessidades de paralelização.

Independente da estratégia, os princípios permanecem constantes: isolamento, documentação, limpeza e validação.

Equipes que investem em gerenciamento estruturado de dados constroem suites E2E que são confiáveis, rápidas e sustentáveis no longo prazo.
