# Arquitetura de Formulários: A Analogia do Corpo Humano

## 🎯 Conceito Central

**FormState é o blueprint de um corpo humano sendo montado. Entity é o ser humano completo e vivo.**

Durante a construção (preenchimento do formulário), o corpo pode estar incompleto, inválido ou em processo de montagem. Apenas quando todas as partes estão completas e validadas, o blueprint se torna um ser humano real (entidade).

---

## 📐 As Três Camadas

### 1. **FormState** - O Blueprint em Construção
**Localização:** `presenter/pages/[feature]/[nome]_form_state.dart`

**O que é:** Estado transitório do formulário. Representa um corpo humano sendo montado - pode estar incompleto, com partes faltando ou inválidas.

**Características:**
- Pode estar incompleto ou inválido
- Contém estado de UI (`isSubmitting`, `errors`, `isDirty`)
- Imutável (usa `copyWith` para mudanças)
- Existe apenas durante o processo de preenchimento

```dart
class ClienteFormState extends Equatable {
  // Partes do corpo sendo montadas
  final String nome;           // Pode estar vazio
  final String cpf;            // Pode estar incompleto
  final String email;          // Pode ter formato errado
  
  // Estado da montagem (não faz parte do corpo final)
  final Map<String, String> errors;  // Problemas encontrados
  final bool isValid;                 // Está completo?
  final bool isSubmitting;            // Processo de finalização
  final bool isDirty;                 // Alguma parte foi mexida?
  
  const ClienteFormState({
    this.nome = '',
    this.cpf = '',
    this.email = '',
    this.errors = const {},
    this.isValid = false,
    this.isSubmitting = false,
    this.isDirty = false,
  });

  // Criar nova versão do blueprint com alterações
  ClienteFormState copyWith({
    String? nome,
    String? cpf,
    String? email,
    Map<String, String>? errors,
    bool? isValid,
    bool? isSubmitting,
    bool? isDirty,
  }) {
    return ClienteFormState(
      nome: nome ?? this.nome,
      cpf: cpf ?? this.cpf,
      email: email ?? this.email,
      errors: errors ?? this.errors,
      isValid: isValid ?? this.isValid,
      isSubmitting: isSubmitting ?? this.isSubmitting,
      isDirty: isDirty ?? this.isDirty,
    );
  }

  // Verificar se o corpo está completo e válido
  ClienteFormState validate() {
    final errors = <String, String>{};
    
    if (nome.isEmpty) {
      errors['nome'] = 'Nome é obrigatório';
    }
    
    if (cpf.isEmpty || !_isValidCPF(cpf)) {
      errors['cpf'] = 'CPF inválido';
    }
    
    if (email.isEmpty || !email.contains('@')) {
      errors['email'] = 'Email inválido';
    }
    
    return copyWith(
      errors: errors,
      isValid: errors.isEmpty,
    );
  }

  // ⚡ Transformar blueprint em ser humano (entidade)
  // Só pode ser chamado quando válido
  Cliente toEntity({required int id}) {
    if (!isValid) {
      throw StateError('Blueprint incompleto não pode virar ser humano');
    }
    
    return Cliente(
      id: id,
      nome: nome,
      cpf: cpf,
      email: email,
    );
  }

  // ⚡ Copiar ser humano existente para blueprint (modo edição)
  static ClienteFormState fromEntity(Cliente cliente) {
    return ClienteFormState(
      nome: cliente.nome,
      cpf: cliente.cpf,
      email: cliente.email,
    ).validate();
  }

  bool _isValidCPF(String cpf) {
    // Lógica de validação
    return cpf.length == 14;
  }

  @override
  List<Object?> get props => [
    nome, cpf, email, errors, isValid, isSubmitting, isDirty
  ];
}
```

---

### 2. **Entity** - O Ser Humano Completo
**Localização:** `domain/entities/[nome]_entity.dart`

**O que é:** Representação completa e válida de um conceito de negócio. Um ser humano vivo - todas as partes essenciais estão presentes e funcionando.

**Características:**
- Sempre válido e completo
- Imutável (nunca é alterado diretamente)
- Não tem estado de UI
- Existe no banco de dados / API

```dart
class Cliente extends Equatable {
  final int id;
  final String nome;
  final String cpf;
  final String email;
  final ClienteStatus status;
  
  const Cliente({
    required this.id,
    required this.nome,
    required this.cpf,
    required this.email,
    required this.status,
  });

  // ✅ Regras de negócio de DOMÍNIO
  bool get isAtivo => status == ClienteStatus.ativo;
  
  // Criar novo ser humano com características alteradas
  Cliente copyWith({
    int? id,
    String? nome,
    String? cpf,
    String? email,
    ClienteStatus? status,
  }) {
    return Cliente(
      id: id ?? this.id,
      nome: nome ?? this.nome,
      cpf: cpf ?? this.cpf,
      email: email ?? this.email,
      status: status ?? this.status,
    );
  }

  @override
  List<Object?> get props => [id, nome, cpf, email, status];
}
```

---

### 3. **Cubit/Controller** - O Construtor/Engenheiro
**Localização:** `presenter/pages/[feature]/[nome]_form_cubit.dart`

**O que é:** Orquestrador que gerencia o processo de construção. O engenheiro que monta o corpo humano e decide quando ele está pronto para "nascer" (virar entidade).

**Características:**
- Gerencia transições de estado do FormState
- Valida antes de converter para entidade
- Chama use cases do domínio
- Lida com erros e loading states

```dart
class ClienteFormCubit extends Cubit<ClienteFormState> {
  final CreateClienteUseCase _createUseCase;
  final UpdateClienteUseCase _updateUseCase;
  
  ClienteFormCubit(
    this._createUseCase,
    this._updateUseCase,
  ) : super(const ClienteFormState());
  
  // 📋 Modo CADASTRO: Começar com blueprint vazio
  void startNewForm() {
    emit(const ClienteFormState());
  }
  
  // ✏️ Modo EDIÇÃO: Copiar ser humano existente para blueprint
  void loadClienteForEdit(Cliente cliente) {
    emit(ClienteFormState.fromEntity(cliente));
  }
  
  // 🔨 Construir partes individuais
  void updateNome(String nome) {
    emit(state.copyWith(nome: nome, isDirty: true).validate());
  }
  
  void updateCpf(String cpf) {
    emit(state.copyWith(cpf: cpf, isDirty: true).validate());
  }
  
  void updateEmail(String email) {
    emit(state.copyWith(email: email, isDirty: true).validate());
  }
  
  // ✨ Finalizar construção: Transformar blueprint em ser humano
  Future<void> submit({int? clienteId}) async {
    // Força validação para mostrar todos os erros
    if (!state.isValid) {
      emit(state.validate());
      return;
    }
    
    // Inicia processo de "nascimento"
    emit(state.copyWith(isSubmitting: true));
    
    try {
      // Blueprint vira ser humano
      final entity = state.toEntity(id: clienteId ?? 0);
      
      // Decide se cria novo ou atualiza existente
      final result = clienteId != null
        ? await _updateUseCase(entity)
        : await _createUseCase(entity);
      
      result.fold(
        (failure) => emit(state.copyWith(
          isSubmitting: false,
          errors: {'submit': failure.message},
          isValid: false,
        )),
        (success) {
          emit(state.copyWith(isSubmitting: false));
          // Navegar de volta, mostrar toast de sucesso, etc
        },
      );
    } catch (e) {
      emit(state.copyWith(
        isSubmitting: false,
        errors: {'submit': e.toString()},
      ));
    }
  }
  
  // 🧹 Resetar blueprint
  void reset() {
    emit(const ClienteFormState());
  }
}
```

---

## 🔄 Fluxos Completos

### **Fluxo 1: CADASTRO (Novo Ser Humano)**

```
1. Tela abre
   ↓
2. FormState vazio criado
   ClienteFormState(nome: '', cpf: '', email: '', isValid: false)
   ↓
3. Usuário preenche nome
   cubit.updateNome('João')
   ↓
4. Novo FormState emitido
   ClienteFormState(nome: 'João', cpf: '', email: '', isValid: false)
   ↓
5. Usuário preenche CPF
   cubit.updateCpf('123.456.789-00')
   ↓
6. Novo FormState emitido
   ClienteFormState(nome: 'João', cpf: '123.456.789-00', email: '', isValid: false)
   ↓
7. Usuário preenche email
   cubit.updateEmail('joao@exemplo.com')
   ↓
8. Novo FormState emitido (agora válido!)
   ClienteFormState(nome: 'João', cpf: '123...', email: 'joao@...', isValid: true)
   ↓
9. Usuário clica em "Salvar"
   cubit.submit()
   ↓
10. FormState → Entity (blueprint vira ser humano)
    Cliente(id: 0, nome: 'João', cpf: '123...', email: 'joao@...')
    ↓
11. UseCase persiste no banco
    ↓
12. Sucesso! Ser humano nasceu e está no mundo (banco de dados)
```

---

### **Fluxo 2: EDIÇÃO (Clonar e Modificar Ser Humano)**

```
1. Usuário clica em "Editar Cliente"
   ↓
2. Cliente existente carregado do banco
   Cliente(id: 123, nome: 'João Silva', cpf: '123...', email: 'joao@...')
   ↓
3. Entity → FormState (ser humano copiado para blueprint)
   cubit.loadClienteForEdit(cliente)
   ↓
4. FormState criado a partir da entidade
   ClienteFormState.fromEntity(cliente)
   ClienteFormState(nome: 'João Silva', cpf: '123...', email: 'joao@...', isValid: true)
   ↓
5. Usuário altera o nome
   cubit.updateNome('João Santos')
   ↓
6. Novo FormState emitido
   ClienteFormState(nome: 'João Santos', cpf: '123...', email: 'joao@...', isDirty: true)
   ↓
7. Usuário clica em "Salvar"
   cubit.submit(clienteId: 123)
   ↓
8. FormState → Entity (blueprint vira NOVO ser humano)
   Cliente(id: 123, nome: 'João Santos', cpf: '123...', email: 'joao@...')
   ⚠️ Entidade ORIGINAL nunca foi tocada!
   ↓
9. UseCase atualiza no banco
   ↓
10. Sucesso! Novo ser humano com características alteradas
```

---

## 📊 Comparação: Antes vs Depois

### ❌ ANTES (Variáveis Soltas)

```dart
class ClienteFormState {
  final nomeController = TextEditingController();
  final cpfController = TextEditingController();
  final emailController = TextEditingController();
  String? nomeError;
  String? cpfError;
  String? emailError;
  bool isLoading = false;
  
  void salvar() {
    // Pegar valores dos controllers
    // Validar cada um
    // Criar entidade manualmente
    // ??? Estado inconsistente
  }
}
```

**Problemas:**
- Estado espalhado em 7+ variáveis
- Sem histórico de mudanças
- Validação fragmentada
- Impossível testar atomicamente
- Bugs sutis de estado

---

### ✅ DEPOIS (FormState Coeso)

```dart
class ClienteFormState extends Equatable {
  final String nome;
  final String cpf;
  final String email;
  final Map<String, String> errors;
  final bool isValid;
  final bool isSubmitting;
  
  // copyWith, validate, toEntity, fromEntity...
}

class ClienteFormCubit extends Cubit<ClienteFormState> {
  void updateNome(String nome) {
    emit(state.copyWith(nome: nome).validate());
  }
  // ...
}
```

**Benefícios:**
- Estado coeso em uma classe
- Histórico automático (cada emit é um snapshot)
- Validação centralizada
- Testes triviais
- Estado sempre consistente

---

## 🎓 Separação de Responsabilidades

| Aspecto | FormState | Entity |
|---------|-----------|--------|
| **Camada** | `presenter/` | `domain/` |
| **Estado** | Transitório, pode ser inválido | Sempre válido |
| **Quando existe** | Durante preenchimento | Após validação completa |
| **Validações** | Regras de UI/UX | Regras de negócio |
| **Campos extras** | `errors`, `isSubmitting`, `isDirty` | Apenas dados de domínio |
| **Mutabilidade** | Imutável (via `copyWith`) | Imutável (via `copyWith`) |
| **Analogia** | Blueprint/Corpo sendo montado | Ser humano vivo |

---

## 🚦 Quando Usar FormState

### ✅ USE para:
- Formulários com **5+ campos**
- Formulários com **validações interdependentes** (campo A afeta campo B)
- Formulários com **regras de limpeza em cascata** (mudar X limpa Y e Z)
- Quando precisa de **undo/redo**
- Quando precisa **salvar rascunho**
- Formulários de **cadastro e edição** complexos

### ❌ NÃO USE para:
- Formulários simples (2-3 campos independentes)
- Campos isolados (ex: busca com 1 TextField)
- Widgets reutilizáveis genéricos

---

## 💡 Regras de Ouro

1. **FormState não é entidade** - São camadas diferentes
2. **FormState pode ser inválido** - Entity nunca
3. **Validação no FormState** - Regras de negócio na Entity
4. **Conversão unidirecional** - FormState → Entity (quando válido)
5. **Imutabilidade sempre** - Nunca altere diretamente, use `copyWith`
6. **Edição = fromEntity** - Nunca modifique a entidade original
7. **Estado de UI no FormState** - `isSubmitting`, `errors`, `isDirty` não vão para Entity

---

## 📝 Checklist de Implementação

Ao criar um novo formulário complexo:

- [ ] Criar `[nome]_form_state.dart` na camada `presenter`
- [ ] Adicionar todos os campos do formulário como propriedades
- [ ] Adicionar campos de controle de UI (`errors`, `isValid`, `isSubmitting`, `isDirty`)
- [ ] Implementar `copyWith()`
- [ ] Implementar `validate()` com todas as regras
- [ ] Implementar `toEntity()` para conversão
- [ ] Implementar `fromEntity()` para modo edição
- [ ] Criar `[nome]_form_cubit.dart`
- [ ] Implementar métodos `update[Campo]()` para cada campo
- [ ] Implementar `submit()` com chamada ao UseCase
- [ ] Implementar `loadForEdit()` se houver modo edição
- [ ] Testar transições de estado
- [ ] Conectar na UI com `BlocBuilder` ou `ValueListenableBuilder`

---

## 🎯 Conclusão

**FormState é o blueprint de um corpo humano sendo construído.**

Durante o processo, o corpo pode estar incompleto, com partes faltando ou inválidas. Apenas quando **todas as validações passam**, o blueprint se transforma em um **ser humano completo** (entidade).

No modo edição, você **copia um ser humano existente** para um blueprint, faz as modificações necessárias e, ao final, **cria um novo ser humano** com as características alteradas. **O original nunca é tocado** - isso é imutabilidade.

Essa arquitetura traz:
- ✅ Clareza de código
- ✅ Estado previsível
- ✅ Testes simples
- ✅ Menos bugs
- ✅ Escalabilidade

**Use para formulários complexos.**

---

*Documento criado em: 01/02/2026*
*Última atualização: 01/02/2026*
