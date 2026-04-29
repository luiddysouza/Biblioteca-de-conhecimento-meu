# Concorrência no Dart/Flutter: Future.wait, compute e Isolates

> Dart não tem threads no sentido clássico — tem isolates. Antes de paralelizar, a pergunta é uma só: a tarefa é I/O ou CPU?

---

## Sumário

1. [Visão Geral](#1-visão-geral)
2. [Future.wait](#2-futurewait)
3. [compute](#3-compute)
4. [Isolate.run](#4-isolaterun)
5. [compute vs Isolate.run](#5-compute-vs-isolaterun)
6. [Árvore de decisão](#6-árvore-de-decisão)
7. [Exemplos práticos consolidados](#7-exemplos-práticos-consolidados)
8. [Checklist](#8-checklist)

---

## 1. Visão Geral

| | `Future.wait` | `compute` | `Isolate.run` |
|---|---|---|---|
| **Modelo** | Concorrência assíncrona | Paralelismo (wrapper simples) | Paralelismo real |
| **Thread** | Mesma thread (event loop) | Thread separada | Thread separada |
| **Memória** | Compartilhada | Isolada (cópia) | Isolada (cópia) |
| **Uso ideal** | Múltiplas chamadas I/O em paralelo | CPU intensivo (caso simples) | CPU intensivo (caso completo) |
| **Aceita lambda** | ✅ | ❌ (top-level ou static) | ✅ |
| **Flutter only** | ❌ | ✅ | ❌ |

---

## 2. Future.wait

Dispara múltiplas operações **assíncronas ao mesmo tempo** e aguarda todas terminarem.
Continua rodando na mesma thread — usa o event loop do Dart para intercalar operações de I/O.

### Quando usar
- Múltiplas chamadas de API independentes
- Leitura de múltiplos arquivos
- Queries de banco de dados paralelas
- Qualquer operação I/O que não dependa uma da outra

### Sintaxe

```dart
final results = await Future.wait([
  buscarUsuario(),
  buscarProdutos(),
  buscarConfiguracoes(),
]);

final usuario      = results[0];
final produtos     = results[1];
final configuracao = results[2];
```

### Tratamento de erros

```dart
// eagerError: true (padrão) — cancela todas se uma falhar
// eagerError: false — aguarda todas, mesmo com falhas
final results = await Future.wait(
  [chamarApiA(), chamarApiB()],
  eagerError: false,
);
```

### Anti-padrão comum — awaits sequenciais independentes

```dart
// ❌ RUIM — executa um após o outro desnecessariamente
final resumo   = await _getResumoUsecase();
final grafico  = await _getGraficoUsecase();

// ✅ BOM — executa ambos ao mesmo tempo
final results = await Future.wait([
  _getResumoUsecase(),
  _getGraficoUsecase(),
]);
final resumo  = results[0];
final grafico = results[1];
```

---

## 3. compute

Wrapper do Flutter sobre `Isolate.run`. Executa uma função em um **isolate separado** (thread separada) para não bloquear a UI.

### Quando usar
- Parsing de JSON/XML grande
- Mapeamento de listas extensas
- Criptografia / hashing
- Qualquer operação CPU-intensiva que possa causar jank

### Assinatura

```dart
Future<R> compute<Q, R>(
  FutureOr<R> Function(Q) callback, // deve ser top-level ou static
  Q message,                        // único argumento permitido
)
```

### Sintaxe

```dart
// Função deve ser top-level (fora de qualquer classe)
List<Produto> _parsearProdutos(String jsonString) {
  final lista = jsonDecode(jsonString) as List;
  return lista.map((e) => Produto.fromJson(e)).toList();
}

// Uso
final produtos = await compute(_parsearProdutos, response.body);
```

### Passando múltiplos argumentos

Como `compute` aceita apenas 1 argumento, use `Record` (Dart 3+), `Map` ou uma classe:

```dart
// Com Record
List<String> _filtrar((List<String> lista, String filtro) args) {
  return args.$1.where((item) => item.contains(args.$2)).toList();
}

final resultado = await compute(_filtrar, (minhaLista, 'flutter'));

// Com Map
dynamic _processar(Map<String, dynamic> args) {
  final dados   = args['dados']  as List;
  final config  = args['config'] as String;
  // ...
}

await compute(_processar, {'dados': lista, 'config': 'valor'});
```

### Limitações

| Limitação | Explicação |
|---|---|
| Função deve ser top-level ou `static` | Closures capturam estado do isolate pai |
| Apenas 1 argumento | Use `Record`, `Map` ou classe como wrapper |
| Objetos são **copiados** | Sem memória compartilhada entre isolates |
| Overhead por chamada | Cria/destrói isolate a cada `compute()` |

---

## 4. Isolate.run

API nativa do Dart (2.19+). Mais flexível que `compute` — aceita lambdas diretamente.

### Quando usar
- Mesmos casos do `compute`, com mais flexibilidade
- Quando precisa de lambdas (closures simples sem captura de estado mutável)
- Projetos Dart puro (sem Flutter)

### Sintaxe básica

```dart
// Aceita lambda diretamente
final resultado = await Isolate.run(() => processamentoPesado(dados));

// Com argumentos capturados (cuidado: são copiados, não compartilhados)
final resultado = await Isolate.run(() => parsearJson(jsonString));
```

### Sintaxe completa com SendPort/ReceivePort

Use quando precisar de comunicação bidirecional ou isolate de longa duração:

```dart
Future<String> processarEmBackground(String dados) async {
  final receivePort = ReceivePort();

  await Isolate.spawn(_trabalhoHeavy, {
    'sendPort': receivePort.sendPort,
    'dados': dados,
  });

  return await receivePort.first as String;
}

// Deve ser top-level ou static
void _trabalhoHeavy(Map<String, dynamic> args) {
  final sendPort = args['sendPort'] as SendPort;
  final dados    = args['dados']    as String;

  final resultado = processarDados(dados);
  sendPort.send(resultado);
}
```

---

## 5. compute vs Isolate.run

```dart
// compute — Flutter only, função deve ser top-level/static
final result = await compute(_minhaFuncao, argumento);

// Isolate.run — Dart puro, aceita lambda (Dart 2.19+)
final result = await Isolate.run(() => _minhaFuncao(argumento));
```

`Isolate.run` é mais moderno e flexível. `compute` existe por compatibilidade histórica e ainda é amplamente usado em codebases Flutter mais antigas.

---

## 6. Árvore de decisão

```
A tarefa é I/O (rede, disco, banco de dados)?
  └─ SIM → Future.wait para múltiplas tarefas paralelas  ✅

A tarefa consome CPU (parse, encode, cálculo, mapeamento de lista grande)?
  └─ SIM → Pode travar a UI thread
       ├─ Caso simples, função já é top-level/static?
       │    └─ SIM → compute()  ✅
       └─ Caso mais complexo ou precisa de lambda?
            └─ Isolate.run()  ✅
```

---

## 7. Exemplos práticos consolidados

### Carregar tela com múltiplas APIs independentes
```dart
Future<void> init() async {
  final results = await Future.wait([
    _getGraficoUsecase(),
    _getInformativosUsecase(),
    _getResumoUsecase(),
  ]);

  grafico      = results[0];
  informativos = results[1];
  resumo       = results[2];
}
```

### Parsear JSON grande recebido da API
```dart
// top-level
List<Produto> _parsear(String json) =>
    (jsonDecode(json) as List).map((e) => Produto.fromJson(e)).toList();

// no datasource
final produtos = await compute(_parsear, response.body);
```

### Codificar arquivo/PDF em base64
```dart
// base64Encode em arquivos grandes bloqueia a UI thread
final base64String = await Isolate.run(() => base64Encode(pdfBytes));
```

### Mapeamento pesado de entidade complexa
```dart
// top-level
ClienteCompleto _mapearCliente(Map<String, dynamic> json) =>
    ClienteCompletoMapper.fromMap(json);

// no repositório
final cliente = await compute(_mapearCliente, response.data);
```

---

## 8. Checklist

- [ ] Chamadas de I/O independentes agrupadas em `Future.wait`
- [ ] Operações CPU-intensivas fora da UI thread (`compute` ou `Isolate.run`)
- [ ] Funções passadas para `compute` são top-level ou `static`
- [ ] Uso de `Isolate.run` quando lambdas são necessárias
- [ ] `Record` ou `Map` usado quando `compute` precisa de múltiplos argumentos
- [ ] Sem awaits sequenciais onde `Future.wait` caberia
