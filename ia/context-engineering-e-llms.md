# Context Engineering e LLMs

> Saber usar uma IA não é escrever prompts bonitos — é entender o que entra no modelo, o que fica de fora e por quê isso muda tudo.

---

## Sumário

1. [O que é Context Engineering](#1-o-que-é-context-engineering)
2. [A janela de contexto — o que entra e o que não entra](#2-a-janela-de-contexto--o-que-entra-e-o-que-não-entra)
3. [Configurando o ambiente de trabalho com IA](#3-configurando-o-ambiente-de-trabalho-com-ia)
4. [Rules — definindo o comportamento base](#4-rules--definindo-o-comportamento-base)
5. [Skills — capacidades especializadas](#5-skills--capacidades-especializadas)
6. [Hooks — IA reagindo a eventos](#6-hooks--ia-reagindo-a-eventos)
7. [Agents — IA tomando decisões e agindo](#7-agents--ia-tomando-decisões-e-agindo)
8. [Construindo contexto de qualidade](#8-construindo-contexto-de-qualidade)
9. [Criando workflows com IA](#9-criando-workflows-com-ia)
10. [Armadilhas comuns](#10-armadilhas-comuns)
11. [Checklist antes de trabalhar com um LLM](#11-checklist-antes-de-trabalhar-com-um-llm)

---

## 1. O que é Context Engineering

**Context Engineering** é a prática de projetar, estruturar e gerenciar o que uma IA recebe como entrada para que ela produza saídas úteis de forma consistente.

É diferente de **Prompt Engineering**:

| Prompt Engineering | Context Engineering |
|---|---|
| Foco em escrever um bom prompt | Foco em todo o sistema de informação que chega ao modelo |
| Pontual — uma interação | Sistêmico — fluxo inteiro |
| "Como peço isso ao modelo?" | "O que o modelo precisa saber para responder bem, sempre?" |
| Artesanal | Arquitetural |

> Context Engineering é sobre **sistemas**, não sobre frases bem escritas.

O contexto inclui tudo que o modelo recebe antes de gerar a resposta: instruções de sistema, histórico de conversa, arquivos abertos, resultados de ferramentas, memórias injetadas, e o próprio prompt do usuário.

---

## 2. A janela de contexto — o que entra e o que não entra

Todo LLM tem uma **janela de contexto** — o limite de tokens que ele consegue processar de uma vez. O que fica de fora dessa janela simplesmente não existe para o modelo.

```
┌────────────────────────────────────────────────────┐
│                  JANELA DE CONTEXTO                │
│                                                    │
│  [System prompt / Rules]                           │
│  [Memórias / Instruções persistentes]              │
│  [Histórico da conversa]                           │
│  [Arquivos / Código abertos / Recuperados]         │
│  [Resultados de ferramentas chamadas]              │
│  [Prompt atual do usuário]          ← você está aqui│
│                                                    │
│  Limite: X tokens — o que ultrapassar é descartado │
└────────────────────────────────────────────────────┘
```

### O que entra no contexto (e você controla)

- **System prompt / Rules**: instruções base de comportamento
- **Arquivos explicitamente referenciados**: o que você abre ou menciona
- **Histórico da conversa**: mensagens anteriores (até o limite)
- **Resultados de ferramentas**: o que o agente buscou, executou, leu
- **Memórias injetadas**: notas persistentes que o sistema adiciona automaticamente

### O que NÃO entra (e causa confusão)

- Arquivos que não foram referenciados
- Histórico além do limite da janela (o modelo "esquece")
- Conhecimento que você assume que o modelo tem mas não foi fornecido
- Decisões tomadas em conversas anteriores (a menos que salvas em memória)

### Por que isso importa na prática

```
Problema comum:
  Você pede ao Copilot para refatorar uma função.
  Ele refatora — mas quebra um contrato com outro módulo.

Por quê?
  O arquivo do outro módulo não estava no contexto.
  O modelo não sabia que aquele contrato existia.

Solução:
  Adicionar o arquivo relacionado ao contexto antes de pedir a refatoração.
```

---

## 3. Configurando o ambiente de trabalho com IA

Ferramentas como **GitHub Copilot**, **Cursor** e **Windsurf** permitem configurar o comportamento da IA além do chat — criando um ambiente onde ela entende seu projeto, suas convenções e seus fluxos de trabalho.

Os quatro pilares de configuração são: **Rules**, **Skills**, **Hooks** e **Agents**.

```
Ambiente de IA bem configurado:

  [Rules]     → "Como você deve se comportar sempre"
  [Skills]    → "O que você sabe fazer de forma especializada"
  [Hooks]     → "O que você faz automaticamente quando X acontece"
  [Agents]    → "Quem você pode chamar para executar tarefas complexas"
```

---

## 4. Rules — definindo o comportamento base

Rules são instruções persistentes que moldam como a IA se comporta em todo o projeto. Elas são injetadas automaticamente no contexto a cada interação.

### O que colocar em Rules

- Linguagem e framework do projeto
- Convenções de código (nomenclatura, estrutura de pastas, padrões)
- Restrições (o que nunca fazer)
- Tom e formato das respostas
- Contexto crítico do negócio que a IA precisa saber sempre

### Exemplo prático (Cursor / Copilot Instructions)

```markdown
# Regras do projeto

## Stack
- Flutter 3.x com Dart, arquitetura Clean + BLoC
- Backend em Node.js com TypeScript

## Convenções
- Nomes de classes: PascalCase
- Nomes de arquivos: snake_case
- BLoC: eventos terminam em Event, estados em State
- Nunca usar StatefulWidget — preferir BLoC ou Riverpod

## Restrições
- Nunca instanciar dependências diretamente — usar injeção via GetIt
- Não criar lógica de negócio em widgets

## Contexto
- Este é um app de marketplace B2B. "Produto" significa item do catálogo, não produto de software.
```

### Boa Rule vs. Rule ruim

| Rule ruim | Rule boa |
|---|---|
| "Escreva código limpo" | "Funções com mais de 20 linhas devem ser quebradas" |
| "Siga as boas práticas" | "Use o padrão Repository para acesso a dados" |
| "Seja conciso" | "Respostas de explicação devem ter no máximo 3 parágrafos" |
| Vago | Específico e verificável |

---

## 5. Skills — capacidades especializadas

Skills são instruções focadas em **como executar uma tarefa específica com qualidade**. São como playbooks: definem o passo a passo que a IA deve seguir para um determinado tipo de trabalho.

### Diferença entre Rules e Skills

| Rules | Skills |
|---|---|
| Sempre ativas | Ativadas para tarefas específicas |
| Define comportamento geral | Define como executar bem uma tarefa |
| "Como você é" | "Como você faz X" |

### Exemplos de Skills úteis

```
skill: criar-pr.md
  → Como estruturar o título e a descrição de um Pull Request
  → O que incluir (contexto, decisões, como testar)
  → Checklist de itens obrigatórios

skill: revisar-codigo.md
  → O que checar em uma code review
  → Como comentar sem ser agressivo
  → Prioridade: segurança > correção > performance > estilo

skill: escrever-teste.md
  → Estrutura Arrange / Act / Assert
  → O que deve ser mockado e o que não deve
  → Nome do teste: deve_[comportamento]_quando_[condição]
```

### Como a IA usa uma Skill

A Skill é carregada no contexto quando você referencia explicitamente a tarefa, ou quando o agente identifica que aquele tipo de trabalho está sendo solicitado. Ela guia o raciocínio do modelo para um output mais consistente e de qualidade.

---

## 6. Hooks — IA reagindo a eventos

Hooks permitem que a IA execute ações automaticamente em resposta a eventos do ambiente de desenvolvimento — sem você precisar pedir manualmente.

### Exemplos de Hooks

```
on: git commit
  → Revisar as mudanças e sugerir uma mensagem de commit no padrão Conventional Commits

on: file save (*.test.dart)
  → Verificar se o teste segue a estrutura definida na skill de testes

on: PR criado
  → Gerar automaticamente o resumo do PR com contexto das mudanças

on: build failed
  → Analisar o erro e sugerir a correção mais provável
```

### Por que Hooks importam

Hooks removem a **fricção de ter que lembrar de pedir** para a IA fazer algo. Você define uma vez o que deve acontecer em determinados momentos, e a IA passa a ser parte do fluxo — não uma ferramenta separada que você precisa abrir.

---

## 7. Agents — IA tomando decisões e agindo

Agents são LLMs que podem executar ferramentas, tomar decisões e completar tarefas multi-step de forma autônoma. Diferente de um chat, um agente não espera você guiar cada passo.

```
Chat comum:
  Você → pergunta → IA → resposta → Você → pergunta → IA → resposta

Agente:
  Você → objetivo → [IA decide o plano]
                         │
              ┌──────────┼──────────┐
              ▼          ▼          ▼
          [Lê arquivo] [Roda teste] [Edita código]
              │          │          │
              └──────────┼──────────┘
                         ▼
                  [IA avalia resultado]
                         │
                  [Continua ou ajusta]
                         │
                         ▼
                  [Entrega para você]
```

### Quando usar um agente vs. chat

| Chat | Agente |
|---|---|
| Tarefa bem definida e pequena | Tarefa aberta com múltiplos passos |
| Você sabe exatamente o que pedir | Você sabe o objetivo, não o caminho |
| Revisão rápida de um trecho | Implementação de uma feature do zero |
| Geração de um único artefato | Geração + teste + ajuste + commit |

### Configurando um agente no seu ambiente

Um agente bem configurado tem:
1. **Objetivo claro**: o que ele deve entregar
2. **Ferramentas disponíveis**: quais ações ele pode executar (ler arquivo, rodar comando, buscar na web)
3. **Contexto de domínio**: as Rules e Skills relevantes carregadas
4. **Critério de parada**: quando ele deve finalizar e reportar

---

## 8. Construindo contexto de qualidade

O contexto que você fornece é o principal fator que determina a qualidade da resposta. Lixo entra, lixo sai.

### Os três componentes do contexto

**1. Contexto de domínio** — "O que estamos construindo?"
- Stack tecnológica
- Regras de negócio relevantes
- Glossário do projeto (termos específicos)

**2. Contexto de tarefa** — "O que precisa ser feito agora?"
- Objetivo concreto
- Restrições específicas dessa tarefa
- Exemplos do que seria uma boa saída

**3. Contexto de estado** — "Onde estamos nesse momento?"
- Código relevante aberto / referenciado
- Erros ou outputs recentes
- Decisões já tomadas que a IA precisa respeitar

### Técnicas para construir contexto efetivo

```
✅ Seja específico sobre o objetivo final, não sobre os passos
   "Quero que a busca retorne resultados em menos de 200ms"
   — e não —
   "Otimize a query SQL da busca"

✅ Forneça exemplos de entrada e saída esperada
   "Input: lista de produtos com estoque | Output: lista filtrada com score de relevância"

✅ Indique o que NÃO fazer
   "Não use cache — esse dado precisa ser sempre fresco"

✅ Contextualize decisões anteriores
   "Já decidimos usar Supabase para auth. Não mude isso."

✅ Aponte os arquivos que importam para a tarefa
   "Veja o contrato em domain/repositories/product_repository.dart antes de mudar a implementação"
```

---

## 9. Criando workflows com IA

Um workflow com IA é uma sequência definida de passos onde humano e modelo colaboram de forma estruturada para entregar um resultado.

### Estrutura de um workflow

```
[Trigger]          → O que inicia o workflow
[Contexto montado] → O que a IA precisa saber
[Ação da IA]       → O que ela executa
[Revisão humana]   → O que você valida
[Próximo passo]    → Continua ou encerra
```

### Exemplo: workflow de implementação de feature

```
1. [Você define o objetivo]
   "Implementar filtro de produtos por categoria na tela de listagem"

2. [Agente monta o contexto]
   → Lê os arquivos relevantes da feature atual
   → Lê as rules do projeto (arquitetura, convenções)
   → Lê a skill de implementação de features

3. [Agente planeja]
   → Propõe lista de arquivos a criar/modificar
   → Você aprova ou ajusta o plano

4. [Agente implementa]
   → Cria/edita os arquivos
   → Roda os testes
   → Corrige o que quebrou

5. [Você revisa]
   → Valida as decisões tomadas
   → Testa manualmente os cenários principais

6. [Agente finaliza]
   → Gera mensagem de commit
   → Gera descrição do PR
```

### Workflows comuns e suas configurações ideais

| Workflow | Rules necessárias | Skills necessárias |
|---|---|---|
| Implementar feature | Stack, arquitetura, convenções | feature-implementation, testes |
| Code review | Padrões de qualidade do time | code-review |
| Debugging | Stack, estrutura de logs | debugging |
| Refatoração | Arquitetura alvo | refactoring, testes |
| Criação de PR | Padrão de commit | criar-pr |

---

## 10. Armadilhas comuns

### "O modelo não entendeu o que eu quis dizer"
→ Na maioria das vezes, o contexto estava incompleto. O modelo trabalha com o que recebeu.

### "A IA fez algo que eu não pedi"
→ Você pediu o objetivo sem as restrições. Adicione constraints explícitos no contexto.

### "Funciona no chat, mas quebra no meu projeto"
→ O ambiente sem Rules e Skills não tem o contexto do projeto. Configure o ambiente antes de trabalhar.

### "Cada vez que peço, ele faz diferente"
→ Sem contexto estruturado (Skills), o modelo interpreta a tarefa de forma diferente a cada vez. Codifique o que "bem feito" significa numa Skill.

### "Gastei mais tempo corrigindo do que teria implementado manualmente"
→ O contexto estava muito vago ou a tarefa era complexa demais para um único passo. Quebre em partes menores ou revise o contexto.

### Sobrecarregar a janela de contexto

```
❌ Colar arquivos inteiros que têm 2000 linhas
❌ Manter histórico de conversa longo com discussões irrelevantes
❌ Incluir logs completos quando apenas o erro importa

✅ Referenciar apenas os arquivos relevantes para a tarefa atual
✅ Começar uma nova conversa quando o contexto ficou sujo
✅ Filtrar: cole só o trecho do código que importa, não o arquivo inteiro
```

---

## 11. Checklist antes de trabalhar com um LLM

### Configuração do ambiente (uma vez por projeto)
- [ ] Rules do projeto criadas (stack, convenções, restrições)
- [ ] Skills para as tarefas mais frequentes definidas
- [ ] Hooks para eventos importantes configurados
- [ ] Agentes mapeados para os workflows principais

### Por tarefa
- [ ] O objetivo está claro e específico?
- [ ] Os arquivos relevantes estão no contexto?
- [ ] As restrições e o que NÃO fazer estão explicitados?
- [ ] Existe uma definição clara de "pronto" para essa tarefa?
- [ ] Há exemplos de input/output esperado (se aplicável)?

### Revisão do resultado
- [ ] O modelo tomou alguma decisão que não estava no contexto?
- [ ] O resultado segue as convenções do projeto?
- [ ] Há algo que o modelo "assumiu" sem você ter pedido?
- [ ] Você consegue explicar por que cada parte do código gerado existe?

Ver [ia/vibe-coding-entregas-com-ai.md](vibe-coding-entregas-com-ai.md) para o fluxo completo de entrega com IA usando esse ambiente configurado.

Ver [ia/orquestracao-de-multiplos-agentes.md](orquestracao-de-multiplos-agentes.md) para construir sistemas com múltiplos agentes especializados.
