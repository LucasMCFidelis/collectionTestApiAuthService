# 🔐 collectionTestApiAuthService

Collection do Postman para testes de integração (end-to-end) do **AuthService** do projeto *Catálogo de Eventos*. A collection cobre os fluxos de **login** e **validação de token JWT**, incluindo cenários de sucesso, erro de validação e erros de autenticação/autorização.

---

## ▶️ Como executar

Essa collection é usada de duas formas: **automaticamente**, dentro do `docker compose` do serviço testado, ou **manualmente**, via Postman/Newman, para depurar um cenário específico. Veja a seção [🌎 Ambientes disponíveis](#-ambientes-disponíveis) para saber qual environment usar em cada caso.

Para rodar manualmente — seja pelo Postman, seja pelo Newman — primeiro suba o ambiente de teste do AuthService via `docker compose`, no repositório do serviço, [`auth-service-eventsCatalog-`](https://github.com/LucasMCFidelis/auth-service-eventsCatalog-) — é lá que estão as instruções detalhadas de setup, profiles e variáveis de ambiente:

```bash
docker compose --profile test up --build -d
```

Isso builda este repositório internamente e já roda a collection automaticamente (contra `ci.environment.json`), mas mantém o AuthService e o mock do UserService publicados nas portas padrão do host (`8082`/`8081`) enquanto os containers estiverem de pé — é contra essas portas que o environment `local` aponta, usado abaixo em ambas as opções. Ao terminar, encerre o ambiente com `docker compose --profile test down -v` no repositório do AuthService.
 
Clone este repositório também — é dele que vêm a collection e os environments usados nas duas opções abaixo:
 
```bash
git clone https://github.com/LucasMCFidelis/collectionTestApiAuthService.git
cd collectionTestApiAuthService
```
 
### Opção 1 — Postman (interface gráfica)
1. Importe a collection em `postman/collections/auth-service.postman_collection.json`.
2. Importe o(s) environment(s) desejado(s) em `postman/environments/` (`local.environment.json` e/ou `ci.environment.json`).
3. Selecione o environment no canto superior direito do Postman.
4. Preencha as variáveis obrigatórias (ver seção [Variáveis](#️-variáveis-necessárias-para-executar-a-collection) abaixo) — os dois environments já vêm preenchidos por padrão, só ajuste se necessário.
5. Execute a collection inteira via **Runner**, ou cada request individualmente.
### Opção 2 — Newman (linha de comando)
 
Com o `docker compose` já rodando e o repositório clonado, aponte o Newman direto para o environment `local`, sem precisar sobrescrever nenhuma URL:
 
```bash
npm install -g newman
 
newman run postman/collections/auth-service.postman_collection.json \
  -e postman/environments/local.environment.json
```

---

## 🌎 Ambientes disponíveis

Só existem **dois** arquivos de environment neste repositório — mantidos deliberadamente enxutos, um para cada forma de execução:

| Ambiente | Quando usar |
|---|---|
| **Local** | Rodar a collection manualmente (Postman ou Newman), na sua máquina, contra o ambiente de teste do AuthService já rodando localmente via `docker compose` (porta padrão do host — ver seção abaixo). Ideal para depurar um cenário específico sem esperar o CI. |
| **CI** | Uso interno, automático: é o environment que o próprio `docker compose` do AuthService injeta no container de testes. Os hostnames (`auth-service`, `user-service`) são os *aliases* de rede dos serviços dentro do Compose, não `localhost` — porta interna sempre `8080`. Normalmente você não precisa selecionar esse environment manualmente. |

Os dois já vêm com `useMock="true"` e `adminEmail`/`adminPassword` preenchidos com credenciais fixas de mock — não é preciso configurar nada para rodar contra o ambiente de teste (mockado) do AuthService, seja localmente, seja no CI.

---

## ⚙️ Variáveis necessárias para executar a Collection

Para que esta collection funcione corretamente no Postman, configure as seguintes variáveis no **Environment**:

---

### 🌐 URLs dos serviços obrigatórios

| Variável            | Descrição                                     |
|---------------------|------------------------------------------------|
| `auth_service_url`  | URL base do AuthService (porta padrão local: `http://localhost:8082/auth`) |
| `user_service_url`  | URL base do UserService, usado para criar/remover o usuário de teste (porta padrão local: `http://localhost:8081/users`) |

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
 
## 🧩 Mock do AuthService (WireMock)
 
Assim como o `collectionTestApiUserService` mantém mappings que simulam o UserService (reaproveitados pelo `auth-service` e pelo `email-service`), este repositório mantém os mappings que simulam o **AuthService**, para serem consumidos por collections de outros serviços que dependem de autenticação/validação de token, sem precisar subir o AuthService real.
 
| Arquivo | Endpoint simulado | `X-Mock-Scenario` | Resposta |
|---|---|---|---|
| `01-validate-token-user.json` | `POST /auth/validate-token` | `SUCCESS_VALIDATE_TOKEN_USER` (ou `SUCCESS_VALIDATE_TOKEN_USER_<id>` para fixar o `userId` decodificado) | `200` — "Token válido", `decoded.roleName` = `"User"` |
| `02-validate-token-admin.json` | `POST /auth/validate-token` | `SUCCESS_VALIDATE_TOKEN_ADMIN` | `200` — "Token válido", `decoded.roleName` = `"Admin"` |
| `03-invalidate-token.json` | `POST /auth/validate-token` | `FAIL_VALIDATE_TOKEN` | `401` — "Token inválido ou não fornecido" |
| `04-sucess-login-user.json` | `POST /auth/login` | `SUCCESS_LOGIN_USER` | `200` — retorna `message` de sucesso e um `userToken` (JWT) fixo |
| `00-fallback.json` | `ANY /auth/*` | *(qualquer cenário não mapeado acima)* | `500` — "Scenario not defined" |
 
 
### Subindo o mock isoladamente
 
Use a imagem buildada a partir do `docker/mock.Dockerfile` deste repositório — os mappings já ficam embutidos na imagem (`COPY wiremock /home/wiremock`), sem precisar de bind-mount:
 
```bash
docker build -f docker/mock.Dockerfile -t auth-service-mock .
 
docker run -d --name wiremock-auth-service -p 8082:8080 auth-service-mock
```
 
Isso expõe o mock em `http://localhost:8082` (porta padrão utilizada no projeto para o AuthService, já configurada no environment `local`). Depois, basta apontar a variável de URL do AuthService do serviço/collection sendo testado para esse endereço e enviar o header `X-Mock-Scenario` desejado.