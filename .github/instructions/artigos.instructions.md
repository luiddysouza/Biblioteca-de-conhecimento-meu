---
description: "Use when: creating a new article, expanding an existing section, adding content to a document, writing about a new topic, or structuring knowledge base entries in this repository. Covers article template, structural rules, quality checklist, and style guidelines for markdown knowledge base articles."
applyTo: "**/*.md"
---

# Diretrizes para Artigos da Base de Conhecimento

## Template de novo artigo

Use esta estrutura completa ao criar um arquivo novo:

```markdown
# Título do Artigo

> Frase ou parágrafo introdutório que contextualiza o tema. Pode ser uma citação, uma premissa ou uma provocação.

---

## Sumário

1. [Nome da Seção Um](#1-nome-da-seção-um)
2. [Nome da Seção Dois](#2-nome-da-seção-dois)
3. [Nome da Seção Três](#3-nome-da-seção-três)

---

## 1. Nome da Seção Um

Conteúdo...

---

## 2. Nome da Seção Dois

Conteúdo...

---

## 3. Nome da Seção Três

Conteúdo...

---

## Checklist

- [ ] Item verificável
- [ ] Outro item
```

---

## Regras de estrutura

- **Título `#`**: direto e específico — descreve o tema, não o tipo de documento
- **Citação introdutória**: blockquote `>` imediatamente após o título; contextualiza, provoca ou estabelece uma premissa
- **Sumário**: sempre numerado; links âncora no formato `[Texto](#N-slug-da-seção)`
- **Seções principais `##`**: sempre numeradas sequencialmente (`## 1.`, `## 2.`, etc.)
- **Subseções `###` e `####`**: use para detalhar sem aumentar a contagem principal
- **Separadores `---`**: entre todas as seções principais
- **Sem frontmatter YAML**: artigos de conteúdo não usam `---` de frontmatter no topo
- **Sem datas de criação ou rodapé de metadados**: a menos que o artigo seja um registro temporal (ex: preparação de entrevista para uma vaga específica)

---

## Elementos obrigatórios por tipo de conteúdo

### Ao explicar um conceito

- Tabela comparativa com o oposto ou alternativa
- **Ao menos 2 exemplos**:
  - Um em linguagem comum ou lúdica (analogia do dia a dia, sem termos técnicos)
  - Um em Flutter/Dart (aplicação prática no contexto do autor)
- Blockquote com o princípio central da seção

### Ao mostrar boas e más práticas

- Padrão ✅ / ❌ com exemplos lado a lado
- Explicação do *porquê* cada prática é correta ou incorreta

### Ao descrever um fluxo ou processo

- ASCII diagram com `┌`, `│`, `└`, `→`, `↓`, `▼`
- Passos numerados com o que acontece em cada etapa

### Ao apresentar múltiplas opções

- Tabela com critérios de comparação
- Regra prática de quando usar cada opção

---

## Cross-references e conhecimento auxiliar

### Conhecimentos complementares

Quando uma seção mencionar um tema que já tem artigo no repositório, não explique o tema inline — referencie o arquivo:

```markdown
Ver [nome-do-arquivo.md](nome-do-arquivo.md) para aprofundar.
```

Se o tema não tiver artigo ainda mas for relevante, sinalize como candidato:

```markdown
> Tema candidato a artigo futuro: [nome sugerido]
```

### Conhecimento auxiliar para conceitos complexos

Se um conceito complexo se beneficia de um pré-requisito para ser entendido, **sugira** ao leitor — mas com limite:

- Máximo **1 sugestão de aprofundamento por seção**
- Nunca crie uma cadeia de dependências (artigo A → B → C → D)
- Se o pré-requisito for básico (algo que um dev Flutter sênior já sabe), apenas mencione o termo sem sugerir leitura

### Conhecimento básico

Antes de explicar um conceito, avalie:

| Situação | O que fazer |
|---|---|
| Conceito básico de Flutter/Dart/OOP | Apenas cite o termo — não explique |
| Conceito na fronteira básico/avançado | Pergunte ao autor antes de detalhar |
| Conceito avançado ou nuançado | Explique com exemplos |

Se houver dúvida: *"Esse conceito [X] merece explicação aqui ou posso assumir que já é conhecido?"*

---

## Como expandir uma seção existente

Ao adicionar conteúdo a um artigo que já existe:

1. **Leia o arquivo inteiro antes** — entenda o nível de profundidade e o estilo já usado
2. **Atualize o Sumário** — adicione a nova seção na lista numerada com link âncora
3. **Mantenha a numeração sequencial** — se o arquivo tem seções 1 a 5, a nova é `## 6.`
4. **Siga o tom do arquivo** — não mude o estilo de escrita ao adicionar uma seção
5. **Adicione separador `---`** antes e depois da nova seção
6. **Verifique cross-references** — se a nova seção mencionar um tema coberto por outro arquivo do repo, adicione o link

---

## Tom e estilo

- **Pedagógico, mas direto** — explica sem ser acadêmico ou formal
- **Orientado à ação** — verbos imperativos: "use", "evite", "prefira", "defina"
- **Parágrafos curtos** — máximo 3-4 linhas
- **Exemplos sempre concretos** — nunca teoria sem exemplo prático imediato
- **Comparações antes/depois** — mostre o estado anterior e o resultado quando demonstrar uma mudança
- Sem linguagem corporativa, sem frases de efeito vazias

---

## Idioma

- Todo conteúdo em **português brasileiro (PT-BR)**
- Termos técnicos amplamente usados em inglês ficam em inglês, sem tradução:
  - BLoC, cache, deploy, branch, commit, pull request, scaffold, pipeline, tradeoff, feature, backend, frontend, API, CI/CD, LLM, prompt, agent, skill, workflow, hook, widget, stream, event-driven, debounce, payload, endpoint, etc.
- Nunca traduzir um termo técnico que é universalmente usado em inglês na comunidade

---

## Checklist de qualidade antes de finalizar

- [ ] O artigo tem título `#`, citação introdutória `>` e Sumário numerado?
- [ ] Todas as seções principais usam `##` e são numeradas?
- [ ] O Sumário tem links âncora para todas as seções?
- [ ] Cada conceito tem pelo menos um exemplo concreto?
- [ ] Há pelo menos uma tabela comparativa?
- [ ] Há um checklist ou seção de resumo no final?
- [ ] O artigo está em PT-BR com termos técnicos em inglês mantidos?
- [ ] Não há frontmatter YAML no topo?
- [ ] Não há referências a build, testes, CI/CD ou estruturas de projeto de software?
- [ ] Se o tema foi coberto em outro artigo do repo, há cross-reference com link?
