# Orquestrando Múltiplos Agentes com Visão Estratégica

> Como construir sistemas onde múltiplas IAs trabalham juntas de forma coordenada para resolver problemas complexos.

---

## Sumário

1. [O que é um agente](#1-o-que-é-um-agente)
2. [Por que múltiplos agentes](#2-por-que-múltiplos-agentes)
3. [Padrões de orquestração](#3-padrões-de-orquestração)
4. [Componentes de um agente](#4-componentes-de-um-agente)
5. [Comunicação entre agentes](#5-comunicação-entre-agentes)
6. [Memória em sistemas multi-agente](#6-memória-em-sistemas-multi-agente)
7. [Visão estratégica na orquestração](#7-visão-estratégica-na-orquestração)
8. [Frameworks populares](#8-frameworks-populares)
9. [Armadilhas e como evitar](#9-armadilhas-e-como-evitar)
10. [Checklist antes de orquestrar](#10-checklist-antes-de-orquestrar)

---

## 1. O que é um agente

Um agente é um sistema de IA que pode **perceber, raciocinar e agir** de forma autônoma para atingir um objetivo.

```
[Entrada / Tarefa]
      │
      ▼
┌─────────────────────────────────────────┐
│              AGENTE                     │
│                                         │
│  Percebe   →   Raciocina   →   Age      │
│  (contexto)   (LLM / lógica)  (tools)  │
│                                         │
│  Memória   ←───────────────────────     │
└─────────────────────────────────────────┘
      │
      ▼
[Resultado / Ação]
```

### Diferença entre LLM, Chain e Agente

| Conceito | O que é | Autonomia |
|---|---|---|
| **LLM (modelo)** | Gera texto dado um prompt | Nenhuma |
| **Chain** | Sequência fixa de chamadas LLM | Nenhuma — fluxo pré-definido |
| **Agente** | Decide quais ferramentas usar e quando | Alta — loop dinâmico |

> Um agente **decide** o próximo passo com base no que observou. Uma chain segue um script.

---

## 2. Por que múltiplos agentes

Um único agente tem limitações:

```
Problemas de um agente único:
  ❌ Janela de contexto finita (não aguenta toda a informação)
  ❌ Não consegue trabalhar em paralelo
  ❌ Um único modelo generalista é menos preciso que especialistas
  ❌ Um erro contamina todo o raciocínio seguinte
  ❌ Difícil de testar e auditar cada etapa
```

Múltiplos agentes resolvem isso:

```
Sistema Multi-Agente:
  ✅ Cada agente tem contexto focado no seu domínio
  ✅ Tarefas independentes rodam em paralelo
  ✅ Agentes especializados têm melhor performance na sua área
  ✅ Falha isolada em um agente não derruba o sistema inteiro
  ✅ Mais fácil de testar, monitorar e substituir partes
```

### Analogia com times de trabalho

```
Projeto: Lançar um novo produto

Agente Orquestrador  =  Gerente de Projeto
  ├── Agente de Pesquisa     =  Analista de mercado
  ├── Agente de Redação      =  Copywriter
  ├── Agente de Design Brief =  Designer (define requisitos)
  └── Agente de Revisão      =  Editor / QA

Cada "pessoa" tem sua especialidade.
O gerente divide o trabalho, coordena e integra os resultados.
```

---

## 3. Padrões de orquestração

### 3.1 — Orchestrator-Worker (mais comum)

Um agente central coordena agentes especializados.

```
[Tarefa chegou]
      │
      ▼
[ORCHESTRATOR]
  Analisa a tarefa
  Decide quem chamar e em qual ordem
      │
  ┌───┼───┐
  ▼   ▼   ▼
[A] [B] [C]   ← Workers especializados
  │   │   │
  └───┼───┘
      ▼
[ORCHESTRATOR]
  Recebe resultados
  Combina e valida
      │
      ▼
[Resultado final]
```

**Quando usar**: tarefa complexa que pode ser decomposta em subtarefas bem definidas.

### 3.2 — Pipeline (sequencial)

Cada agente processa e passa o resultado para o próximo. A saída de um é a entrada do outro.

```
[Input]
  │
  ▼
[Agente 1: Coleta de dados]
  │  output: dados brutos
  ▼
[Agente 2: Análise]
  │  output: insights
  ▼
[Agente 3: Geração de relatório]
  │  output: documento final
  ▼
[Output]
```

**Quando usar**: fluxo com dependência clara entre etapas, sem paralelismo possível.

### 3.3 — Parallel Fan-Out / Fan-In

O orquestrador distribui trabalho em paralelo e consolida os resultados.

```
[Tarefa: analisar 5 documentos]
      │
      ▼
[Orquestrador]  ──── distribui ────►  [Agente A: doc 1]
                                  ──► [Agente B: doc 2]
                                  ──► [Agente C: doc 3]
                                  ──► [Agente D: doc 4]
                                  ──► [Agente E: doc 5]
                                          │
                                          ▼ (todos terminam)
[Orquestrador] ◄──── consolida ──── resultados de A,B,C,D,E
      │
      ▼
[Relatório consolidado]
```

**Quando usar**: tarefas independentes que podem ser paralelizadas (análise em lote, pesquisa multi-fonte).

### 3.4 — Supervisor com Revisão (Critic Pattern)

Um agente gera, outro critica e valida antes de seguir.

```
[Tarefa: Escrever código]
      │
      ▼
[Agente Gerador]  →  [código gerado]
                              │
                              ▼
                    [Agente Crítico / Revisor]
                      "Esse código está correto?"
                      "Tem bugs? Segue boas práticas?"
                              │
                    ┌─────────┴──────────┐
                    ▼                    ▼
              [APROVADO]          [REJEITADO]
              → entrega           → volta para Gerador
                                    com feedback
```

**Quando usar**: tarefas onde qualidade é crítica (código, documentação legal, decisões médicas).

### 3.5 — Hierarquia de Supervisores

Para sistemas muito complexos, supervisores supervisionam outros supervisores.

```
[Supervisor Estratégico]
   ├── [Supervisor de Pesquisa]
   │      ├── Agente Web Search
   │      ├── Agente de Banco de Dados
   │      └── Agente de Documentos
   │
   └── [Supervisor de Síntese]
          ├── Agente Escritor
          └── Agente Formatador
```

---

## 4. Componentes de um agente

Todo agente bem construído tem:

```
┌────────────────────────────────────────────────────┐
│                     AGENTE                         │
│                                                    │
│  🧠 LLM (cérebro)          — raciocina, decide     │
│                                                    │
│  🛠️  Tools (ferramentas)   — age no mundo          │
│     ├── search_web()                               │
│     ├── run_code()                                 │
│     ├── read_file()                                │
│     └── call_api()                                 │
│                                                    │
│  💾 Memory (memória)       — mantém contexto       │
│     ├── In-context (janela atual)                  │
│     ├── External (banco vetorial)                  │
│     └── Episódica (histórico de conversas)         │
│                                                    │
│  📋 Instructions (system prompt)                   │
│     Define: papel, objetivo, restrições, formato   │
└────────────────────────────────────────────────────┘
```

### Exemplo de system prompt bem definido

```
Você é um agente especializado em análise de código.

Seu papel:
  - Analisar código fornecido e identificar bugs, problemas de segurança
    e oportunidades de melhoria.

Você DEVE:
  - Sempre citar o arquivo e linha do problema identificado
  - Categorizar cada issue como: Bug | Segurança | Performance | Estilo
  - Priorizar issues críticas antes de cosméticas

Você NÃO DEVE:
  - Reescrever código completo sem ser solicitado
  - Inventar problemas que não existem
  - Fazer suposições sobre intenção sem evidência no código

Formato de saída: JSON estruturado conforme schema fornecido.
```

> System prompts vagos geram agentes imprevisíveis. Seja específico sobre papel, objetivo e restrições.

---

## 5. Comunicação entre agentes

### Modo Direto (síncrono)

```
Orquestrador chama Agente A e ESPERA a resposta antes de continuar.

Orquestrador ──► Agente A
             ◄── resposta
             ──► Agente B (só começa depois)
```

**Trade-off**: simples, mas lento se os agentes puderem rodar em paralelo.

### Modo Baseado em Mensagens (assíncrono)

```
Orquestrador publica tarefas numa fila.
Agentes consomem quando disponíveis e publicam resultados.

[Orquestrador] → [Fila de tarefas] → [Agente A]
                                   → [Agente B]
                                   → [Agente C]
                      ↑
[Orquestrador] ← [Fila de resultados] ← todos retornam aqui
```

**Trade-off**: mais complexo, mas permite paralelismo real e escalabilidade.

### Shared State (estado compartilhado)

Agentes leem e escrevem num estado global.

```
                  [Estado Global]
                  {
                    task: "...",
                    research: [...],    ← escrito pelo Agente de Pesquisa
                    draft: "...",       ← escrito pelo Agente Escritor
                    review_notes: [...] ← escrito pelo Agente Revisor
                  }

[Agente A] ──lê e escreve──► [Estado Global] ◄── lê e escreve── [Agente B]
```

**Trade-off**: fácil de inspecionar, mas pode ter conflitos se dois agentes modificam a mesma parte.

---

## 6. Memória em sistemas multi-agente

| Tipo de memória | Escopo | Onde fica | Exemplo |
|---|---|---|---|
| **In-context** | Conversação atual | Janela do LLM | Mensagens da conversa |
| **External (vetorial)** | Persistente | Vector DB | Documentos, FAQs, histórico |
| **Episódica** | Por sessão | Cache / DB | O que foi feito nessa tarefa |
| **Procedural** | Permanente | System prompt | Como o agente deve se comportar |

### Problema de contexto longo

```
Tarefa longa → contexto cresce → LLM "esquece" o início → erros acumulam

Solução 1: Summarization — resumir periodicamente e descartar histórico antigo
Solução 2: External memory — salvar fatos importantes fora da janela de contexto
Solução 3: Handoff documents — ao transferir para outro agente, passar um briefing resumido
```

---

## 7. Visão estratégica na orquestração

A visão estratégica é o que separa um orquestrador de um simples dispatcher.

### O orquestrador precisa saber

```
1. OBJETIVO FINAL
   "O que precisa estar verdadeiro ao final desta tarefa?"

2. ESTADO ATUAL
   "O que já sabemos? O que já foi feito? O que falta?"

3. RECURSOS DISPONÍVEIS
   "Quais agentes/tools tenho? Quais são os custos (tempo, tokens)?"

4. PLANO E ORDEM
   "Quais subtarefas dependem de outras? O que pode rodar em paralelo?"

5. GESTÃO DE FALHAS
   "Se o Agente X falhar, qual é o plano B?"

6. CRITÉRIO DE SUCESSO
   "Como vou saber que terminei? Quando é 'bom o suficiente'?"
```

### O loop ReAct (Reason + Act)

É o padrão de raciocínio mais comum em agentes:

```
[Observação inicial]
       │
       ▼
  [RACIOCINAR]
  "O que eu sei? O que preciso descobrir? Qual é o próximo passo?"
       │
       ▼
  [AGIR]
  Chama uma tool, chama um agente, escreve algo
       │
       ▼
  [OBSERVAR resultado]
  "O que a ação retornou? Isso me ajuda a avançar?"
       │
       └──────────► [RACIOCINAR novamente]
                    (loop até atingir o objetivo)
```

### Decomposição estratégica de tarefas complexas

```
Tarefa: "Crie um relatório de análise competitiva sobre o mercado de ERP"

❌ Abordagem ingênua:
  → passa a tarefa inteira para um único agente
  → o agente alucina porque não tem os dados necessários

✅ Abordagem estratégica do Orquestrador:

  1. [Planejamento]
     "Quais empresas devo analisar? Quais dimensões comparar?"
     → gera estrutura do relatório

  2. [Coleta de dados] (paralelo)
     Agente A → pesquisa SAP
     Agente B → pesquisa Oracle
     Agente C → pesquisa TOTVS

  3. [Análise] (sequencial, depende dos dados)
     Agente Analista → compara os dados coletados

  4. [Redação]
     Agente Escritor → escreve o relatório com base na análise

  5. [Revisão]
     Agente Revisor → verifica consistência e qualidade

  6. [Entrega]
     Orquestrador → consolida e formata saída final
```

### Guardrails estratégicos

O orquestrador deve definir limites claros para evitar comportamentos indesejados:

```python
# Exemplo de guardrails num orquestrador
MAX_ITERATIONS = 10        # Evita loops infinitos
MAX_COST_USD = 5.0         # Controle de custo
TIMEOUT_SECONDS = 300      # Timeout total da tarefa
ALLOWED_TOOLS = [          # Lista de tools permitidas para cada agente
    "search_web",
    "read_file",
    "write_file",
]
FORBIDDEN_ACTIONS = [      # O que jamais pode ser feito
    "delete_database",
    "send_email_external",
    "execute_shell",
]
```

---

## 8. Frameworks populares

| Framework | Linguagem | Ponto forte | Quando usar |
|---|---|---|---|
| **LangGraph** | Python | Grafos de estado, controle fino do fluxo | Orquestração complexa com ciclos e revisões |
| **AutoGen** | Python | Multi-agente conversacional | Agentes que "debatem" entre si para resolver problemas |
| **CrewAI** | Python | Abstração de alto nível (roles, tasks, crew) | Prototipagem rápida de times de agentes |
| **Semantic Kernel** | C#, Python | Integração enterprise, plugins | Sistemas .NET, ambientes corporativos |
| **Agno** | Python | Leve, flexível, agnóstico de modelo | Sistemas custom sem muita abstração |

### Exemplo conceitual com CrewAI

```python
from crewai import Agent, Task, Crew

pesquisador = Agent(
    role="Pesquisador de mercado",
    goal="Coletar dados sobre concorrentes no mercado de ERP",
    backstory="Especialista em inteligência competitiva com 10 anos de experiência",
    tools=[search_web, read_documents],
)

analista = Agent(
    role="Analista estratégico",
    goal="Identificar oportunidades e ameaças com base nos dados coletados",
    backstory="Analista focado em transformar dados em insights acionáveis",
)

tarefa_pesquisa = Task(
    description="Pesquise os 3 principais concorrentes: SAP, Oracle, TOTVS",
    agent=pesquisador,
    expected_output="Relatório com dados de cada concorrente",
)

tarefa_analise = Task(
    description="Analise os dados e identifique gaps de mercado",
    agent=analista,
    context=[tarefa_pesquisa],  # Depende da pesquisa
    expected_output="Lista de oportunidades estratégicas",
)

equipe = Crew(
    agents=[pesquisador, analista],
    tasks=[tarefa_pesquisa, tarefa_analise],
)

resultado = equipe.kickoff()
```

---

## 9. Armadilhas e como evitar

### Delegação sem verificação
O orquestrador delega e assume que o resultado está correto.
> Remédio: use o Critic Pattern para validar saídas críticas antes de usá-las como input para o próximo agente.

### Context pollution (contaminação de contexto)
Um agente passa contexto desnecessário ou incorreto para o próximo, que começa a raciocinar errado.
> Remédio: use "handoff documents" estruturados — defina exatamente o que deve ser passado entre agentes.

### Loops infinitos
Agente A chama Agente B, que chama Agente A, que chama Agente B...
> Remédio: defina `MAX_ITERATIONS` e detecte ciclos no grafo de chamadas.

### Over-engineering de agentes
Criar 10 agentes para um problema que um único agente resolve bem.
> Regra: use um agente se o problema couber em uma janela de contexto. Só divida quando houver razão clara.

### Falta de observabilidade
O sistema multi-agente é uma "caixa preta". Algo falha e você não sabe onde.
> Remédio: log estruturado em cada etapa, com: agente chamado, input recebido, output gerado, tempo de execução, custo de tokens.

```
Log estruturado de uma execução:
{
  "trace_id": "abc-123",
  "agent": "pesquisador",
  "step": 3,
  "input": "Pesquise SAP ERP pricing",
  "tool_called": "search_web",
  "tool_result": "...",
  "output": "SAP cobra entre $100-300/user/month...",
  "tokens_used": 1240,
  "duration_ms": 2300,
  "status": "success"
}
```

### Alucinação em cascata
Um agente alucina um dado. O próximo usa esse dado como verdade. O resultado final está completamente errado.
> Remédio: agentes de validação antes de usar dados críticos, e grounding com fontes verificáveis.

---

## 10. Checklist antes de orquestrar

- [ ] O problema realmente precisa de múltiplos agentes? (não é over-engineering?)
- [ ] Os papéis e responsabilidades de cada agente estão claramente definidos?
- [ ] As dependências entre tarefas estão mapeadas? (o que pode rodar em paralelo?)
- [ ] Existe um mecanismo de revisão para saídas críticas? (Critic Pattern)
- [ ] Os guardrails estão definidos? (max iterações, custo, timeout, tools permitidas)
- [ ] Existe observabilidade? (logs estruturados com trace_id)
- [ ] A estratégia de memória está definida? (o que persiste, o que descarta)
- [ ] Existe um plano de fallback se um agente falhar?
- [ ] O critério de sucesso está claro? (como o orquestrador sabe que terminou?)

---

## Conexão com os outros conceitos

```
Orquestração de Agentes na prática:

  Contexto de Negócio  →  define QUAL problema o sistema de agentes resolve
                          e QUAIS são os critérios de sucesso

  Arquitetura          →  define COMO os agentes se comunicam
                          (API REST, filas, estado compartilhado)
                          e os NFRs do sistema (latência, custo, confiabilidade)
```

- Ver [contexto-de-negocio-para-produto.md](contexto-de-negocio-para-produto.md) para definir o problema antes de construir o sistema
- Ver [arquitetura-de-sistemas-complexos.md](arquitetura-de-sistemas-complexos.md) para decisões de infraestrutura do sistema de agentes
