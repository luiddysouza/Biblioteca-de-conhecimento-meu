# Vibe Coding — Entregas com IA do Zero ao Deploy

> Ir rápido com IA não é apertar Tab no autocomplete. É saber conduzir o modelo, tomar decisões pelo caminho e ser capaz de explicar cada uma delas.

---

## Sumário

1. [O que é Vibe Coding (e o que não é)](#1-o-que-é-vibe-coding-e-o-que-não-é)
2. [A mentalidade certa para entregar com IA](#2-a-mentalidade-certa-para-entregar-com-ia)
3. [O fluxo: do zero ao deploy](#3-o-fluxo-do-zero-ao-deploy)
4. [Fase 1 — Entender antes de gerar](#4-fase-1--entender-antes-de-gerar)
5. [Fase 2 — Planejar com o modelo](#5-fase-2--planejar-com-o-modelo)
6. [Fase 3 — Implementar em ciclos curtos](#6-fase-3--implementar-em-ciclos-curtos)
7. [Fase 4 — Revisar e explicar as decisões](#7-fase-4--revisar-e-explicar-as-decisões)
8. [Fase 5 — Testar e validar](#8-fase-5--testar-e-validar)
9. [Fase 6 — Deploy com consciência](#9-fase-6--deploy-com-consciência)
10. [Explicando decisões tomadas pelo caminho](#10-explicando-decisões-tomadas-pelo-caminho)
11. [Armadilhas do Vibe Coding](#11-armadilhas-do-vibe-coding)
12. [Checklist de entrega com IA](#12-checklist-de-entrega-com-ai)

---

## 1. O que é Vibe Coding (e o que não é)

**Vibe Coding** é a prática de usar IA de forma intensiva e iterativa para implementar features rapidamente — mantendo o engenheiro no papel de decisor, não de digitador.

O termo foi cunhado por Andrej Karpathy (2025) para descrever um novo modo de trabalhar onde você se entrega ao fluxo colaborativo com o modelo.

### O que é

```
✅ Usar a IA para gerar a estrutura inicial e iterar em cima
✅ Pedir ao modelo para explorar alternativas de implementação
✅ Deixar o agente escrever código repetitivo enquanto você decide a arquitetura
✅ Ir do protótipo funcional ao código de produção num único ciclo de trabalho
✅ Revisar e ser capaz de defender cada decisão gerada
```

### O que NÃO é

```
❌ Aceitar todo código gerado sem entender o que foi feito
❌ Usar a IA para fugir de decisões técnicas difíceis
❌ Entregar código que você não consegue explicar
❌ Substituir raciocínio por geração indiscriminada
❌ "Funciona, então está certo" — sem testes, sem revisão
```

> A diferença entre Vibe Coding e "código jogado pela IA" é **quem está no controle das decisões**.

---

## 2. A mentalidade certa para entregar com IA

O engenheiro que entrega bem com IA não pensa em si como programador — pensa em si como **arquiteto e decisor**.

| Mentalidade antiga | Mentalidade com IA |
|---|---|
| "Preciso escrever esse código" | "Preciso que esse comportamento exista" |
| "Deixa eu pensar em como implementar" | "Deixa eu definir o que precisa acontecer" |
| "Estou com dúvida sobre essa sintaxe" | "Esse trecho atende ao requisito?" |
| O engenheiro gera o código | O engenheiro guia, revisa e decide |

### O que o engenheiro faz

- Define o **objetivo** e as **restrições**
- Decide a **arquitetura** (o modelo sugere, você escolhe)
- Avalia **tradeoffs** (performance vs. simplicidade, acoplamento vs. flexibilidade)
- Garante que o código **funciona e pode ser mantido**
- Explica **por que** cada decisão foi tomada

### O que a IA faz

- Gera implementações baseadas no contexto fornecido
- Sugere alternativas quando pedido
- Executa código repetitivo e boilerplate
- Busca padrões similares na base de código
- Aponta possíveis problemas (quando configurada para isso)

---

## 3. O fluxo: do zero ao deploy

```
[Requisito / Problema]
        │
        ▼
[Fase 1 — Entender]
  Qual problema real isso resolve?
  Qual é o contexto técnico atual?
        │
        ▼
[Fase 2 — Planejar com o modelo]
  Quais arquivos serão afetados?
  Qual arquitetura usar?
  Quais são os riscos?
        │
        ▼
[Fase 3 — Implementar em ciclos curtos]
  Gerar → Revisar → Ajustar → Repetir
        │
        ▼
[Fase 4 — Revisar e explicar decisões]
  Você consegue defender cada trecho?
        │
        ▼
[Fase 5 — Testar e validar]
  Testes automatizados + validação manual
        │
        ▼
[Fase 6 — Deploy com consciência]
  O que vai ao ar, o que está pendente, o que foi assumido
```

---

## 4. Fase 1 — Entender antes de gerar

Antes de abrir o chat, responda:

**Do lado do problema:**
- Qual comportamento precisa existir no produto ao final?
- Quem usa isso e com que frequência?
- Existe algo parecido já implementado que posso usar como referência?

**Do lado técnico:**
- Quais partes do código são afetadas?
- Há dependências externas (APIs, banco, outros serviços)?
- Existe alguma restrição técnica que não pode ser violada?

### Por que isso importa

```
Sem entendimento prévio:
  IA gera uma solução para o problema que você descreveu
  — não necessariamente para o problema real.

Exemplo:
  Você pede: "Adicione paginação na lista de produtos"
  IA implementa: paginação client-side com todos os dados carregados de uma vez

  O problema real era: lista lenta com 10.000 produtos
  A solução certa era: paginação server-side com query limitada no banco

  A IA não errou — você não contextualizou o problema certo.
```

---

## 5. Fase 2 — Planejar com o modelo

Use a IA para **pensar junto**, não só para gerar código.

### Como planejar com o modelo

```
Prompt de planejamento:
  "Preciso implementar [objetivo].
   O contexto atual é [descreva a arquitetura relevante].
   As restrições são [liste o que não pode mudar].
   
   Antes de gerar qualquer código:
   1. Liste os arquivos que precisarão ser criados ou modificados
   2. Descreva a abordagem que você usaria
   3. Aponte os principais riscos ou decisões que precisam ser feitas"
```

### O que avaliar no plano

- Os arquivos listados fazem sentido com a arquitetura do projeto?
- A abordagem segue os padrões já estabelecidos?
- Os riscos apontados são reais? Há riscos que o modelo não viu?
- Há decisões no plano que você ainda não tomou?

> Não avance para a implementação se ainda há decisões abertas. Feche as decisões primeiro.

### Dividindo em partes

Feature grandes devem ser quebradas em partes independentes que possam ser implementadas, testadas e commitadas separadamente:

```
Feature: Sistema de notificações

Parte 1: Modelo de dados da notificação
Parte 2: Serviço de criação e persistência
Parte 3: Lógica de disparo (quando notificar)
Parte 4: UI — listagem de notificações
Parte 5: UI — badge de não lidas
Parte 6: Marcar como lida
```

Cada parte tem contexto menor, é mais fácil de revisar e gera commits com significado.

---

## 6. Fase 3 — Implementar em ciclos curtos

### O ciclo básico

```
[Peça uma parte específica]
        │
        ▼
[Leia o que foi gerado — entenda, não apenas olhe]
        │
        ▼
[Identifique o que está errado ou incompleto]
        │
        ▼
[Corrija com contexto adicional, não com "tenta de novo"]
        │
        ▼
[Avance para a próxima parte]
```

### Como pedir implementações melhores

```
❌ Fraco:
   "Implemente a tela de listagem de produtos"

✅ Forte:
   "Implemente a ProductListPage em Flutter.
    - Use o ProductBloc que já existe em lib/features/products/bloc/
    - O estado ProductLoaded tem o campo List<Product> products
    - A UI deve ter um ListView.builder com ProductCard (já existe)
    - Em estado de loading, exibir um CircularProgressIndicator centralizado
    - Em estado de erro, exibir o ErrorWidget padrão do projeto
    - Não crie novos widgets — use os existentes"
```

### Quanto revisar por ciclo

Implemente no máximo uma responsabilidade por vez. Se o agente gerou 5 arquivos de uma vez, divida a revisão: entenda um por um antes de aceitar tudo junto.

---

## 7. Fase 4 — Revisar e explicar as decisões

Esta é a fase que separa Vibe Coding de código aleatório gerado por IA.

### A regra do "e se me perguntarem?"

Para cada trecho de código gerado, você deve conseguir responder:

1. **O que esse código faz?** (comportamento)
2. **Por que foi feito assim e não de outra forma?** (decisão)
3. **O que acontece se isso falhar?** (resiliência)
4. **Esse código segue os padrões do projeto?** (consistência)

Se você não consegue responder, não comite — entenda primeiro.

### Sinais de que a revisão foi superficial

```
❌ "Funciona no teste manual, então está ok"
❌ "O modelo gerou, deve estar certo"
❌ "Não entendi essa parte mas parece funcionar"
❌ "Vou entender depois se precisar mexer"
```

### Como aprofundar a revisão com a própria IA

```
"Explique o que esse trecho faz linha por linha"
"Por que você escolheu [abordagem X] em vez de [abordagem Y]?"
"Quais são os cenários de edge case que esse código não trata?"
"Existe alguma decisão aqui que eu deveria rever dada a arquitetura do projeto?"
```

---

## 8. Fase 5 — Testar e validar

### O que testar depois de implementar com IA

**Testes automatizados:**
- O comportamento principal está coberto por testes?
- Os edge cases foram considerados?
- Os testes foram gerados junto com o código ou depois?

**Validação manual:**
- O comportamento está correto no cenário principal?
- Os estados de loading, erro e vazio estão funcionando?
- A integração com as partes adjacentes do sistema está correta?

### Pedindo testes para a IA

```
"Agora escreva os testes para o ProductRepository que acabamos de implementar.
 - Cubra o cenário de sucesso com lista não vazia
 - Cubra o cenário de lista vazia
 - Cubra o cenário de erro de rede (lançar ProductException)
 - Use mockito para mockar o datasource
 - Siga a estrutura Arrange / Act / Assert"
```

> Testes gerados junto com o código têm mais contexto. Gere os testes enquanto a implementação ainda está fresca na conversa.

---

## 9. Fase 6 — Deploy com consciência

Antes de fazer o deploy, seja capaz de responder:

### O que está indo ao ar

- Qual comportamento novo ou alterado o usuário vai ver?
- Existe feature flag? Precisa de um?
- Tem migração de banco de dados envolvida?

### O que foi assumido e pode ser revisitado

- Quais decisões foram tomadas por conveniência (não por convicção)?
- O que é dívida técnica consciente que precisa ser registrada?
- O que foi simplificado para caber no prazo?

### O que monitorar depois

- Qual é o comportamento esperado em produção?
- Há logs suficientes para debugar se algo der errado?
- Qual é o plano de rollback se necessário?

---

## 10. Explicando decisões tomadas pelo caminho

Ser capaz de explicar o que foi implementado é tão importante quanto implementar.

### Estrutura para explicar uma decisão técnica

```
Contexto:
  "Precisávamos de [objetivo]. A situação atual era [estado anterior]."

Decisão:
  "Escolhemos [abordagem X]."

Alternativas consideradas:
  "Avaliamos [Y] e [Z], mas descartamos porque [razão concreta]."

Consequências:
  "Isso nos dá [vantagem]. O tradeoff é [limitação ou custo]."

Próximo passo (se houver):
  "Se a carga crescer acima de X, precisaremos revisitar [parte específica]."
```

### Exemplo real

```
Contexto:
  "Precisávamos exibir uma lista de produtos com filtros em tempo real.
   A lista pode ter até 5.000 itens."

Decisão:
  "Implementamos filtro server-side com debounce de 300ms no campo de busca."

Alternativas consideradas:
  "Avaliamos filtro client-side (mais simples), mas com 5.000 itens
   a performance no dispositivo seria ruim em aparelhos intermediários."

Consequências:
  "A busca é sempre precisa e leve no cliente. O custo é uma chamada de API
   a cada busca — aceitável dado o volume esperado de uso."

Próximo passo:
  "Se a latência da API for problema, podemos adicionar cache de resultados
   recentes no cliente com TTL de 1 minuto."
```

---

## 11. Armadilhas do Vibe Coding

### Geração em excesso sem revisão

Deixar o agente gerar arquivos em cadeia sem revisar cada passo cria uma base de código que ninguém entende — nem você, nem o modelo nas próximas interações.

### Contexto degradado ao longo da sessão

Conversas longas acumulam ruído. O modelo começa a misturar decisões de partes diferentes da feature. Quando a conversa fica longa demais, abra uma nova e resuma o contexto atual.

### Aceitar a primeira solução

A primeira implementação gerada é raramente a melhor. Use o modelo para comparar abordagens antes de escolher.

### Testes como formalidade

Pedir ao modelo para "escrever testes" sem contexto produz testes que cobrem o código gerado, não o comportamento esperado. Descreva os cenários que importam, não peça testes genéricos.

### Perder a capacidade de explicar

Se você entregou algo que não consegue defender, você não entregou — você repassou. A IA não está na reunião de review, você está.

---

## 12. Checklist de entrega com IA

### Antes de começar
- [ ] Entendi o problema real que precisa ser resolvido?
- [ ] Mapeei o contexto técnico relevante (arquivos, dependências, restrições)?
- [ ] Quebrei a feature em partes pequenas e independentes?

### Durante a implementação
- [ ] Cada parte foi revisada antes de avançar para a próxima?
- [ ] Consigo explicar o que cada trecho de código faz?
- [ ] Consigo explicar por que foi feito assim e não de outra forma?
- [ ] Os testes cobrem os cenários que importam (não só o caminho feliz)?

### Antes do deploy
- [ ] O comportamento foi validado manualmente nos cenários principais?
- [ ] As decisões de tradeoff foram registradas (comentário, ADR, ou PR description)?
- [ ] A dívida técnica consciente foi documentada?
- [ ] Há logs suficientes para diagnosticar problemas em produção?

### Após o deploy
- [ ] O comportamento em produção está como esperado?
- [ ] Alguém do time conseguiria dar manutenção nesse código sem você?

Ver [ia/context-engineering-e-llms.md](context-engineering-e-llms.md) para configurar o ambiente de IA que suporta esse fluxo de entrega.
