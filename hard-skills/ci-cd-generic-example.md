# CI/CD para Flutter

> Automatizar build, testes e distribuição não é luxo — é a base que permite entregar com confiança e consistência em qualquer velocidade.

---

## Sumário

1. [Conceitos-chave](#1-conceitos-chave)
2. [Fluxo recomendado](#2-fluxo-recomendado)
3. [GitHub Actions](#3-github-actions)
4. [Codemagic](#4-codemagic)
5. [Branch protection](#5-branch-protection)
6. [Ordem de implementação](#6-ordem-de-implementação)
7. [Checklist](#7-checklist)

---

## 1. Conceitos-chave

| Termo | Significado |
|---|---|
| **CI** | Continuous Integration — build + testes automáticos a cada push/PR |
| **CD** | Continuous Delivery — entrega automática do build para um destino |
| **Workflow** | Arquivo YAML que define quando e o que executar |
| **Runner** | Máquina virtual que executa o workflow |
| **Artifact** | Arquivo gerado pelo build (APK, IPA, AAB) |
| **Secret** | Variável sensível (senha, chave) armazenada de forma segura |

---

## 2. Fluxo recomendado

```
PR aberto
  └── CI: analyze + test + build check

Merge na main
  └── CD: build release + entrega para testadores (Firebase / TestFlight)

Tag criada (v1.0.0)
  └── CD: build release + publicar nas stores
```

---

## 3. GitHub Actions

### Estrutura de arquivos

```
.github/
  workflows/
    ci.yml
    cd-staging.yml
    cd-production.yml
```

### ci.yml — Validação em todo PR

```yaml
name: CI

on:
  pull_request:
    branches: [main, develop]

jobs:
  analyze-and-test:
    name: Analyze & Test
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.19.x'
          channel: 'stable'
          cache: true

      - name: Install dependencies
        run: flutter pub get

      - name: Analyze
        run: flutter analyze --fatal-infos

      - name: Run tests
        run: flutter test --coverage

      - name: Upload coverage (opcional)
        uses: codecov/codecov-action@v4
        with:
          token: ${{ secrets.CODECOV_TOKEN }}
          file: coverage/lcov.info
```

---

### cd-staging.yml — Build e entrega para testadores

```yaml
name: CD Staging

on:
  push:
    branches: [main]

jobs:
  build-android:
    name: Build Android (AAB)
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-java@v4
        with:
          distribution: 'zulu'
          java-version: '17'

      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.19.x'
          channel: 'stable'
          cache: true

      - run: flutter pub get

      - name: Decode keystore
        run: |
          echo "${{ secrets.KEYSTORE_BASE64 }}" | base64 --decode > android/app/keystore.jks

      - name: Build AAB
        env:
          KEY_ALIAS: ${{ secrets.KEY_ALIAS }}
          KEY_PASSWORD: ${{ secrets.KEY_PASSWORD }}
          STORE_PASSWORD: ${{ secrets.STORE_PASSWORD }}
        run: |
          flutter build appbundle --release \
            --dart-define=ENV=staging

      - name: Upload to Firebase App Distribution
        uses: wzieba/Firebase-Distribution-Github-Action@v1
        with:
          appId: ${{ secrets.FIREBASE_APP_ID_ANDROID }}
          serviceCredentialsFileContent: ${{ secrets.FIREBASE_SERVICE_ACCOUNT }}
          groups: testers
          file: build/app/outputs/bundle/release/app-release.aab

  build-ios:
    name: Build iOS (IPA)
    runs-on: macos-latest

    steps:
      - uses: actions/checkout@v4

      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.19.x'
          channel: 'stable'

      - run: flutter pub get

      - name: Import certificate e provisioning profile
        env:
          BUILD_CERTIFICATE_BASE64: ${{ secrets.BUILD_CERTIFICATE_BASE64 }}
          P12_PASSWORD: ${{ secrets.P12_PASSWORD }}
          BUILD_PROVISION_PROFILE_BASE64: ${{ secrets.BUILD_PROVISION_PROFILE_BASE64 }}
          KEYCHAIN_PASSWORD: ${{ secrets.KEYCHAIN_PASSWORD }}
        run: |
          CERTIFICATE_PATH=$RUNNER_TEMP/build_certificate.p12
          PP_PATH=$RUNNER_TEMP/build_pp.mobileprovision
          KEYCHAIN_PATH=$RUNNER_TEMP/app-signing.keychain-db

          echo -n "$BUILD_CERTIFICATE_BASE64" | base64 --decode -o $CERTIFICATE_PATH
          echo -n "$BUILD_PROVISION_PROFILE_BASE64" | base64 --decode -o $PP_PATH

          security create-keychain -p "$KEYCHAIN_PASSWORD" $KEYCHAIN_PATH
          security set-keychain-settings -lut 21600 $KEYCHAIN_PATH
          security unlock-keychain -p "$KEYCHAIN_PASSWORD" $KEYCHAIN_PATH
          security import $CERTIFICATE_PATH -P "$P12_PASSWORD" -A -t cert -f pkcs12 -k $KEYCHAIN_PATH
          security list-keychain -d user -s $KEYCHAIN_PATH

          mkdir -p ~/Library/MobileDevice/Provisioning\ Profiles
          cp $PP_PATH ~/Library/MobileDevice/Provisioning\ Profiles

      - name: Build IPA
        run: |
          flutter build ipa --release \
            --export-options-plist=ios/ExportOptions.plist

      - name: Upload to TestFlight
        uses: Apple-Actions/upload-testflight-build@v1
        with:
          app-path: build/ios/ipa/*.ipa
          issuer-id: ${{ secrets.APPSTORE_ISSUER_ID }}
          api-key-id: ${{ secrets.APPSTORE_API_KEY_ID }}
          api-private-key: ${{ secrets.APPSTORE_API_PRIVATE_KEY }}
```

---

### cd-production.yml — Publicar nas stores via tag

```yaml
name: CD Production

on:
  push:
    tags:
      - 'v*.*.*'   # dispara em tags como v1.2.0

jobs:
  deploy-android:
    name: Deploy Google Play (Production)
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: 'zulu'
          java-version: '17'
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.19.x'
          cache: true

      - run: flutter pub get

      - name: Decode keystore
        run: echo "${{ secrets.KEYSTORE_BASE64 }}" | base64 --decode > android/app/keystore.jks

      - name: Build AAB
        run: flutter build appbundle --release

      - name: Upload to Google Play
        uses: r0adkll/upload-google-play@v1
        with:
          serviceAccountJsonPlainText: ${{ secrets.GOOGLE_PLAY_SERVICE_ACCOUNT }}
          packageName: com.empresa.app
          releaseFiles: build/app/outputs/bundle/release/app-release.aab
          track: production

  deploy-ios:
    name: Deploy App Store (Production)
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.19.x'
      - run: flutter pub get
      - name: Build IPA
        run: flutter build ipa --release --export-options-plist=ios/ExportOptions.plist
      - name: Upload to App Store
        uses: Apple-Actions/upload-testflight-build@v1
        with:
          app-path: build/ios/ipa/*.ipa
          issuer-id: ${{ secrets.APPSTORE_ISSUER_ID }}
          api-key-id: ${{ secrets.APPSTORE_API_KEY_ID }}
          api-private-key: ${{ secrets.APPSTORE_API_PRIVATE_KEY }}
```

---

### Secrets necessários (GitHub → Settings → Secrets)

| Secret | Descrição |
|---|---|
| `KEYSTORE_BASE64` | Keystore Android em base64 (`base64 keystore.jks`) |
| `KEY_ALIAS` | Alias da chave no keystore |
| `KEY_PASSWORD` | Senha da chave |
| `STORE_PASSWORD` | Senha do keystore |
| `FIREBASE_APP_ID_ANDROID` | App ID no Firebase |
| `FIREBASE_SERVICE_ACCOUNT` | JSON da service account do Firebase |
| `BUILD_CERTIFICATE_BASE64` | Certificado .p12 iOS em base64 |
| `P12_PASSWORD` | Senha do certificado .p12 |
| `BUILD_PROVISION_PROFILE_BASE64` | Provisioning profile em base64 |
| `KEYCHAIN_PASSWORD` | Senha temporária para o keychain no runner |
| `APPSTORE_ISSUER_ID` | Issuer ID da App Store Connect API |
| `APPSTORE_API_KEY_ID` | Key ID da App Store Connect API |
| `APPSTORE_API_PRIVATE_KEY` | Chave privada (.p8) da App Store Connect API |
| `GOOGLE_PLAY_SERVICE_ACCOUNT` | JSON da service account do Google Play |

---

## 4. Codemagic

Plataforma de CI/CD mobile-first com suporte nativo a Flutter. Configuração via `codemagic.yaml` na raiz do projeto.

### Estrutura mínima do codemagic.yaml

```yaml
workflows:
  ci-pull-request:
    name: CI - Pull Request
    triggering:
      events: [pull_request]
      branch_patterns:
        - pattern: main
        - pattern: develop
    environment:
      flutter: 3.19.x
    scripts:
      - name: Install dependencies
        script: flutter pub get
      - name: Analyze
        script: flutter analyze --fatal-infos
      - name: Run tests
        script: flutter test --coverage

  cd-staging-android:
    name: CD Staging - Android
    triggering:
      events: [push]
      branch_patterns:
        - pattern: main
    environment:
      flutter: 3.19.x
      android_signing:
        - keystore_reference    # nome do keystore cadastrado no Codemagic
      groups:
        - firebase_credentials  # grupo de variáveis no Codemagic
    scripts:
      - name: Install dependencies
        script: flutter pub get
      - name: Build AAB
        script: |
          flutter build appbundle --release \
            --dart-define=ENV=staging
    artifacts:
      - build/app/outputs/bundle/release/app-release.aab
    publishing:
      firebase:
        firebase_token: $FIREBASE_TOKEN
        android:
          app_id: $FIREBASE_APP_ID_ANDROID
          groups:
            - testers

  cd-staging-ios:
    name: CD Staging - iOS
    triggering:
      events: [push]
      branch_patterns:
        - pattern: main
    environment:
      flutter: 3.19.x
      ios_signing:
        distribution_type: ad_hoc       # ou app_store para produção
        bundle_identifier: com.empresa.app
      groups:
        - appstore_credentials
    scripts:
      - name: Install dependencies
        script: flutter pub get
      - name: Build IPA
        script: |
          flutter build ipa --release \
            --export-options-plist=$CM_BUILD_DIR/ios/ExportOptions.plist
    artifacts:
      - build/ios/ipa/*.ipa
    publishing:
      firebase:
        firebase_token: $FIREBASE_TOKEN
        ios:
          app_id: $FIREBASE_APP_ID_IOS
          groups:
            - testers

  cd-production:
    name: CD Production - Stores
    triggering:
      tag_patterns:
        - pattern: 'v*'    # dispara em tags como v1.2.0
    environment:
      flutter: 3.19.x
      android_signing:
        - keystore_reference
      ios_signing:
        distribution_type: app_store
        bundle_identifier: com.empresa.app
      groups:
        - firebase_credentials
        - appstore_credentials
        - google_play_credentials
    scripts:
      - name: Install dependencies
        script: flutter pub get
      - name: Build Android
        script: flutter build appbundle --release
      - name: Build iOS
        script: flutter build ipa --release --export-options-plist=$CM_BUILD_DIR/ios/ExportOptions.plist
    artifacts:
      - build/app/outputs/bundle/release/app-release.aab
      - build/ios/ipa/*.ipa
    publishing:
      google_play:
        credentials: $GOOGLE_PLAY_SERVICE_ACCOUNT
        track: production
      app_store_connect:
        api_key: $APP_STORE_CONNECT_PRIVATE_KEY
        key_id: $APP_STORE_CONNECT_KEY_IDENTIFIER
        issuer_id: $APP_STORE_CONNECT_ISSUER_ID
        submit_to_testflight: true
```

---

### Variáveis de ambiente no Codemagic

No painel do Codemagic: **Teams → Environment variables** ou direto no app.

Organize por **grupos** e referencie no YAML:

```yaml
environment:
  groups:
    - firebase_credentials      # FIREBASE_TOKEN, FIREBASE_APP_ID_ANDROID, etc.
    - appstore_credentials      # APP_STORE_CONNECT_*, CERTIFICATE_*, etc.
    - google_play_credentials   # GOOGLE_PLAY_SERVICE_ACCOUNT
```

Para o keystore Android, use a seção dedicada do Codemagic (Teams → Code signing → Android) — ele cuida do encode/decode automaticamente. O mesmo vale para certificados iOS.

---

### Comparativo: GitHub Actions vs Codemagic

| Critério | GitHub Actions | Codemagic |
|---|---|---|
| **Curva de aprendizado** | Média | Baixa (Flutter nativo) |
| **Configuração iOS signing** | Manual (scripts bash) | Automática via UI |
| **Minutos gratuitos** | 2.000/mês (projetos privados) | 500/mês |
| **Runner macOS** | Pago (10x custo) | Incluído |
| **Flexibilidade** | Alta (qualquer linguagem/stack) | Focado em mobile |
| **Melhor para** | Times que já usam GitHub fortemente | Times focados em mobile/Flutter |

---

## 5. Branch protection

Independente da ferramenta, configure no GitHub:

```
Settings → Branches → Branch protection rules → main

[x] Require a pull request before merging
[x] Require status checks to pass before merging
    └── Adicione o job do CI (ex: "analyze-and-test")
[x] Require branches to be up to date before merging
[x] Do not allow bypassing the above settings
```

---

## 6. Ordem de implementação

| Etapa | O que fazer | Valor entregue |
|---|---|---|
| 1 | CI: `analyze` + `test` em todo PR | Impede código quebrado de chegar na main |
| 2 | Branch protection | Ninguém pula o CI |
| 3 | CD: build automático ao mergear na main | Testadores sempre têm build atualizado |
| 4 | CD: publicar nas stores via tag | Release controlado e sem processo manual |

---

## 7. Checklist

- [ ] CI configurado: `flutter analyze` + `flutter test` em todo PR
- [ ] Branch protection ativado na `main` — CI deve passar antes de merge
- [ ] Secrets de assinatura configurados (keystore Android, certificado iOS)
- [ ] CD de staging: build automático ao mergear na `main`
- [ ] Distribuição para testadores via Firebase App Distribution ou TestFlight
- [ ] CD de produção disparado por tag (`v*.*.*`) — não por push direto
- [ ] `flutter analyze --fatal-infos` no CI — não só erros
