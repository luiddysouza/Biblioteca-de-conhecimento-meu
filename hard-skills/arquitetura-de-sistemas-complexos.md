# Definindo Arquitetura de Sistemas Complexos

> Como sair do caos da complexidade acidental e tomar decisões de arquitetura com clareza e intenção.

---

## Sumário

1. [O que torna um sistema "complexo"](#1-o-que-torna-um-sistema-complexo)
2. [O processo de definição de arquitetura](#2-o-processo-de-definição-de-arquitetura)
3. [Requisitos não-funcionais (NFRs)](#3-requisitos-não-funcionais-nfrs)
4. [Principais decisões arquiteturais](#4-principais-decisões-arquiteturais)
5. [Padrões para sistemas distribuídos](#5-padrões-para-sistemas-distribuídos)
6. [ADR — Architecture Decision Record](#6-adr--architecture-decision-record)
7. [Lei de Conway](#7-lei-de-conway)
8. [Armadilhas comuns](#8-armadilhas-comuns)
9. [Checklist antes de decidir](#9-checklist-antes-de-decidir)
10. [Comunicando decisões técnicas com clareza](#10-comunicando-decisões-técnicas-com-clareza)

---

## 1. O que torna um sistema "complexo"

Complexidade não é só "muitas linhas de código". Existem dois tipos:

| Tipo | O que é | Exemplo |
|---|---|---|
| **Complexidade essencial** | Dificuldade inerente do problema de negócio | Calcular rotas em tempo real, regras de tributação |
| **Complexidade acidental** | Dificuldade que nós mesmos criamos | Tecnologia errada, abstração prematura, acoplamento desnecessário |

**Seu objetivo como arquiteto é: eliminar complexidade acidental, gerenciar a essencial.**

### Sinais de que o sistema ficou complexo demais para o time entender

- Ninguém sabe o que acontece se um serviço cair
- Cada feature nova quebra algo "sem relação"
- O onboarding de um dev novo leva semanas
- Nenhum CI passa sem ajustes manuais
- Há medo de mexer em certas partes do código

---

## 2. O processo de definição de arquitetura

Arquitetura não começa com tecnologia. Começa com perguntas.

### Passo 1 — Entender o contexto do negócio

Antes de qualquer diagrama:
- Qual é o problema real que estamos resolvendo?
- Quem são os usuários e como eles usam o sistema?
- Quais são os limites aceitáveis de falha?
- Qual é o volume esperado (usuários, transações, dados)?
- Qual é o orçamento de time e infraestrutura?

### Passo 2 — Mapear os atores e fluxos principais

Use o **Modelo C4** para ir do macro ao micro:

```
[Nível 1 - System Context]
  Quem usa o sistema? Quais sistemas externos ele se conecta?
        ↓
[Nível 2 - Container]
  Quais são as peças grandes? (APIs, bancos, filas, apps)
        ↓
[Nível 3 - Component]
  Dentro de cada container, quais são os módulos?
        ↓
[Nível 4 - Code]
  Classes, funções — só entre aqui quando necessário
```

> Nunca comece do Nível 4. O erro mais comum é entrar nos detalhes antes de entender o sistema como um todo.

### Passo 3 — Identificar os Quality Attributes (Atributos de Qualidade)

Estes são os critérios que vão guiar suas decisões:

- **Disponibilidade**: o sistema pode ficar fora do ar? Por quanto tempo?
- **Escalabilidade**: precisa crescer como? Vertical ou horizontal?
- **Latência**: qual é o tempo de resposta aceitável?
- **Consistência**: dados precisam ser sempre corretos, ou eventual é aceitável?
- **Segurança**: quais são os requisitos regulatórios e de compliance?
- **Manutenibilidade**: quão fácil é mudar, testar e fazer deploy?

### Passo 4 — Identificar os trade-offs

Toda decisão de arquitetura é um trade-off. Não existe escolha perfeita.

```
Consistência forte   ←→   Alta disponibilidade     (Teorema CAP)
Acoplamento baixo    ←→   Complexidade operacional  (Microserviços)
Flexibilidade        ←→   Overhead de abstração     (Over-engineering)
Entrega rápida       ←→   Dívida técnica            (MVP vs qualidade)
```

---

## 3. Requisitos não-funcionais (NFRs)

NFRs são o que define a QUALIDADE do sistema. Geralmente não aparecem nas histórias de usuário, mas definem toda a arquitetura.

```
Exemplo de NFRs para um sistema de pagamentos:

Disponibilidade:    99.99% uptime (máx. ~52 min/ano fora do ar)
Latência:           < 200ms para 95% das transações
Throughput:         10.000 transações por segundo no pico
Segurança:          PCI-DSS compliance obrigatório
RPO:                0 (nenhuma transação pode ser perdida)
RTO:                < 5 minutos para recuperação de falha
```

| Sigla | Conceito | O que mede |
|---|---|---|
| **RPO** | Recovery Point Objective | Quantos dados posso perder em uma falha? |
| **RTO** | Recovery Time Objective | Em quanto tempo devo voltar ao ar? |
| **SLA** | Service Level Agreement | Compromisso formal de disponibilidade |
| **SLO** | Service Level Objective | Meta interna de qualidade de serviço |

---

## 4. Principais decisões arquiteturais

### Monólito vs Microserviços

Não é uma questão de "qual é melhor". É uma questão de **onde você está na jornada**.

```
Monólito                        Microserviços
────────────────────────        ────────────────────────────
✅ Simples de desenvolver       ✅ Escala independente por domínio
✅ Fácil de debugar             ✅ Deploy independente por serviço
✅ Transações ACID naturais     ✅ Tecnologias distintas por serviço
✅ Ótimo para times pequenos    ✅ Isolamento de falha

❌ Escalar = escalar tudo       ❌ Latência de rede entre serviços
❌ Deploy arriscado (tudo junto)❌ Consistência distribuída complexa
❌ Acoplamento cresce com tempo ❌ Overhead operacional alto
```

> **Regra prática**: comece com monólito modular. Extraia serviços quando um domínio específico exigir escala ou deploy independente — não antes.

### Event-Driven Architecture (EDA)

Componentes se comunicam através de **eventos** em vez de chamadas diretas.

```
Sem EDA (acoplamento direto):
  Pedido → chama → Estoque → chama → Notificação → chama → Faturamento

Com EDA (desacoplado):
  Pedido → publica [PedidoCriado]
                ↓ consome
           Estoque → publica [EstoqueReservado]
                         ↓ consome
                    Notificação (independente)
                    Faturamento (independente)
```

**Quando usar EDA:**
- Operações que podem acontecer em paralelo e de forma independente
- Integrações com sistemas externos
- Auditoria e rastreabilidade (Event Sourcing)
- Quando o produtor não precisa saber quem consome

**Cuidados:**
- Consistência eventual: os sistemas ficam "sincronizados depois", não na hora
- Debugging distribuído é mais difícil (use correlation IDs)
- Precisa de uma estratégia para eventos que falham (dead-letter queue)

### CQRS — Command Query Responsibility Segregation

Separa o modelo de **escrita** (Command) do modelo de **leitura** (Query).

```
                    ┌─────────────────────────┐
   Comando          │                         │   Query
  (Escrita)  ──────►│   Command Handler       │◄──────── (Leitura)
                    │   + Write Database      │   Read Database
                    │                         │   (otimizado para leitura)
                    │   Evento publicado  ────►│   Projection atualiza
                    └─────────────────────────┘
```

**Quando faz sentido:**
- Alta diferença entre volume de leituras e escritas
- Queries complexas que precisam de modelos desnormalizados
- Sistemas que precisam de Event Sourcing

---

## 5. Padrões para sistemas distribuídos

### Circuit Breaker

Evita que uma falha em um serviço cascateie para o resto do sistema.

```
Estado CLOSED (normal):
  A → chama → B → responde ✅

Estado OPEN (B falhou):
  A → tenta chamar B → Circuit Breaker retorna erro imediato ❌
  (sem esperar timeout, sem sobrecarregar B)

Estado HALF-OPEN (testando recuperação):
  A → deixa passar uma chamada de teste → se B responder ✅ → volta para CLOSED
```

### Saga Pattern

Gerencia transações distribuídas que precisam garantir consistência entre múltiplos serviços.

```
Saga de Pedido (Coreografia):

  PedidoCriado
      ↓
  [Estoque] reserva → EstoqueReservado
                           ↓
                      [Pagamento] processa → PagamentoAprovado
                                                  ↓
                                             [Entrega] agenda ✅

Se falhar no Pagamento:
  PagamentoRecusado
      ↓
  [Estoque] libera reserva (compensação) ↩
      ↓
  [Pedido] cancelado ↩
```

### API Gateway

Ponto único de entrada que centraliza autenticação, rate limiting, roteamento e logging.

```
Cliente
   │
   ▼
[API Gateway]
   ├── Auth/JWT validation
   ├── Rate limiting
   ├── Request routing
   └── Logging / Tracing
         │
    ┌────┼────┐
    ▼    ▼    ▼
 Svc A  Svc B  Svc C
```

---

## 6. ADR — Architecture Decision Record

**ADR é um documento curto que registra UMA decisão de arquitetura importante.**

Por que usar:
- Evita tomar a mesma decisão errada duas vezes
- Novos membros do time entendem o "porquê" das escolhas
- Facilita revisitar decisões quando o contexto muda

### Template de ADR

```markdown
# ADR-001: Uso de PostgreSQL como banco principal

## Status
Aceito

## Contexto
Precisamos de um banco relacional que suporte transações ACID e tenha
suporte robusto a JSON para dados semiestruturados. O time tem experiência
prévia com PostgreSQL.

## Decisão
Adotaremos PostgreSQL como banco de dados principal para todos os serviços
que precisam de persistência relacional.

## Consequências
✅ ACID nativo
✅ Suporte a JSONB para dados flexíveis
✅ Ecossistema maduro (pgvector, TimescaleDB, etc.)
❌ Escala horizontal mais complexa que bancos NoSQL
❌ Requer gestão de schema migrations
```

---

## 7. Lei de Conway

> "Qualquer organização que projeta um sistema irá produzir um design cuja estrutura é uma cópia da estrutura de comunicação da organização."
> — Melvin Conway, 1967

Em outras palavras: **a arquitetura do seu software vai refletir os silos da sua empresa**.

```
Time A (Auth)   Time B (Pedidos)   Time C (Estoque)
     │                │                   │
     ▼                ▼                   ▼
[Serviço Auth]  [Serviço Pedidos]  [Serviço Estoque]
      └───────────────┴───────────────────┘
              Arquitetura resultante
```

**Implicação prática**: se você quer mudar a arquitetura, muitas vezes precisa mudar a estrutura do time (Inverse Conway Maneuver).

---

## 8. Armadilhas comuns

### Over-engineering prematuro
Construir microserviços, Kubernetes e event sourcing para um sistema que tem 10 usuários.
> Regra: adicione complexidade quando o problema real aparecer, não antes.

### Big Ball of Mud
Sistema que cresceu sem arquitetura intencional. Tudo acoplado com tudo.
> Solução: modularização incremental com strangler fig pattern.

### Distributed Monolith
Você "separou em microserviços" mas eles ainda deployam juntos e compartilham banco.
> É o pior dos dois mundos: complexidade de microserviços sem os benefícios.

### Ignorar observabilidade desde o início
Não planejar logs estruturados, métricas, distributed tracing e alertas.
> Você só vai sentir a falta quando o sistema estiver em produção quebrando e você não souber por quê.

---

## 9. Checklist antes de decidir

Antes de definir ou validar uma arquitetura, responda:

- [ ] Quais são os atributos de qualidade mais críticos? (disponibilidade, latência, consistência...)
- [ ] Qual é o volume esperado de dados e usuários nos próximos 12 meses?
- [ ] O time tem capacidade operacional para manter essa complexidade?
- [ ] Os trade-offs foram explicitados e aceitos pelos stakeholders?
- [ ] Existe uma estratégia de observabilidade (logs, métricas, tracing)?
- [ ] Existe uma estratégia de falha e recuperação (circuit breaker, retry, fallback)?
- [ ] As decisões importantes foram documentadas em ADRs?
- [ ] A arquitetura reflete os limites naturais dos domínios de negócio (DDD)?

---

## 10. Comunicando decisões técnicas com clareza

Tomar uma boa decisão de arquitetura e não conseguir explicá-la tem o mesmo efeito prático de não ter tomado uma boa decisão. A decisão só ganha vida quando o time a entende e a segue.

### O erro mais comum: explicar a solução sem explicar o problema

```
❌ "Vamos usar CQRS aqui."

✅ "Temos 50x mais leituras do que escritas nesse domínio, e as queries estão
    ficando lentas porque o modelo de escrita não é adequado para leitura.
    CQRS resolve isso separando os dois modelos — a escrita continua simples
    e a leitura usa uma projeção otimizada. O custo é manter a sincronia
    entre os dois lados via eventos."
```

Quem ouve a segunda versão entende o problema, a decisão e o tradeoff — e consegue aplicar o mesmo raciocínio em situações futuras.

---

### Estrutura para explicar qualquer decisão técnica

```
1. Contexto
   "Qual era a situação e por que ela exigia uma decisão?"

2. Opções consideradas
   "Quais alternativas foram avaliadas?"

3. Decisão
   "O que foi escolhido e por quê venceu as alternativas?"

4. Tradeoffs reais
   "O que ganhamos? O que abrimos mão? O que ainda é incerto?"

5. Consequências
   "O que muda no código, no processo e no time a partir dessa decisão?"
```

> Não existe decisão de arquitetura sem tradeoff. Se você não consegue apontar o que foi aberto mão, a decisão ainda não foi bem analisada.

---

### Adaptando a explicação para cada audiência

A mesma decisão precisa ser explicada de formas diferentes dependendo de quem está ouvindo.

| Audiência | O que importa | Como explicar |
|---|---|---|
| **Time técnico** | Precisão técnica, impacto no código, padrões afetados | Use terminologia correta, mostre o diagrama, discuta edge cases |
| **Tech Lead / Arquiteto** | Tradeoffs, alternativas descartadas, riscos a longo prazo | Foque nos porquês e nas consequências não óbvias |
| **PM / Product** | Impacto no prazo, custo, risco de mudança futura | Traduza para impacto no produto e na velocidade do time |
| **Stakeholder de negócio** | Risco, custo, prazo | Nunca use jargão técnico — use analogias e consequências concretas |

### Exemplo da mesma decisão para audiências diferentes

**Decisão:** migrar de banco de dados relacional para uma solução com leitura via cache distribuído.

```
Para o time técnico:
  "Vamos adicionar Redis como cache de leitura à frente do PostgreSQL.
   O padrão é Cache-Aside: a aplicação verifica o cache antes de bater no banco.
   TTL de 5 minutos. Precisamos garantir invalidação quando o dado for atualizado."

Para o PM:
  "A tela de listagem vai ficar ~10x mais rápida para o usuário.
   O trabalho leva ~3 dias. O único risco é que, em casos raros, o usuário
   pode ver um dado com até 5 minutos de atraso — aceitável dado o contexto."

Para o stakeholder de negócio:
  "Estamos com lentidão que pode afetar a conversão. A correção leva 3 dias
   e reduz o custo de infraestrutura em ~30% no pico de acesso."
```

---

### Tradeoffs reais mais comuns — e como verbalizá-los

| Situação | Tradeoff | Como verbalizar |
|---|---|---|
| Monólito vs. Microserviços | Velocidade agora vs. flexibilidade no futuro | "Ganhamos 3 sprints agora. Se precisarmos escalar esse domínio independentemente em 6 meses, vamos gastar ~2 semanas de extração." |
| Consistência eventual | Disponibilidade vs. precisão imediata dos dados | "O dado pode levar até X segundos para propagar. Para esse caso de uso, é aceitável. Não seria aceitável em transações financeiras." |
| Cache | Performance vs. frescor do dado | "Respostas até 10x mais rápidas. O tradeoff é que o dado pode ter até 5 minutos de atraso. Implementamos invalidação ativa nos updates críticos." |
| Abstração extra | Flexibilidade vs. overhead de manutenção | "Essa camada nos permite trocar o provider de pagamento sem tocar no domínio. O custo é mais um nível de indireção que o dev precisa entender no onboarding." |
| Dívida técnica consciente | Velocidade de entrega agora vs. custo de manutenção depois | "Fazemos o simples agora para não travar o lançamento. Registramos a dívida e revisitamos no próximo ciclo quando tivermos dados de uso real." |

---

### Sinais de que uma decisão foi mal comunicada

```
❌ O time implementa de formas diferentes em partes distintas do sistema
❌ A decisão precisa ser reexplicada toda vez que alguém novo entra
❌ Membros do time não sabem por que aquela regra existe
❌ A decisão é revertida por alguém que "não sabia" que ela havia sido tomada
❌ Stakeholders questionam a decisão meses depois porque não lembravam do contexto
```

A solução para todos esses casos é o mesmo: **documente no ADR e explique no PR**. Decisão sem registro é decisão perdida.

---

### Como usar ADR + PR Description como par de comunicação

```
ADR:
  Registro permanente, contexto completo, para referência futura.
  "Por que essa decisão foi tomada?"

PR Description:
  Comunicação imediata, para quem vai revisar e mergear agora.
  "O que mudou, como testar e o que precisa de atenção."

Os dois se complementam:
  - PR aponta para o ADR quando a decisão é significativa
  - ADR documenta o que PR description não tem espaço para aprofundar
```

### Template de PR Description para decisões arquiteturais

```markdown
## O que esse PR faz
Implementa [comportamento]. Resolve [problema concreto].

## Decisão técnica relevante
[Explique a principal decisão tomada, em 2-3 linhas]
Ver ADR-00X para o contexto completo.

## Tradeoffs assumidos
- ✅ [O que ganhamos]
- ⚠️ [O que abrimos mão ou o que ainda é incerto]

## Como testar
1. [Passo 1]
2. [Passo 2]

## O que ficou de fora (dívida consciente)
- [Item que foi deixado para depois, com justificativa]
```

---

## Referências e próximos passos

- Aprofundar em **Domain-Driven Design** para alinhar arquitetura com domínios de negócio
- Ver [visão_macro_para_engenheiro_ou_arquiteto_de_software.md](visão_macro_para_engenheiro_ou_arquiteto_de_software.md) para aplicação em Flutter
- Estudar **Designing Data-Intensive Applications** (Martin Kleppmann)
- Estudar **Software Architecture: The Hard Parts** (Ford, Richards)
