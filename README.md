# 🔐 collectionTestApiAuthService

Collection do Postman para testes de integração (end-to-end) do **AuthService** do projeto *Catálogo de Eventos*. A collection cobre os fluxos de **login** e **validação de token JWT**, incluindo cenários de sucesso, erro de validação e erros de autenticação/autorização.

---

## ▶️ Como executar

### Opção 1 — Postman (interface gráfica)
1. Importe a collection em `postman/collections/auth-service.postman_collection`.
2. Importe o(s) environment(s) desejado(s) em `postman/environments/`.
3. Selecione o environment no canto superior direito do Postman.
4. Preencha as variáveis obrigatórias (ver seção [Variáveis](#️-variáveis-necessárias-para-executar-a-collection) abaixo).
5. Execute a collection inteira via **Runner**, ou cada request individualmente.

### Opção 2 — Newman (linha de comando / CI)
```bash
npm install -g newman

newman run postman/collections/auth-service.postman_collection \
  -e postman/environments/local-mock.environment.json
```
Basta trocar o arquivo de environment (`-e`) para rodar contra outro ambiente (`local-real`, `ci` ou `production`).

> ℹ️ A execução via cli para os ambienntes de `local-real.environment.json` e `production.environment.json` exige ingetar as variaveis `adminEmail` `adminPassword` já que por segurança elas não são definidas nesses environments

---

## ⚙️ Variáveis necessárias para executar a Collection

Para que esta collection funcione corretamente no Postman, configure as seguintes variáveis no **Environment**:

---

### 🌐 URLs dos serviços obrigatórios

| Variável            | Descrição                                     |
|---------------------|------------------------------------------------|
| `auth_service_url`  | URL base do AuthService (ex.: `http://localhost:3232/auth`) |
| `user_service_url`  | URL base do UserService, usado para criar/remover o usuário de teste (ex.: `http://localhost:8089/users`) |

---

### 🎭 Modo de execução (mock x real)

| Variável  | Descrição                                                                 |
|-----------|-----------------------------------------------------------------------------|
| `useMock` | `"true"` para rodar contra um serviço mockado (sem criar/remover usuários reais nem depender do UserService); `"false"` para rodar a integração real ponta a ponta. |

Quando `useMock` é `"true"`:
- A criação do usuário temporário é simulada (não chama o `user_service_url`), usando o e-mail fixo `mock.user@test.com`.
- As requisições de login enviam o header `x-mock-scenario`, permitindo que o mock retorne a resposta correspondente ao cenário testado (ex.: `SUCCESS_VALIDATE_USER`, `SUCCESS_VALIDATE_ADMIN`).
- A exclusão do usuário de teste ao final dos testes é ignorada.

---

### 👤 Credenciais do Admin

Essas variáveis permitem que a collection teste o login como administrador.

| Variável        | Descrição                 |
|-----------------|----------------------------|
| `adminEmail`    | Email do administrador     |
| `adminPassword` | Senha do administrador     |

---

### 🧪 Usuário temporário (preenchido automaticamente)

Essas variáveis são criadas e atualizadas automaticamente pelos scripts da collection (pre-request/test scripts), a partir das funções auxiliares `createSimpleUserRequest`, `deleteSimpleUserRequest` e `performLogin` — registradas como variáveis de collection e reutilizadas entre requests. Não é necessário preenchê-las manualmente.

| Variável                     | Preenchida automaticamente | Descrição                     |
|------------------------------|-----------------------------|----------------------------------|
| `userRecentCadastreId`       | ✔️                          | ID do usuário criado             |
| `userRecentCadastreEmail`    | ✔️                          | Email do usuário criado          |
| `userRecentCadastrePassword` | ✔️                          | Senha usada na criação           |
| `userRecentCadastreToken`    | ✔️                          | Token JWT do usuário criado      |

> ℹ️ O usuário temporário é criado antes dos testes que dependem dele e removido (via `deleteSimpleUserRequest`) logo após o teste concluir, evitando resíduos de dados entre execuções.

---

## ✔️ Resumo rápido

### 🔧 Configure manualmente:
- `auth_service_url`
- `user_service_url`
- `useMock`
- `adminEmail`
- `adminPassword`

### 🤖 Variáveis gerenciadas automaticamente pelos scripts:
- `userRecentCadastreId`
- `userRecentCadastreEmail`
- `userRecentCadastrePassword`
- `userRecentCadastreToken`

---

## 🧪 Testes da collection

A collection está organizada em duas pastas de testes: **Login** e **Validação do token**. Todos os requests validam o status HTTP retornado, a presença da propriedade `message` no corpo da resposta e o conteúdo textual da mensagem retornada.

### 📂 Login (`POST {{auth_service_url}}/login`)

| Cenário | O que valida | Retorno esperado |
|---|---|---|
| **Login de usuário com dados corretos** | Cria um usuário temporário (via `createSimpleUserRequest`) e efetua login com o e-mail e senha gerados. Confirma que a resposta contém `message` e `userToken`. Ao final, remove o usuário criado. | `200 OK` |
| **Login de Admin** | Efetua login com as credenciais de administrador (`adminEmail`/`adminPassword`) e confirma o retorno de `message` e `userToken`. | `200 OK` |
| **Login de usuário com email invalido** | Envia um e-mail em formato inválido (sem `@`). Confirma que a API rejeita e retorna a mensagem "Email deve ser um email válido". | `400 Bad Request` |
| **Login de usuário sem fornecer email** | Envia apenas a senha, omitindo o campo de e-mail. Confirma a mensagem "Email é obrigatório". | `400 Bad Request` |
| **Login de usuário sem fornecer senha** | Envia apenas o e-mail, omitindo a senha. Confirma a mensagem "Senha é obrigatória". | `400 Bad Request` |
| **Login de usuário com email não cadastrado** | Utiliza um e-mail que não existe na base. Confirma a mensagem "Usuário não encontrado". | `404 Not Found` |
| **Login de usuário com senha incorreta** | Utiliza o e-mail do admin com uma senha errada. Confirma a mensagem "Credenciais inválidas". | `401 Unauthorized` |

### 📂 Validação do token (`POST {{auth_service_url}}/validate-token`)

| Cenário | O que valida | Retorno esperado |
|---|---|---|
| **Validação com token valido** | Cria um usuário temporário, faz login (`performLogin`) e valida o token JWT retornado. Confirma `message` = "Token válido", a presença do objeto `decoded` com `userId`, `userEmail`, `roleName`, `iat` e `exp`, e que `roleName` é `"User"`. Remove o usuário ao final. | `200 OK` |
| **Validação com token admin valido** | Faz login como administrador (`performLogin`) e valida o token gerado. Confirma `message` = "Token válido", os mesmos campos decodificados e que `roleName` é `"Admin"`. | `200 OK` |
| **Validação sem fornecer token** | Envia a requisição sem token no header. Confirma a mensagem "Token não fornecido". | `400 Bad Request` |
| **Validação com token invalido** | Envia um token malformado/inexistente. Confirma a mensagem "Token inválido ou não fornecido". | `401 Unauthorized` |

---

## 🛠️ Funções auxiliares (scripts de collection)

Essas funções são definidas no **pre-request script da collection** e ficam disponíveis para todos os requests via variáveis de collection (padrão usado para compartilhar funções entre requests no Postman):

- **`createSimpleUserRequest(attempt)`** — cria um usuário comum no `user_service_url` com dados aleatórios (`{{$randomWord}}` / `{{$randomEmail}}`), com até 5 tentativas em caso de falha. Em modo mock, apenas simula a criação preenchendo as variáveis com valores fixos. Ao ter sucesso, preenche `userRecentCadastreId`, `userRecentCadastreEmail`, `userRecentCadastrePassword` e `userRecentCadastreToken`.
- **`deleteSimpleUserRequest(token, userId)`** — remove o usuário temporário criado para o teste, usando o token e o ID informados, e limpa as variáveis de collection correspondentes. Em modo mock, não faz nenhuma chamada.
- **`performLogin(email, password, scenario, tokenVarName)`** — realiza login no AuthService e salva o token retornado na variável de collection indicada (`tokenVarName`, padrão `adminToken`). Em modo mock, envia o header `x-mock-scenario` para direcionar a resposta simulada correta.

---

## 🌎 Ambientes disponíveis

| Ambiente | Arquivo | `useMock` | Uso recomendado |
|---|---|---|---|
| Local Mock | `local-mock.environment.json` | `true` | Rodar os testes localmente sem depender do UserService real |
| Local Real Integration | `local-real.environment.json` | `false` | Rodar os testes localmente contra os serviços reais em execução na máquina |
| CI Environment | `ci.environment.json` | `true` | Execução automatizada em pipelines de integração contínua |
| Production Integration | `production.environment.json` | `false` | Testes contra o ambiente de produção (usar com cautela) |