# Entendendo Contexto de Negócio e Traduzindo em Produto

> Como sair da zona de "executor de tarefas" e começar a pensar como quem constrói soluções reais para problemas reais.

---

## Sumário

1. [Por que entender o negócio importa](#1-por-que-entender-o-negócio-importa)
2. [O ciclo: Problema → Solução → Produto](#2-o-ciclo-problema--solução--produto)
3. [Ferramentas para entender o contexto](#3-ferramentas-para-entender-o-contexto)
4. [Domain-Driven Design — Linguagem Ubíqua](#4-domain-driven-design--linguagem-ubíqua)
5. [Event Storming](#5-event-storming)
6. [Jobs to Be Done (JTBD)](#6-jobs-to-be-done-jtbd)
7. [De contexto para backlog](#7-de-contexto-para-backlog)
8. [OKRs e como conectar estratégia com execução](#8-okrs-e-como-conectar-estratégia-com-execução)
9. [Armadilhas ao traduzir negócio em produto](#9-armadilhas-ao-traduzir-negócio-em-produto)

---

## 1. Por que entender o negócio importa

Um engenheiro que não entende o negócio constrói a coisa certa do jeito errado — ou pior, constrói a coisa errada perfeitamente.

```
❌ Cenário sem contexto de negócio:
  PM: "Preciso de um botão de exportar para Excel"
  Dev: constrói o botão de exportar para Excel perfeito
  Resultado: ninguém usa, porque o problema real era que o relatório
             chegava tarde demais para as decisões diárias do time

✅ Cenário com contexto de negócio:
  Dev: "Por que você precisa exportar para Excel? O que você faz com isso?"
  PM: "Preciso mostrar os números para o diretor toda manhã"
  Dev: propõe um dashboard automático com e-mail diário às 8h
  Resultado: resolve o problema real, sem exportação nenhuma
```

**A pergunta mais poderosa que um engenheiro pode fazer: "Por quê?"**

---

## 2. O ciclo: Problema → Solução → Produto

```
[Negócio tem um problema]
          │
          ▼
[Entender: quem sofre, como sofre, quanto custa esse sofrimento]
          │
          ▼
[Formular hipótese de solução]
          │
          ▼
[Validar hipótese com o mínimo de esforço (MVP / Spike / Protótipo)]
          │
          ▼
[Construir o produto iterativamente]
          │
          ▼
[Medir se o problema foi resolvido]
          │
          ▼
[Refinar ou pivotar]
```

> O produto nunca é o objetivo. O objetivo é resolver o problema do negócio. O produto é apenas o veículo.

---

## 3. Ferramentas para entender o contexto

### 3.1 — As 5 Perguntas Fundamentais

Antes de qualquer estimativa ou arquitetura, responda:

| Pergunta | O que revela |
|---|---|
| **Qual é o problema?** | O pain point real, não o sintoma |
| **Quem sofre com ele?** | O usuário-alvo e seus comportamentos |
| **Qual é o custo do problema?** | Urgência e prioridade |
| **O que já foi tentado?** | Restrições e aprendizados anteriores |
| **Como vamos saber que resolvemos?** | Métricas de sucesso |

### 3.2 — Mapa de Stakeholders

```
                    [CEO / Diretoria]
                    Quer: crescimento, margem
                         │
           ┌─────────────┼─────────────┐
           ▼             ▼             ▼
       [Vendas]      [Produto]     [Operações]
       Quer: CRM     Quer: NPS     Quer: eficiência
       rápido        alto          operacional
           │             │             │
           └─────────────┼─────────────┘
                         ▼
                  [Time de Eng]
                  Traduz tudo isso
                  em software
```

Entenda o que cada stakeholder quer e por quê. Os conflitos entre eles geram as decisões mais difíceis de produto.

### 3.3 — User Journey Map

Mapeie a jornada completa do usuário, do problema até o resultado desejado:

```
Persona: Maria, gestora de RH de uma empresa de 200 funcionários

[Contexto]       Maria precisa processar folha de pagamento
     │
     ▼
[Ação]           Abre o sistema → Coleta dados de ponto → Calcula descontos
     │
     ▼
[Dor atual]      ⚠️ Processo manual em planilha leva 2 dias
                 ⚠️ Erros frequentes geram retrabalho e reclamações
     │
     ▼
[Resultado desejado]  Processar folha em horas, com zero erro de cálculo
```

---

## 4. Domain-Driven Design — Linguagem Ubíqua

O maior problema na comunicação entre negócio e engenharia é que **falam idiomas diferentes**.

### O problema

```
Negócio diz: "O cliente faz um pedido"
Engenharia cria: Order, Customer, Cart, Checkout, Purchase, Transaction...

Resultado: ninguém sabe ao certo o que é um "pedido" no código.
```

### A solução: Linguagem Ubíqua (Ubiquitous Language)

Crie um glossário compartilhado entre negócio e engenharia. Todo mundo usa os mesmos termos.

```
Glossário do domínio de Vendas:

Pedido (Order):
  Uma solicitação de compra feita por um Cliente, contendo
  um ou mais Itens, com status que vai de Rascunho até Entregue.

Cliente (Customer):
  Pessoa ou empresa cadastrada que já realizou ao menos um Pedido.

Prospecto (Prospect):
  Pessoa ou empresa em processo de negociação, ainda sem Pedido confirmado.
  ⚠️ Não é um Cliente — termos diferentes têm modelos de dados diferentes.
```

> Quando o código usa os mesmos termos que o negócio, qualquer pessoa consegue entender o que uma classe `Order` ou um método `placeOrder()` faz.

### Bounded Contexts

Nem sempre o mesmo termo tem o mesmo significado em diferentes partes do negócio.

```
"Produto" no contexto de Catálogo:
  → tem nome, descrição, fotos, categorias

"Produto" no contexto de Estoque:
  → tem SKU, localização no armazém, quantidade disponível

"Produto" no contexto de Faturamento:
  → tem preço, alíquota de imposto, NCM

Solução: três Bounded Contexts diferentes, cada um com seu modelo de Produto.
Eles se comunicam via eventos, não compartilhando o mesmo objeto.
```

---

## 5. Event Storming

Event Storming é um workshop colaborativo para explorar e mapear um domínio de negócio usando **eventos de domínio** como ponto de partida.

### Como funciona

```
Post-its laranjas  = Eventos de domínio   (algo que aconteceu, no passado)
Post-its azuis     = Comandos             (ação que causou o evento)
Post-its amarelos  = Atores               (quem executou o comando)
Post-its roxos     = Políticas            (regra de negócio automática)
Post-its rosa      = Sistemas externos    (integração de terceiros)
Post-its verdes    = Read models          (dado que o ator precisa ver)
```

### Exemplo simplificado: fluxo de compra

```
[Cliente]
    │
    │ Faz pedido
    ▼
[Comando: PlacedOrder]
    │
    ▼
[Evento: PedidoCriado] ──→ [Política: Se estoque disponível, reservar]
                                    │
                                    ▼
                          [Evento: EstoqueReservado] ──→ [Política: Cobrar pagamento]
                                                                  │
                                                                  ▼
                                                        [Evento: PagamentoAprovado]
                                                                  │
                                                                  ▼
                                                        [Evento: PedidoConfirmado]
```

> Event Storming é excelente para descobrir **onde estão os Bounded Contexts** e quais são os eventos mais críticos do negócio.

---

## 6. Jobs to Be Done (JTBD)

As pessoas não compram produtos. Elas "contratam" produtos para fazer um trabalho.

### A estrutura

```
Quando [situação]
  eu quero [motivação]
  para que [resultado esperado]
```

### Exemplo

```
Feature request recebido: "Quero notificações push"

JTBD real:
  "Quando meu pedido tem alguma atualização,
   eu quero ser avisado imediatamente,
   para que eu não precise ficar entrando no app para verificar"

Implicação técnica:
  → Pode ser push notification
  → Pode ser e-mail
  → Pode ser SMS
  → Pode ser um badge no ícone do app
  O "como" é menos importante que o "para quê"
```

**Benefício do JTBD**: evita que você construa a feature pedida sem resolver o job real.

---

## 7. De contexto para backlog

Depois de entender o contexto, o processo de construir o backlog:

### 1. Épicos (visão estratégica)

Grandes objetivos de negócio, sem detalhe técnico:
```
Épico: Reduzir o tempo de onboarding de novos clientes de 5 dias para 1 dia
```

### 2. Histórias de Usuário (visão do usuário)

```
Como [persona]
Eu quero [ação]
Para que [benefício de negócio]

Critérios de aceitação:
- [ ] ...
- [ ] ...
```

Exemplo:
```
Como operador de cadastro,
eu quero importar dados do cliente via planilha Excel,
para que eu não precise digitar cada campo manualmente.

Critérios de aceitação:
- [ ] Aceita arquivos .xlsx e .csv
- [ ] Valida os campos obrigatórios antes de importar
- [ ] Exibe relatório de erros por linha no caso de dados inválidos
- [ ] Importações de até 500 registros completam em menos de 30 segundos
```

### 3. Tasks (visão do engenheiro)

As histórias se quebram em tasks técnicas. Aqui sim entra a implementação.

### Critérios de priorização — Matriz de Impacto x Esforço

```
                Alto Impacto
                     │
  Faça depois   ─────┼───── Faça primeiro ✅
  (se necessário)    │      (quick wins + high value)
                     │
Alto Esforço ────────┼──────────────── Baixo Esforço
                     │
  Descarte ❌   ─────┼───── Faça se sobrar tempo
  (waste)            │
                     │
                Baixo Impacto
```

---

## 8. OKRs e como conectar estratégia com execução

**OKR = Objective + Key Results**

Estrutura:
```
Objective:  Uma declaração qualitativa e inspiradora de onde queremos chegar
Key Result: Uma métrica quantitativa que mede o progresso em direção ao objetivo
```

Exemplo:
```
Objetivo:    Tornar a experiência de checkout referência no mercado
Key Results:
  KR1: Reduzir o abandono de carrinho de 68% para 45%
  KR2: Aumentar o NPS do fluxo de pagamento de 32 para 60
  KR3: Reduzir o tempo médio de checkout de 4 min para 90 seg
```

### Como OKRs guiam decisões de produto

```
Feature solicitada: "Adicionar mais opções de parcelamento"

Sem OKR:  → equipe aceita, porque parece útil
Com OKR:  → "Isso reduz abandono de carrinho ou melhora o tempo de checkout?"
           → Se sim: prioriza
           → Se não: documenta para outro ciclo
```

> OKRs são filtros de prioridade. Se uma feature não move nenhum KR, é possível que ela não precise ser feita agora.

---

## 9. Armadilhas ao traduzir negócio em produto

### Solucionite
Pular direto para a solução antes de entender o problema completamente.
> Remédio: passe mais tempo em perguntas. "O que exatamente está causando esse problema?"

### Feature Factory
O time entrega features sem medir se elas resolveram algum problema.
> Remédio: defina métricas de sucesso ANTES de construir. Revise após o lançamento.

### Proxy de usuário
Stakeholders internos definem o produto sem falar com usuários reais.
> Remédio: entrevistas com usuários, testes de usabilidade, análise de comportamento (analytics).

### Escopo inflado
"Vamos aproveitar e já adicionar X, Y e Z"
> Remédio: MVP real — o mínimo que valida a hipótese. Tudo que vai além é aposta.

### Perda de contexto no telefone sem fio
```
CEO → VP → PM → Tech Lead → Dev
"Aumentar retenção" vira "adicionar gamificação" vira "sistema de pontos complexo"
```
> Remédio: times pequenos com acesso direto ao problema. Documentar o "porquê" em cada nível.

---

## Conexão com arquitetura

O contexto de negócio determina diretamente decisões de arquitetura:

| Contexto de negócio | Implicação arquitetural |
|---|---|
| Alta temporalidade (Black Friday) | Necessidade de auto-scaling |
| Regulatório (banco, saúde) | Auditoria, criptografia, compliance |
| Multi-tenant (SaaS) | Isolamento de dados por tenant |
| Domínios independentes | Bounded Contexts, possível microserviço |
| Time pequeno, validando mercado | Monólito, simplicidade acima de tudo |

> Ver [arquitetura-de-sistemas-complexos.md](arquitetura-de-sistemas-complexos.md) para aprofundamento nas decisões técnicas.
