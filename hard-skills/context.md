# Context e SnackBar no Flutter

> `BuildContext` é uma posição na árvore de widgets — não um acesso global. Errar o context é a causa mais comum do SnackBar aparecer no lugar errado.

---

## Sumário

1. [O que é Context](#1-o-que-é-context)
2. [Como funciona `.of(context)`](#2-como-funciona-ofcontext)
3. [O problema com Dialogs e ModalBottomSheets](#3-o-problema-com-dialogs-e-modalbottomsheets)
4. [Diferença entre contexts](#4-diferença-entre-contexts)
5. [Solução para Dialog/ModalBottomSheet](#5-solução-para-dialogmodalbottomsheet)
6. [Regras práticas](#6-regras-práticas)
7. [Exemplo real — Hierarquia completa](#7-exemplo-real--hierarquia-completa)
8. [Checklist](#8-checklist)

---

## 1. O que é Context

`BuildContext` é uma referência à **posição de um widget na árvore**. Não é o widget em si - é um ponteiro que diz "estou aqui nessa hierarquia".

```dart
class MinhaTelaState extends State<MinhaTela> {
  @override
  Widget build(BuildContext context) {  // ← Context do State
    return Scaffold(  // Scaffold criado DEPOIS
      body: Builder(
        builder: (context) {  // ← Context ABAIXO do Scaffold
          // Esses são contexts DIFERENTES
        },
      ),
    );
  }
}
```

---

## 2. Como funciona `.of(context)`

Quando você chama `ScaffoldMessenger.of(context)`, o Flutter **sobe a árvore de widgets** procurando o `ScaffoldMessenger` mais próximo **acima** desse context.

```dart
MaterialApp
  └─ Scaffold ← ScaffoldMessenger está aqui
       └─ Column
            └─ Button (context) ← Chamou .of(context)
                                 ← Sobe até achar Scaffold ✅
```

---

## 3. O problema com Dialogs e ModalBottomSheets

```dart
MaterialApp
  └─ Scaffold (da tela base) ← ScaffoldMessenger está AQUI
       └─ showModalBottomSheet ← Overlay visual, mas mesma árvore lógica
            └─ Dialog ← Overlay visual, mas mesma árvore lógica
                 └─ Button (context) ← Chamou .of(context)
```

**Resultado:** `ScaffoldMessenger.of(context)` acha o Scaffold da **tela base**, que está **embaixo** do Dialog e ModalBottomSheet visualmente. A snackbar aparece escondida.

---

## 4. Diferença entre contexts

### Context do build vs Context interno

```dart
class _DialogState extends State<Dialog> {
  @override
  Widget build(BuildContext context) {  // ← Context A
    return Scaffold(  // Scaffold não existe "acima" de A
      body: Builder(
        builder: (context) {  // ← Context B (abaixo do Scaffold)
          // Context A: NÃO vê o Scaffold (mesmo nível)
          // Context B: VÊ o Scaffold (está abaixo dele)
          
          ScaffoldMessenger.of(context); // ✅ Funciona com B
          // ScaffoldMessenger.of(context A); ❌ Falha
        },
      ),
    );
  }
}
```

### Por que Builder ou métodos separados funcionam

```dart
// ❌ NÃO FUNCIONA
Scaffold(
  body: ElevatedButton(
    onPressed: () {
      ScaffoldMessenger.of(context).showSnackBar(...); // context do build
    },
  ),
)

// ✅ FUNCIONA
Scaffold(
  body: Builder(
    builder: (scaffoldContext) {  // Novo context abaixo do Scaffold
      return ElevatedButton(
        onPressed: () {
          ScaffoldMessenger.of(scaffoldContext).showSnackBar(...);
        },
      );
    },
  ),
)

// ✅ FUNCIONA (método cria novo context automaticamente)
Scaffold(
  body: _buildButton(),  // _buildButton retorna Widget com novo context
)
```

---

## 5. Solução para Dialog/ModalBottomSheet

### Opção 1: Scaffold fora do Dialog (recomendado)
```dart
Scaffold(  // ← ScaffoldMessenger aqui
  body: showDialog(
    builder: (context) => Dialog(
      child: Container(...),
    ),
  ),
)
```
**Resultado:** SnackBar aparece no fundo da tela, visível acima de tudo.

### Opção 2: Scaffold dentro do Dialog
```dart
Dialog(
  child: Scaffold(  // ← ScaffoldMessenger aqui
    body: Container(...),
  ),
)
```
**Resultado:** SnackBar aparece dentro do Dialog (flutuando no meio).

### Opção 3: GlobalKey (máximo controle)
```dart
final _scaffoldKey = GlobalKey<ScaffoldMessengerState>();

ScaffoldMessenger(
  key: _scaffoldKey,
  child: Scaffold(...),
)

// Uso
_scaffoldKey.currentState?.showSnackBar(...);
```
**Vantagem:** Não depende de context, acesso direto.

---

## 6. Regras práticas

1. **Context = posição na árvore**, não é "a tela toda"
2. **`.of(context)` sobe a árvore** procurando widgets acima
3. **Dialog/Modal não isolam context** - ainda apontam pra árvore base
4. **Para controlar onde algo aparece:** coloque Scaffold/Overlay na camada certa
5. **Builder/métodos criam novo context** automaticamente abaixo do widget pai

---

## 7. Exemplo real — Hierarquia completa

```dart
MaterialApp
  └─ Scaffold (Tela Base)
       │  ├─ AppBar
       │  └─ Body
       │       └─ Button "Abrir Dialog"
       │
       └─ showDialog (cria Overlay Route)
            └─ Dialog
                 └─ Content
                      └─ Button "Salvar"
                           └─ onPressed chama usecase
                                └─ usecase chama SnackBar
                                     └─ ScaffoldMessenger.of(context)
                                          └─ Sobe até Scaffold (Tela Base)
                                               └─ SnackBar aparece EMBAIXO do Dialog ❌
```

**Solução:** Scaffold envolvendo `showDialog` garante que context aponta pro Scaffold correto.

---

## 8. Checklist

- [ ] O context usado em `.of(context)` está **abaixo** do widget alvo na árvore
- [ ] Dialogs e ModalBottomSheets que precisam exibir SnackBar usam `Builder` ou têm Scaffold próprio
- [ ] Nenhum `ScaffoldMessenger.of(context)` usa o context do `build()` diretamente quando há Scaffold no mesmo nível
- [ ] `GlobalKey<ScaffoldMessengerState>` considerado quando o context é inacessível no ponto de uso
