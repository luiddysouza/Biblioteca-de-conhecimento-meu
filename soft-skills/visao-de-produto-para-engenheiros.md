# Engenheiro com Visão de Produto

> Um dev com visão de produto não executa tarefas — resolve problemas reais. Sair da zona de executor começa com uma única mudança de pergunta: do "como?" para o "por quê?".

---

## Sumário

1. [A diferença de mentalidade](#1-a-diferença-de-mentalidade)
2. [Por que entender o negócio importa](#2-por-que-entender-o-negócio-importa)
3. [O ciclo: Problema → Solução → Produto](#3-o-ciclo-problema--solução--produto)
4. [Ferramentas para entender o contexto](#4-ferramentas-para-entender-o-contexto)
5. [Domain-Driven Design — Linguagem Ubíqua e Bounded Contexts](#5-domain-driven-design--linguagem-ubíqua-e-bounded-contexts)
6. [Event Storming e Jobs to Be Done](#6-event-storming-e-jobs-to-be-done)
7. [Os três eixos do perfil com visão de produto](#7-os-três-eixos-do-perfil-com-visão-de-produto)
8. [Do contexto ao backlog](#8-do-contexto-ao-backlog)
9. [OKRs como bússola de decisão](#9-okrs-como-bússola-de-decisão)
10. [Decisões técnicas e impacto no produto](#10-decisões-técnicas-e-impacto-no-produto)
11. [Armadilhas](#11-armadilhas)
12. [Desenvolvendo visão de produto no projeto atual](#12-desenvolvendo-visão-de-produto-no-projeto-atual)
13. [Exercícios práticos por situação](#13-exercícios-práticos-por-situação)
14. [O perfil resultante](#14-o-perfil-resultante)

---

## 1. A diferença de mentalidade

| Dev executor | Dev com visão de produto |
|---|---|
| "Preciso implementar esse botão." | "Por que esse botão existe? O que o usuário precisa fazer?" |
| "A task diz X, vou fazer X." | "X resolve o problema real? Existe forma mais simples?" |
| "O bug foi corrigido." | "Por que esse bug chegou em produção? O que precisamos mudar no processo?" |
| Entrega código | Entrega valor |

A virada de chave não é sobre saber mais tecnologia — é sobre **mudar a pergunta de partida**.

---

## 2. Por que entender o negócio importa

Um engenheiro que não entende o negócio constrói a coisa certa do jeito errado — ou pior, constrói a coisa errada perfeitamente.

```
❌ Sem contexto de negócio:
  PM: "Preciso de um botão de exportar para Excel"
  Dev: constrói o botão perfeito
  Resultado: ninguém usa, porque o problema real era que o relatório
             chegava tarde demais para as decisões diárias do time

✅ Com contexto de negócio:
  Dev: "Por que você precisa exportar para Excel? O que você faz com isso?"
  PM: "Preciso mostrar os números para o diretor toda manhã"
  Dev: propõe um dashboard automático com e-mail diário às 8h
  Resultado: resolve o problema real, sem exportação nenhuma
```

**A pergunta mais poderosa que um engenheiro pode fazer: "Por quê?"**

---

## 3. O ciclo: Problema → Solução → Produto

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

## 4. Ferramentas para entender o contexto

### 4.1 — As 5 Perguntas Fundamentais

Antes de qualquer estimativa ou arquitetura, responda:

| Pergunta | O que revela |
|---|---|
| **Qual é o problema?** | O pain point real, não o sintoma |
| **Quem sofre com ele?** | O usuário-alvo e seus comportamentos |
| **Qual é o custo do problema?** | Urgência e prioridade |
| **O que já foi tentado?** | Restrições e aprendizados anteriores |
| **Como vamos saber que resolvemos?** | Métricas de sucesso |

### 4.2 — Mapa de Stakeholders

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

### 4.3 — User Journey Map

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

## 5. Domain-Driven Design — Linguagem Ubíqua e Bounded Contexts

O maior problema na comunicação entre negócio e engenharia é que **falam idiomas diferentes**.

```
Negócio diz: "O cliente faz um pedido"
Engenharia cria: Order, Customer, Cart, Checkout, Purchase, Transaction...

Resultado: ninguém sabe ao certo o que é um "pedido" no código.
```

### Linguagem Ubíqua (Ubiquitous Language)

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

> Quando o código usa os mesmos termos que o negócio, qualquer pessoa entende o que uma classe `Order` ou um método `placeOrder()` faz.

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

## 6. Event Storming e Jobs to Be Done

### Event Storming

Event Storming é um workshop colaborativo para explorar e mapear um domínio de negócio usando **eventos de domínio** como ponto de partida.

```
Post-its laranjas  = Eventos de domínio   (algo que aconteceu, no passado)
Post-its azuis     = Comandos             (ação que causou o evento)
Post-its amarelos  = Atores               (quem executou o comando)
Post-its roxos     = Políticas            (regra de negócio automática)
Post-its rosa      = Sistemas externos    (integração de terceiros)
Post-its verdes    = Read models          (dado que o ator precisa ver)
```

Exemplo simplificado — fluxo de compra:

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

### Jobs to Be Done (JTBD)

As pessoas não compram produtos. Elas "contratam" produtos para fazer um trabalho.

Estrutura:
```
Quando [situação]
  eu quero [motivação]
  para que [resultado esperado]
```

Exemplo:
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

**Benefício do JTBD**: evita construir a feature pedida sem resolver o job real.

---

## 7. Os três eixos do perfil com visão de produto

### Eixo 1 — Entender o "porquê" antes do "como"

- Conhecer o usuário final, o contexto de negócio e as métricas que importam
- Questionar requisitos que não fazem sentido antes de implementá-los
- Saber dizer "não" (ou "de outra forma") quando a solução técnica não serve o produto

**Exemplo concreto:**

Um PM pede um campo de filtro com 12 opções numa lista. O dev executor implementa os 12 filtros. O dev com visão de produto pergunta: *"Quais filtros os usuários realmente usam? Temos dados disso?"* — e pode descobrir que 80% dos acessos usam apenas 2 filtros, simplificando a UI e reduzindo complexidade de código.

### Eixo 2 — Tomar decisões técnicas orientadas a impacto

- A escolha de arquitetura serve ao time e ao produto, não ao ego técnico
- Complexidade tem custo — simplicidade é uma feature
- Dívida técnica é uma decisão de negócio, não só um problema de código

**Exemplo concreto:**

O time precisa entregar uma feature de notificações em 1 semana. Você conhece uma solução elegante que levaria 3 semanas. A decisão com visão de produto é: implementar a solução "boa o suficiente" agora, registrar a dívida técnica conscientemente, e propor a refatoração no próximo ciclo — com justificativa de negócio (ex: "isso vai travar o roadmap Q3 se não fizermos").

### Eixo 3 — Comunicar através das fronteiras

- Traduzir linguagem técnica para stakeholders sem perder precisão
- Trazer perspectiva técnica para conversas de produto (viabilidade, custo, risco)
- Participar ativamente de definições antes de chegar no refinamento

**Exemplo concreto:**

Em vez de dizer *"a migração para a nova API vai exigir refatorar o repositório de dados"*, diga: *"Para usar a nova API, precisamos de ~3 dias de ajuste técnico. Se não fizermos isso agora, cada nova feature nessa área vai custar o dobro do tempo."* — isso gera uma decisão, não uma dúvida.

---

## 8. Do contexto ao backlog

### Épicos (visão estratégica)

Grandes objetivos de negócio, sem detalhe técnico:
```
Épico: Reduzir o tempo de onboarding de novos clientes de 5 dias para 1 dia
```

### Histórias de Usuário (visão do usuário)

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

### Tasks (visão do engenheiro)

As histórias se quebram em tasks técnicas. Aqui sim entra a implementação.

### Critérios de priorização — Matriz de Impacto × Esforço

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

## 9. OKRs como bússola de decisão

**OKR = Objective + Key Results**

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

### Como OKRs filtram decisões de produto

```
Feature solicitada: "Adicionar mais opções de parcelamento"

Sem OKR:  → equipe aceita, porque parece útil
Com OKR:  → "Isso reduz abandono de carrinho ou melhora o tempo de checkout?"
           → Se sim: prioriza
           → Se não: documenta para outro ciclo
```

> OKRs são filtros de prioridade. Se uma feature não move nenhum KR, é possível que ela não precise ser feita agora.

---

## 10. Decisões técnicas e impacto no produto

Decisões técnicas nunca são apenas técnicas. Toda escolha de arquitetura tem consequências diretas em **prazo**, **custo**, **escalabilidade** e **experiência do usuário**.

```
            Decisão de Arquitetura
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
     Prazo        Custo    Escalabilidade
        └───────────┴───────────┘
                    │
                    ▼
          Experiência do Usuário
              (consequência final)
```

### O eixo mais ignorado: UX

| Decisão técnica | Impacto na UX |
|---|---|
| Sem cache na listagem principal | Usuário espera 3s toda vez que abre a tela |
| Consistência eventual sem feedback | Usuário atualiza e o dado "volta" — parece bug |
| Tratamento genérico de erros | Usuário vê "erro desconhecido" sem saber o que fazer |
| Sem estado offline | App inutilizável em conexão ruim |
| Loading states descuidados | Usuário clica duas vezes achando que não funcionou |

### Exemplo: decisão com os quatro eixos

**Cenário:** escolher como implementar busca de produtos num app de e-commerce.

```
Opção A — Filtro client-side
  Prazo:          rápido (1 dia)
  Custo:          zero de infra adicional
  Escalabilidade: não escala — com 10k produtos, o app trava
  UX:             rápido no início; degrada conforme o catálogo cresce

Opção B — Busca server-side com query SQL
  Prazo:          médio (3 dias)
  Custo:          custo marginal de queries no banco existente
  Escalabilidade: aguenta bem até ~100k produtos com índice correto
  UX:             consistente, resultados corretos, leve no dispositivo

Opção C — Elasticsearch dedicado
  Prazo:          longo (2 semanas + configuração)
  Custo:          $200–500/mês de infra adicional
  Escalabilidade: ilimitada para busca textual
  UX:             busca mais relevante, tolerante a erros de digitação

Decisão com visão de produto:
  Hoje: catálogo tem 2k produtos → Opção B
  Critério de revisão: quando busca apresentar latência > 500ms ou catálogo
  ultrapassar 50k produtos, avaliar migração para Opção C.
```

### Como o negócio determina a arquitetura

| Contexto de negócio | Implicação arquitetural |
|---|---|
| Alta temporalidade (Black Friday) | Necessidade de auto-scaling |
| Regulatório (banco, saúde) | Auditoria, criptografia, compliance |
| Multi-tenant (SaaS) | Isolamento de dados por tenant |
| Domínios independentes | Bounded Contexts, possível microserviço |
| Time pequeno, validando mercado | Monólito, simplicidade acima de tudo |

Ver [hard-skills/arquitetura-de-sistemas-complexos.md](../hard-skills/arquitetura-de-sistemas-complexos.md) para aprofundar nos padrões e decisões de arquitetura.

---

## 11. Armadilhas

### Solucionite

Pular direto para a solução antes de entender o problema completamente.

> Remédio: passe mais tempo em perguntas. *"O que exatamente está causando esse problema?"*

### Feature Factory

O time entrega features sem medir se elas resolveram algum problema.

> Remédio: defina métricas de sucesso **antes** de construir. Revise após o lançamento.

### Proxy de usuário

Stakeholders internos definem o produto sem falar com usuários reais.

> Remédio: entrevistas com usuários, testes de usabilidade, análise de comportamento (analytics).

### Escopo inflado

"Vamos aproveitar e já adicionar X, Y e Z."

> Remédio: MVP real — o mínimo que valida a hipótese. Tudo que vai além é aposta.

### Mentalidade de executor

Receber a task, implementar, fechar — sem questionar se é o que o usuário precisa.

> Remédio: antes de qualquer implementação, responda: *"Se isso funcionar perfeitamente, o que o usuário consegue fazer que não conseguia antes?"*

### Perda de contexto no telefone sem fio

```
CEO → VP → PM → Tech Lead → Dev
"Aumentar retenção" vira "adicionar gamificação" vira "sistema de pontos complexo"
```

> Remédio: times pequenos com acesso direto ao problema. Documentar o "porquê" em cada nível.

---

## 12. Desenvolvendo visão de produto no projeto atual

Você não precisa mudar de projeto para desenvolver essa visão. Pode começar agora, onde está.

### Passo 1 — Entenda o produto que você está construindo

Perguntas que você deve conseguir responder sem consultar ninguém:

- Quem é o usuário principal do sistema?
- Qual o problema central que o produto resolve?
- Como o produto gera valor (ou receita)?
- Quais são as métricas de sucesso do produto?
- Qual a jornada principal do usuário?

Se você não sabe responder, esse é o primeiro exercício: **descubra**. Converse com o PM, leia documentação de produto, assista sessões de UX se houver.

### Passo 2 — Mapeie onde o código encontra o negócio

Em qualquer projeto, há pontos onde decisões técnicas têm impacto direto no produto:

- Regras de negócio implementadas no código (ex: cálculo de desconto, elegibilidade)
- Fluxos críticos para o usuário (ex: checkout, cadastro, autenticação)
- Integrações com sistemas externos (ex: pagamento, notificação)

**Exercício:** pegue uma feature que você implementou recentemente. Responda: *"Se isso falhar, o que o usuário não consegue fazer? Qual o impacto no negócio?"*

### Passo 3 — Adote o hábito de perguntar antes de implementar

Para cada task, antes de abrir o editor, responda:

1. **Por que isso existe?** — qual dor do usuário isso resolve
2. **Como o sucesso será medido?** — o que muda depois que isso for entregue
3. **Existe uma solução mais simples?** — que resolva 80% do problema com 20% do esforço
4. **Quais são os riscos?** — o que pode dar errado para o usuário

Isso leva 5 minutos e muda completamente a qualidade das decisões.

### Passo 4 — Participe do processo antes do refinamento

A maioria dos devs entra no processo no refinamento técnico. Dev com visão de produto entra antes:

- Peça para participar de conversas de discovery
- Leia os tickets antes do refinamento e chegue com perguntas
- Quando um requisito parecer estranho, questione — não implemente no piloto automático

---

## 13. Exercícios práticos por situação

### Situação: você recebeu uma task nova

**O "5 Porquês" rápido:**

```
Task: "Adicionar campo de CEP no cadastro"
Por quê? → Para calcular frete
Por quê isso agora? → Estamos lançando entrega regional
O que o usuário precisa? → Saber o custo de entrega antes de finalizar a compra

Conclusão: o campo de CEP é meio, não fim.
Pergunta válida: faz sentido pedir CEP no cadastro ou só no checkout?
```

---

### Situação: você está em um code review

Além de qualidade de código, pergunte:

- Esse código reflete corretamente a regra de negócio?
- Se a lógica estiver errada, qual o impacto para o usuário?
- A nomenclatura do código comunica o propósito de negócio?

```dart
// Nome técnico — o que faz, mas não por quê
bool validateField(String value) { ... }

// Nome orientado a negócio — comunica a intenção
bool isEligibleForDiscount(String couponCode) { ... }
```

---

### Situação: você está decidindo uma arquitetura

Antes de escolher um padrão ou biblioteca, responda:

- Qual o tamanho e vida útil esperada desse sistema?
- Quantas pessoas vão manter isso?
- Qual a frequência de mudança dessa parte do código?

```
Exemplo: implementar cache local de dados

Opção A: cache manual com Map em memória — simples, suficiente
Opção B: Hive com TTL e sincronização — robusto, complexo

Se o dado muda raramente e o app tem 2 devs → Opção A.
Se o dado é crítico e tem time maior → Opção B.

A decisão é de produto + contexto, não só técnica.
```

---

### Situação: você identificou um problema técnico

Antes de levar o problema ao time, monte um argumento com três partes:

1. **O que é** (técnico): "Temos N+1 queries no carregamento da lista de pedidos"
2. **O que causa** (produto): "A tela de pedidos demora 4s para carregar em conexões 4G"
3. **O que custa ignorar** (negócio): "Usuários com conexão lenta abandonam essa tela — potencial impacto na retenção"

Isso transforma um problema técnico em uma decisão de priorização.

---

### Situação: você está usando IA para codar

Antes de pedir código para a IA, responda você mesmo:

- O que essa função precisa fazer?
- Quais são as entradas e saídas esperadas?
- Quais casos de borda importam?

Depois use a IA para implementar — não para pensar no lugar.

```
Ruim: "Crie uma função para calcular desconto"

Bom: "Crie uma função que recebe o valor do pedido (double) e o tipo de cliente
(enum: regular, premium, vip). Regras: premium tem 10% de desconto, vip tem 20%,
regular não tem desconto. O desconto não pode deixar o valor abaixo de R$5,00."
```

A diferença é que no segundo caso, **você já pensou o problema** — a IA só executa.

Ver [ia/vibe-coding-entregas-com-ai.md](../ia/vibe-coding-entregas-com-ai.md) para o processo completo de entrega com IA.

---

## 14. O perfil resultante

Um dev nesse nível age como um **multiplicador**: cada decisão considera código, produto e pessoas ao mesmo tempo.

```
Dev executor        →  entrega código
Dev sênior          →  entrega código de qualidade
Dev com visão       →  entrega valor com código de qualidade
Dev com visão + IA  →  entrega valor, mais rápido, com menos fricção
```

A IA amplifica a **velocidade de execução**.
A visão de produto amplifica a **relevância do que é executado**.

Os dois juntos criam alguém capaz de ir do problema do usuário ao código funcionando com muito mais autonomia — e muito menos retrabalho.

> **Resumo prático:** antes de qualquer task, gaste 5 minutos entendendo o "porquê". Use a IA para comprimir execução, não para substituir esse entendimento. Com o tempo, isso vira instinto.

---

## Checklist

- [ ] Consigo explicar qual problema o produto resolve sem consultar ninguém
- [ ] Questiono requisitos que não fazem sentido antes de implementar
- [ ] Defino métricas de sucesso antes de construir
- [ ] Conheço os stakeholders e seus interesses conflitantes
- [ ] Uso linguagem ubíqua no código (termos do domínio, não termos técnicos genéricos)
- [ ] Avalio decisões técnicas pelos quatro eixos: prazo, custo, escalabilidade, UX
- [ ] Traduzo problemas técnicos para impacto de produto ao comunicar ao time
- [ ] Antes de usar IA, já decidi o que precisa ser feito — a IA só executa
