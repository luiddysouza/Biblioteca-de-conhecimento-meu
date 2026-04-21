# Visão Macro de um Projeto Flutter

> Como parar de ler código linha por linha e começar a enxergar o sistema como um todo.

---

## Sumário

1. [Arquitetura](#1-arquitetura)
2. [Cobertura de Testes](#2-cobertura-de-testes)
3. [Complexidade Ciclomática](#3-complexidade-ciclomática)
4. [Testes de Mutação](#4-testes-de-mutação)
5. [Fluxo Recomendado](#5-fluxo-recomendado)

---

## 1. Arquitetura

Antes de abrir qualquer arquivo `.dart`, você precisa entender as **fronteiras do sistema**: quem depende de quem, quais são as camadas e onde estão os acoplamentos implícitos.

### Modelo C4 aplicado ao Flutter

O modelo C4 organiza a arquitetura em 4 níveis de zoom:

```
[Nível 1 - Context]   O app Flutter no ecossistema (APIs, Firebase, etc.)
      ↓
[Nível 2 - Container] App client, BFF, serviços externos
      ↓
[Nível 3 - Component] Features, camadas (data/domain/presentation)
      ↓
[Nível 4 - Code]      Classes, widgets, blocs — o que você já conhece
```

Comece sempre do Nível 1. Entenda quais sistemas externos o app consome antes de entrar em qualquer feature.

---

### Estrutura de um projeto Flutter bem arquitetado

Um projeto Flutter saudável costuma seguir uma separação por **features + camadas**:

```
lib/
├── core/
│   ├── network/          # Dio, interceptors, http client
│   ├── storage/          # SharedPreferences, Hive, SecureStorage
│   ├── error/            # Failures, exceptions, Either<L,R>
│   └── di/               # Injeção de dependência (GetIt, etc.)
│
├── features/
│   ├── auth/
│   │   ├── data/
│   │   │   ├── datasources/    # AuthRemoteDataSource, AuthLocalDataSource
│   │   │   ├── models/         # UserModel (fromJson/toJson)
│   │   │   └── repositories/   # AuthRepositoryImpl
│   │   ├── domain/
│   │   │   ├── entities/       # User (objeto puro, sem lógica de serialização)
│   │   │   ├── repositories/   # AuthRepository (contrato/interface)
│   │   │   └── usecases/       # LoginUseCase, LogoutUseCase
│   │   └── presentation/
│   │       ├── bloc/           # AuthBloc, AuthEvent, AuthState
│   │       ├── pages/          # LoginPage, RegisterPage
│   │       └── widgets/        # LoginForm, PasswordField
│   │
│   └── checkout/
│       └── ... (mesma estrutura)
│
└── main.dart
```

---

### Sinais de arquitetura degradada

| Sinal | O que significa | Como detectar |
|---|---|---|
| Widget com 500+ linhas | Lógica de negócio misturada com UI | `find lib -name "*.dart" \| xargs wc -l \| sort -rn` |
| `import` cruzando features diretamente | Acoplamento entre features | `grep -r "import.*features/auth" lib/features/checkout/` |
| `BuildContext` chegando no domain/data | Vazamento de camada | Grep por `BuildContext` fora de `presentation/` |
| Repositório chamando outro repositório | Violação de responsabilidade única | Análise manual ou dart_code_metrics |

---

### Ferramentas de análise de dependência no Flutter/Dart

**`dart_code_metrics`** — a principal ferramenta para análise estática em Dart:

```yaml
# pubspec.yaml
dev_dependencies:
  dart_code_metrics: ^5.x.x
```

```yaml
# analysis_options.yaml
dart_code_metrics:
  anti-patterns:
    - long-method
    - long-parameter-list
  metrics:
    cyclomatic-complexity: 10
    maximum-nesting-level: 5
    number-of-parameters: 4
    source-lines-of-code: 50
  metrics-exclude:
    - test/**
  rules:
    - no-boolean-literal-compare
    - no-empty-block
    - prefer-trailing-comma
    - avoid-unnecessary-setstate
```

Rodar a análise:

```bash
dart run dart_code_metrics:metrics analyze lib
```

Gerar relatório HTML:

```bash
dart run dart_code_metrics:metrics analyze lib --reporter=html --output-directory=reports/
```

---

### Detectar acoplamento entre módulos com git

Arquivos que sempre mudam juntos estão acoplados, mesmo que o código não mostre isso diretamente:

```bash
# Quais arquivos mudaram juntos nos últimos 100 commits?
git log --name-only --pretty=format: -n 100 | sort | uniq -c | sort -rn | head -30
```

Se `checkout_bloc.dart` e `auth_repository_impl.dart` aparecem juntos toda semana, há um acoplamento implícito que vale investigar.

---

## 2. Cobertura de Testes

Cobertura é uma métrica de **risco**, não de qualidade. 80% de cobertura não significa que 80% do seu código está bem testado — significa que 80% das linhas foram *executadas* por algum teste.

### Tipos de cobertura

| Tipo | O que mede | Exemplo |
|---|---|---|
| **Line/Statement** | Linhas executadas | `if (x > 0) return x;` — basta entrar no if |
| **Branch** | Todos os `if/else` cobertos | Precisa testar `x > 0` E `x <= 0` |
| **Path** | Combinações de caminhos | Mais granular, raro em ferramentas padrão |

**Branch coverage é o que você deve perseguir**, não line coverage.

---

### Configurando cobertura no Flutter

```bash
# Rodar testes com geração de cobertura
flutter test --coverage

# O arquivo lcov.info é gerado em coverage/lcov.info
```

Para visualizar o relatório localmente (requer `lcov` instalado):

```bash
# Linux/macOS
genhtml coverage/lcov.info -o coverage/html
open coverage/html/index.html

# Windows (via chocolatey ou WSL)
# Alternativa: usar o pacote test_coverage ou ver no CI
```

**Excluindo arquivos gerados** (models com `fromJson`, arquivos `*.g.dart`):

```bash
# Remover arquivos gerados do relatório
lcov --remove coverage/lcov.info \
  '*/**.g.dart' \
  '*/**.freezed.dart' \
  '*/**.gr.dart' \
  -o coverage/lcov_filtered.info
```

---

### Exemplo prático: testando um UseCase

```dart
// domain/usecases/login_usecase.dart
class LoginUseCase {
  final AuthRepository repository;

  LoginUseCase(this.repository);

  Future<Either<Failure, User>> call(LoginParams params) async {
    if (params.email.isEmpty || params.password.isEmpty) {
      return Left(ValidationFailure('Campos obrigatórios'));
    }
    return repository.login(params.email, params.password);
  }
}
```

**Teste com branch coverage adequado:**

```dart
// test/features/auth/domain/usecases/login_usecase_test.dart
void main() {
  late LoginUseCase useCase;
  late MockAuthRepository mockRepository;

  setUp(() {
    mockRepository = MockAuthRepository();
    useCase = LoginUseCase(mockRepository);
  });

  group('LoginUseCase', () {
    // Branch 1: email vazio
    test('deve retornar ValidationFailure quando email estiver vazio', () async {
      final result = await useCase(LoginParams(email: '', password: '123'));
      expect(result, Left(ValidationFailure('Campos obrigatórios')));
      verifyNever(mockRepository.login(any, any)); // o repo nunca foi chamado
    });

    // Branch 2: senha vazia
    test('deve retornar ValidationFailure quando senha estiver vazia', () async {
      final result = await useCase(LoginParams(email: 'a@b.com', password: ''));
      expect(result, Left(ValidationFailure('Campos obrigatórios')));
    });

    // Branch 3: caminho feliz
    test('deve retornar User quando credenciais forem válidas', () async {
      final user = User(id: '1', email: 'a@b.com');
      when(mockRepository.login('a@b.com', '123')).thenAnswer((_) async => Right(user));

      final result = await useCase(LoginParams(email: 'a@b.com', password: '123'));
      expect(result, Right(user));
    });

    // Branch 4: falha de rede
    test('deve propagar ServerFailure quando o repositório falhar', () async {
      when(mockRepository.login(any, any))
          .thenAnswer((_) async => Left(ServerFailure('Sem conexão')));

      final result = await useCase(LoginParams(email: 'a@b.com', password: '123'));
      expect(result, Left(ServerFailure('Sem conexão')));
    });
  });
}
```

Sem testar os 4 branches acima, sua cobertura de linha pode ser 100%, mas a cobertura de branch está incompleta.

---

### O que uma boa cobertura realmente exige

```
UseCase com 1 if/else         → mínimo 2 testes
Bloc com 3 eventos diferentes → mínimo 3 testes de estado resultante
Repository com fallback local → testar sucesso remoto, falha remota + sucesso local, falha total
```

---

## 3. Complexidade Ciclomática

Criada por Thomas McCabe (1976), a complexidade ciclomática (CC) mede o número de **caminhos linearmente independentes** que existem em um método.

### A fórmula

$$CC = E - N + 2P$$

Onde:
- $E$ = número de arestas no grafo de fluxo
- $N$ = número de nós
- $P$ = número de componentes conectados (geralmente 1)

**Na prática**, CC = 1 + número de decisões (cada `if`, `else if`, `for`, `while`, `case`, `&&`, `||`, `??` adiciona 1).

---

### Tabela de risco

| CC | Risco | O que fazer |
|---|---|---|
| 1–5 | Muito baixo | OK |
| 6–10 | Baixo | OK, considere testes de branch |
| 11–20 | Moderado | Refatorar se possível |
| 21–50 | Alto | Refatoração necessária |
| > 50 | Inestestável | Dívida técnica crítica |

---

### Exemplo em Flutter: Bloc com alta complexidade

```dart
// RUIM — CC ≈ 12 (difícil de testar adequadamente)
Stream<CheckoutState> mapEventToState(CheckoutEvent event) async* {
  if (event is LoadCart) {
    if (cart.items.isEmpty) {
      yield EmptyCartState();
    } else {
      yield LoadingState();
      try {
        final result = await repository.getCart();
        if (result.isRight()) {
          final cart = result.getOrElse(() => Cart.empty());
          if (cart.hasDiscount) {
            yield CartWithDiscountState(cart);
          } else {
            yield CartLoadedState(cart);
          }
        } else {
          yield CartErrorState('Erro ao carregar');
        }
      } catch (e) {
        if (e is NetworkException) {
          yield OfflineState();
        } else {
          yield CartErrorState(e.toString());
        }
      }
    }
  } else if (event is ApplyCoupon) {
    // mais lógica...
  }
}
```

```dart
// BOM — cada método tem CC ≤ 5
Stream<CheckoutState> mapEventToState(CheckoutEvent event) async* {
  if (event is LoadCart) yield* _handleLoadCart();
  if (event is ApplyCoupon) yield* _handleApplyCoupon(event);
}

Stream<CheckoutState> _handleLoadCart() async* {
  if (cart.items.isEmpty) { yield EmptyCartState(); return; }
  yield LoadingState();
  yield* _fetchAndEmitCart();
}

Stream<CheckoutState> _fetchAndEmitCart() async* {
  try {
    final result = await repository.getCart();
    yield result.fold(
      (failure) => CartErrorState(failure.message),
      (cart) => cart.hasDiscount ? CartWithDiscountState(cart) : CartLoadedState(cart),
    );
  } on NetworkException {
    yield OfflineState();
  } catch (e) {
    yield CartErrorState(e.toString());
  }
}
```

---

### Medindo CC no Flutter com dart_code_metrics

```bash
# Ver complexidade de todos os arquivos
dart run dart_code_metrics:metrics analyze lib --metrics=cyclomatic-complexity

# Filtrar apenas o que está acima do limite
dart run dart_code_metrics:metrics analyze lib \
  --metrics=cyclomatic-complexity \
  --metrics-config='{"cyclomatic-complexity": 10}'
```

Saída típica:

```
lib/features/checkout/presentation/bloc/checkout_bloc.dart
  mapEventToState (line 45)
    cyclomatic-complexity: 14 ⚠️  (threshold: 10)
```

---

### Integração no CI (GitHub Actions)

```yaml
# .github/workflows/quality.yml
- name: Dart Code Metrics
  run: |
    dart run dart_code_metrics:metrics analyze lib \
      --fatal-warnings \
      --metrics-config='{"cyclomatic-complexity": 10, "source-lines-of-code": 100}'
```

---

## 4. Testes de Mutação

É o nível mais honesto de qualidade de testes. O conceito é simples: **se você introduzir um bug no código e nenhum teste falhar, seus testes não estão validando nada de útil.**

### Como funciona

```
Código original:   if (discount > 0)
Mutante gerado:    if (discount >= 0)   ← os testes detectam essa diferença?
Mutante gerado:    if (discount < 0)    ← e essa?
Mutante gerado:    if (true)            ← e essa?
```

Cada mutante é compilado e a suite de testes é executada:
- **Mutante morto** = pelo menos um teste falhou (bom)
- **Mutante sobreviveu** = nenhum teste percebeu o bug (ruim)

### A métrica

$$\text{Mutation Score} = \frac{\text{mutantes mortos}}{\text{mutantes gerados}} \times 100$$

| Score | Interpretação |
|---|---|
| > 80% | Boa cobertura de comportamento |
| 60–80% | Aceitável para código menos crítico |
| < 60% | Testes frágeis, risco real |

---

### Mutation Testing no Flutter/Dart

Infelizmente, **não existe uma ferramenta madura de mutation testing para Dart/Flutter** ainda (2026). As alternativas são:

#### Opção 1: Stryker via Dart (experimental)

Ainda não há suporte oficial do Stryker para Dart. Acompanhe: https://github.com/stryker-mutator/stryker-dart

#### Opção 2: Mutation manual com scripts

Você pode simular mutation testing com scripts que alteram o código, rodam os testes e revertem:

```bash
#!/bin/bash
# mutation_test.sh — exemplo conceitual

ORIGINAL="if (discount > 0)"
MUTANT="if (discount >= 0)"

# Aplicar mutante
sed -i "s/$ORIGINAL/$MUTANT/g" lib/features/checkout/domain/usecases/apply_discount_usecase.dart

# Rodar testes
flutter test test/features/checkout/ --no-pub
RESULT=$?

# Reverter
sed -i "s/$MUTANT/$ORIGINAL/g" lib/features/checkout/domain/usecases/apply_discount_usecase.dart

if [ $RESULT -eq 0 ]; then
  echo "MUTANTE SOBREVIVEU ⚠️  — seus testes não detectaram o bug"
else
  echo "MUTANTE MORTO ✅ — testes detectaram a regressão"
fi
```

#### Opção 3: Revisão manual orientada a mutação

A abordagem mais prática no ecossistema Dart atual é fazer **revisão manual orientada a mutação**: ao revisar ou escrever testes, pergunte explicitamente:

```
"Se eu trocar > por >=, algum teste falha?"
"Se eu remover essa condição, algum teste falha?"
"Se eu trocar && por ||, algum teste falha?"
```

---

### Exemplo: identificando teste fraco manualmente

```dart
// Código
double applyDiscount(double price, double discountPercent) {
  if (discountPercent > 0 && discountPercent <= 100) {
    return price - (price * discountPercent / 100);
  }
  return price;
}
```

```dart
// Teste com cobertura de linha 100% mas mutation score baixo
test('aplica desconto corretamente', () {
  expect(applyDiscount(100.0, 10.0), equals(90.0));
});
```

Este teste passa, mas não detecta nenhum destes mutantes:

| Mutante | Sobrevive? |
|---|---|
| `discountPercent > 0` → `discountPercent >= 0` | ✅ sobrevive |
| `discountPercent <= 100` → `discountPercent < 100` | ✅ sobrevive |
| `&&` → `\|\|` | ✅ sobrevive |
| `price - (...)` → `price + (...)` | ❌ morre |

**Testes que matam esses mutantes:**

```dart
group('applyDiscount', () {
  test('retorna preço sem desconto quando discountPercent é 0', () {
    expect(applyDiscount(100.0, 0.0), equals(100.0)); // mata mutante >= 0
  });

  test('aplica desconto quando discountPercent é exatamente 100', () {
    expect(applyDiscount(100.0, 100.0), equals(0.0)); // mata mutante < 100
  });

  test('retorna preço sem desconto quando discountPercent é negativo', () {
    expect(applyDiscount(100.0, -10.0), equals(100.0)); // mata mutante ||
  });

  test('retorna preço sem desconto quando discountPercent excede 100', () {
    expect(applyDiscount(100.0, 110.0), equals(100.0)); // valida boundary
  });

  test('subtrai o desconto (não adiciona)', () {
    final result = applyDiscount(100.0, 10.0);
    expect(result, lessThan(100.0)); // semântica explícita
    expect(result, equals(90.0));
  });
});
```

---

### Onde aplicar mutation testing tem mais ROI

Não aplique mutation testing em tudo — é caro. Priorize:

```
✅ Domain layer (entities, value objects, use cases) — regras de negócio puras
✅ Funções de cálculo (preço, desconto, frete, juros)
✅ Validadores (CPF, email, cartão de crédito)
✅ Lógica de estado em Blocs críticos
❌ Widgets (custo alto, benefício baixo)
❌ Código de serialização gerado (*.g.dart)
❌ Glue code (repositório que só repassa chamadas)
```

---

## 5. Fluxo Recomendado

Use cada etapa como filtro para a próxima. Não adianta fazer mutation testing em código com CC 30.

```
┌─────────────────────────────────────────────────────────┐
│  1. ARQUITETURA                                         │
│     dart_code_metrics + análise manual                  │
│     Pergunta: "As camadas respeitam as fronteiras?"     │
│     Saída: lista de violações de arquitetura            │
└────────────────────┬────────────────────────────────────┘
                     │ (foque nas features com mais violações)
                     ▼
┌─────────────────────────────────────────────────────────┐
│  2. COMPLEXIDADE CICLOMÁTICA                            │
│     dart_code_metrics --metrics=cyclomatic-complexity   │
│     Pergunta: "Quais métodos são inestestáveis?"        │
│     Saída: lista de métodos com CC > 10 para refatorar  │
└────────────────────┬────────────────────────────────────┘
                     │ (após refatorar, medir cobertura)
                     ▼
┌─────────────────────────────────────────────────────────┐
│  3. BRANCH COVERAGE                                     │
│     flutter test --coverage                             │
│     Pergunta: "Onde há buracos óbvios?"                 │
│     Saída: relatório lcov com branches não cobertos     │
└────────────────────┬────────────────────────────────────┘
                     │ (nos módulos críticos com boa cobertura)
                     ▼
┌─────────────────────────────────────────────────────────┐
│  4. MUTATION TESTING (módulos críticos)                 │
│     Revisão manual orientada a mutação                  │
│     Pergunta: "Os testes validam comportamento?"        │
│     Saída: testes melhorados que matam os mutantes      │
└─────────────────────────────────────────────────────────┘
```

---

## Referências e Ferramentas

| Ferramenta | Propósito | Link |
|---|---|---|
| `dart_code_metrics` | CC, acoplamento, métricas gerais | [pub.dev](https://pub.dev/packages/dart_code_metrics) |
| `flutter test --coverage` | Cobertura de linha e branch | Nativo no Flutter SDK |
| `lcov` / `genhtml` | Visualização de cobertura | Via package manager |
| `very_good_coverage` | Enforçar threshold de cobertura no CI | [pub.dev](https://pub.dev/packages/very_good_coverage) |
| C4 Model | Documentação de arquitetura | [c4model.com](https://c4model.com) |
| Stryker (Dart — WIP) | Mutation testing | [GitHub](https://github.com/stryker-mutator/stryker-dart) |
