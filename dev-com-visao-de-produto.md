# Dev com Visão de Produto + IA como Ferramenta

> Um dev com visão de produto não executa tarefas — resolve problemas reais.

---

## Sumário

1. [A diferença de mentalidade](#1-a-diferença-de-mentalidade)
2. [Os três eixos que definem esse perfil](#2-os-três-eixos-que-definem-esse-perfil)
3. [A IA nesse contexto](#3-a-ia-nesse-contexto)
4. [Como desenvolver visão de produto no projeto atual](#4-como-desenvolver-visão-de-produto-no-projeto-atual)
5. [Exercícios práticos por situação](#5-exercícios-práticos-por-situação)
6. [O perfil resultante](#6-o-perfil-resultante)

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

## 2. Os três eixos que definem esse perfil

### Eixo 1 — Entender o "porquê" antes do "como"

- Conhecer o usuário final, o contexto de negócio e as métricas que importam
- Questionar requisitos que não fazem sentido antes de implementá-los
- Saber dizer "não" (ou "de outra forma") quando a solução técnica não serve o produto

**Exemplo concreto:**

Um PM pede um campo de filtro com 12 opções numa lista. O dev executor implementa os 12 filtros. O dev com visão de produto pergunta: *"Quais filtros os usuários realmente usam? Temos dados disso?"* — e pode descobrir que 80% dos acessos usam apenas 2 filtros, simplificando a UI e reduzindo complexidade de código.

---

### Eixo 2 — Tomar decisões técnicas orientadas a impacto

- A escolha de arquitetura serve ao time e ao produto, não ao ego técnico
- Complexidade tem custo — simplicidade é uma feature
- Dívida técnica é uma decisão de negócio, não só um problema de código

**Exemplo concreto:**

O time precisa entregar uma feature de notificações em 1 semana. Você conhece uma solução elegante que levaria 3 semanas. A decisão com visão de produto é: implementar a solução "boa o suficiente" agora, registrar a dívida técnica conscientemente, e propor a refatoração no próximo ciclo — com justificativa de negócio (ex: "isso vai travar o roadmap Q3 se não fizermos").

---

### Eixo 3 — Comunicar através das fronteiras

- Traduzir linguagem técnica para stakeholders sem perder precisão
- Trazer perspectiva técnica para conversas de produto (viabilidade, custo, risco)
- Participar ativamente de definições antes de chegar no refinamento

**Exemplo concreto:**

Em vez de dizer *"a migração para a nova API vai exigir refatorar o repositório de dados"*, você diz: *"Para usar a nova API, precisamos de ~3 dias de ajuste técnico. Se não fizermos isso agora, cada nova feature nessa área vai custar o dobro do tempo."* — isso gera uma decisão, não uma dúvida.

---

## 3. A IA nesse contexto

A IA muda o que é **caro** de fazer, mas não muda o que é **valioso** de pensar.

| Antes da IA | Com IA como ferramenta |
|---|---|
| Escrever boilerplate era caro | Boilerplate é commoditizado |
| Pesquisar documentação era lento | Contexto técnico é instantâneo |
| Prototipar levava dias | Prototipação levou horas |
| O julgamento era diferencial | **O julgamento é o único diferencial** |

### Como usar a IA com visão de produto

**Para comprimir validação:**
```
Prompt útil: "Tenho esse problema de UX [descreva]. Quais abordagens comuns existem
para resolvê-lo? Quais os trade-offs de cada uma?"
```
Você não terceiriza a decisão — você acelera o levantamento de opções.

**Para questionar requisitos:**
```
Prompt útil: "Esse requisito foi descrito assim [cole o requisito]. Que perguntas
eu deveria fazer ao PM antes de começar a implementar?"
```

**Para pensar em impacto antes de codar:**
```
Prompt útil: "Vou implementar [feature X]. Que casos de borda, impactos em outras
partes do sistema ou riscos de produto eu deveria considerar primeiro?"
```

### Riscos a evitar

- **Aceitar código gerado sem entender** — você perde o modelo mental do sistema
- **Usar IA para responder "como fazer" antes de resolver "o que fazer"**
- **Delegar decisões de arquitetura ou produto para o modelo** — ele não conhece seu contexto

---

## 4. Como desenvolver visão de produto no projeto atual

Você não precisa mudar de projeto para desenvolver essa visão. Pode começar agora, onde está.

### Passo 1 — Entenda o produto que você está construindo

Perguntas que você deve conseguir responder sem consultar ninguém:

- Quem é o usuário principal do sistema?
- Qual o problema central que o produto resolve?
- Como o produto gera valor (ou receita)?
- Quais são as métricas de sucesso do produto?
- Qual a jornada principal do usuário?

Se você não sabe responder, esse é o primeiro exercício: **descubra**. Converse com o PM, leia documentação de produto, assista sessões de UX se houver.

---

### Passo 2 — Mapeie onde o código encontra o negócio

Em qualquer projeto, há pontos onde decisões técnicas têm impacto direto no produto:

- Regras de negócio implementadas no código (ex: cálculo de desconto, elegibilidade)
- Fluxos críticos para o usuário (ex: checkout, cadastro, autenticação)
- Integrações com sistemas externos (ex: pagamento, notificação)

**Exercício:** Pegue uma feature que você implementou recentemente. Responda: *"Se isso falhar, o que o usuário não consegue fazer? Qual o impacto no negócio?"*

---

### Passo 3 — Adote o hábito de perguntar antes de implementar

Para cada task que chegar, antes de abrir o editor, responda:

1. **Por que isso existe?** — qual dor do usuário isso resolve
2. **Como o sucesso será medido?** — o que muda depois que isso for entregue
3. **Existe uma solução mais simples?** — que resolva 80% do problema com 20% do esforço
4. **Quais são os riscos?** — o que pode dar errado para o usuário

Isso leva 5 minutos e muda completamente a qualidade das decisões.

---

### Passo 4 — Participe do processo antes do refinamento

A maioria dos devs entra no processo no refinamento técnico. Dev com visão de produto entra antes:

- Peça para participar de conversas de discovery
- Leia os tickets antes do refinamento e chegue com perguntas
- Quando um requisito parecer estranho, questione — não implemente no piloto automático

---

## 5. Exercícios práticos por situação

### Situação: Você recebeu uma task nova

**Exercício — O "5 Porquês" rápido:**

Pegue o título da task e pergunte "por quê?" até chegar na necessidade real do usuário.

```
Task: "Adicionar campo de CEP no cadastro"
Por quê? → Para calcular frete
Por quê isso agora? → Estamos lançando entrega regional
O que o usuário precisa? → Saber o custo de entrega antes de finalizar a compra

Conclusão: o campo de CEP é meio, não fim.
Pergunta válida: faz sentido pedir CEP no cadastro ou só no checkout?
```

---

### Situação: Você está em um code review

**Exercício — Revisar com olhos de produto:**

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

### Situação: Você está decidindo uma arquitetura

**Exercício — Decisão com contexto de produto:**

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

### Situação: Você identificou um problema técnico

**Exercício — Traduzir para impacto:**

Antes de levar o problema ao time, monte um argumento com três partes:

1. **O que é** (técnico): "Temos N+1 queries no carregamento da lista de pedidos"
2. **O que causa** (produto): "A tela de pedidos demora 4s para carregar em conexões 4G"
3. **O que custa ignorar** (negócio): "Usuários com conexão lenta abandonam essa tela — potencial impacto na retenção"

Isso transforma um problema técnico em uma decisão de priorização.

---

### Situação: Você está usando IA para codar

**Exercício — Perguntar antes de gerar:**

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

---

## 6. O perfil resultante

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

---

> **Resumo prático:** antes de qualquer task, gaste 5 minutos entendendo o "porquê". Use a IA para comprimir execução, não para substituir esse entendimento. Com o tempo, isso vira instinto.
