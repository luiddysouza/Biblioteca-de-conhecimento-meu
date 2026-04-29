# Biblioteca de Conhecimento — Instruções para o Copilot

## O que é este repositório

Este é um repositório de **base de conhecimento pessoal** — uma coleção de artigos em markdown sobre engenharia de software, arquitetura, produto e boas práticas. Não é um projeto de software. Não há código-fonte, testes, build ou CI/CD.

O objetivo de cada arquivo é explicar um tema de forma clara, estruturada e com exemplos concretos, para servir de referência futura.

---

## Perfil do autor

- **Framework principal:** Flutter com Dart
- **Experiência:** 5 anos em desenvolvimento de software
- **Nível assumido:** sênior — conceitos fundamentais de programação, Flutter, Dart, OOP, widgets, gerenciamento de estado e arquitetura mobile são conhecidos e vivenciados

### O que isso significa na prática

Não explique, não defina e não introduza conceitos que um dev Flutter sênior domina:
- Widgets básicos (`Column`, `Row`, `Text`, `ListView`, etc.)
- `StatefulWidget` vs `StatelessWidget`
- Sintaxe Dart (async/await, null safety, generics básicos)
- Conceitos gerais de OOP (herança, polimorfismo, encapsulamento)
- O que é um repositório, uma API, uma requisição HTTP
- Padrões amplamente conhecidos como BLoC, Provider, Clean Architecture

Quando um artigo mencionar um desses temas como contexto ou pré-requisito, **cite o termo diretamente** sem explicá-lo. Reserve o espaço do artigo para o que é novo, nuançado ou não óbvio.

### Quando há dúvida sobre o nível

Se um conceito estiver na fronteira entre básico e avançado, pergunte antes de detalhar:
> *"Esse conceito [X] merece explicação aqui ou posso assumir que já é conhecido?"*

---

## Idioma

- Todo o conteúdo é escrito em **português brasileiro (PT-BR)**
- Termos técnicos amplamente usados em inglês são mantidos em inglês sem tradução: BLoC, cache, hooks, deploy, branch, commit, pull request, scaffold, pipeline, tradeoff, feature, backend, frontend, API, CI/CD, LLM, prompt, agent, skill, workflow, etc.
- Nunca responda ou gere conteúdo em inglês, a menos que seja um trecho de código ou um termo técnico citado inline

---

## Estrutura padrão de um artigo

Todo artigo segue esta estrutura:

```
# Título do Artigo

> Frase ou parágrafo introdutório que contextualiza o tema. Pode ser uma citação, uma premissa ou uma provocação.

---

## Sumário

1. [Seção Um](#1-seção-um)
2. [Seção Dois](#2-seção-dois)
...

---

## 1. Seção Um

conteúdo...

---

## 2. Seção Dois

conteúdo...
```

### Regras da estrutura

- O título `#` descreve o tema do artigo — direto e específico
- A citação introdutória fica em blockquote `>` logo após o título
- O Sumário é sempre numerado com links âncora para cada seção
- As seções principais usam `##` e são numeradas (`## 1.`, `## 2.`, etc.)
- As subseções usam `###` e `####`
- Separadores `---` são usados entre seções principais

---

## Elementos recorrentes

Use estes elementos para tornar o conteúdo mais claro e escaneável:

### Tabelas comparativas
Para comparar abordagens, estados ou conceitos:
```markdown
| Conceito A | Conceito B |
|---|---|
| característica | característica |
```

### Blocos de código
Sempre com a linguagem tagueada, mesmo para pseudocódigo ou exemplos genéricos:
```markdown
```dart
// código aqui
```
```
Para exemplos sem linguagem específica, use blocos sem tag ou com `text`.

### ASCII flowcharts e diagramas
Para representar fluxos, arquiteturas ou sequências — usando `┌`, `│`, `└`, `→`, `↓`, `▼`:
```
[Entrada]
    │
    ▼
[Processamento]
    │
    ▼
[Saída]
```

### Padrão ✅ / ❌
Para mostrar práticas corretas e incorretas de forma visual:
```
✅ Faça assim
❌ Não faça assim
```

### Blockquotes para princípios-chave
Para destacar regras, insights ou resumos importantes:
```markdown
> Esta é a regra central desta seção.
```

### Checklists no final
A maioria dos artigos termina com um checklist prático:
```markdown
- [ ] Item para verificar
- [ ] Outro item
```

---

## Tom e estilo de escrita

- **Pedagógico, mas direto** — explica sem ser acadêmico ou formal
- **Orientado à ação** — usa verbos imperativos ("use", "evite", "prefira", "defina")
- **Exemplos sempre concretos** — nunca explicação teórica sem exemplo prático
- **Comparações antes/depois** — quando mostrar uma mudança, mostre o estado anterior e o resultado
- **Parágrafos curtos** — máximo 3-4 linhas por parágrafo
- Não use linguagem corporativa ou frases de efeito vazias

---

## O que nunca fazer

- ❌ Criar arquivos `.dart`, `.ts`, `.py`, `.yaml` ou qualquer arquivo de código
- ❌ Sugerir estruturas de projeto de software (pastas `lib/`, `src/`, `test/`)
- ❌ Mencionar comandos de build, test runner, linter ou CI/CD como se fossem necessários aqui
- ❌ Responder em inglês quando a pergunta for em português
- ❌ Criar arquivos de documentação sobre mudanças (changelogs, release notes) a menos que explicitamente pedido
- ❌ Adicionar frontmatter YAML nos artigos de conteúdo (`.md` que não são instruções/prompts)
- ❌ Usar formatação inconsistente com o estilo já estabelecido nos arquivos existentes
- ❌ Explicar conceitos básicos que o autor já domina (ver seção "Perfil do autor")
- ❌ Criar cascata de conhecimento auxiliar — ao sugerir aprofundamento, limite-se a **1 referência por seção**

---

## Cross-references

Quando um artigo mencionar um tema coberto por outro arquivo do repositório, adicione um link:
```markdown
Ver [arquitetura-de-sistemas-complexos.md](arquitetura-de-sistemas-complexos.md) para aprofundar.
```
