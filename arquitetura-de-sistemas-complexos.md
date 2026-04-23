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

## Referências e próximos passos

- Aprofundar em **Domain-Driven Design** para alinhar arquitetura com domínios de negócio
- Ver [visão_macro_para_engenheiro_ou_arquiteto_de_software.md](visão_macro_para_engenheiro_ou_arquiteto_de_software.md) para aplicação em Flutter
- Estudar **Designing Data-Intensive Applications** (Martin Kleppmann)
- Estudar **Software Architecture: The Hard Parts** (Ford, Richards)
