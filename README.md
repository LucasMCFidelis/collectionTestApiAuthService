# ⚙️ Variáveis necessárias para executar a Collection

Para que esta collection funcione corretamente no Postman, configure as seguintes variáveis no **Environment**:

---

## 🌐 URLs dos serviços obrigatórios

| Variável            | Descrição                                  |
|---------------------|----------------------------------------------|
| `auth_service_url`  | URL base do AuthService                      |
| `user_service_url`  | URL base do UserService (criação de usuários) |

---

## 👤 Credenciais do Admin

Essas variáveis permitem que a collection teste o login como administrador.

| Variável        | Descrição                 |
|-----------------|---------------------------|
| `adminEmail`    | Email do administrador    |
| `adminPassword` | Senha do administrador    |

---

## 🧪 Usuário temporário (preenchido automaticamente)

Essas variáveis são criadas e atualizadas automaticamente pelos scripts da collection.

| Variável                    | Preenchida automaticamente | Descrição                                |
|-----------------------------|----------------------------|--------------------------------------------|
| `userRecentCadastreId`      | ✔️                         | ID do usuário criado                       |
| `userRecentCadastreEmail`   | ✔️                         | Email do usuário criado                    |
| `userRecentCadastrePassword`| ✔️                         | Senha usada na criação                     |
| `userRecentCadastreToken`   | ✔️                         | Token JWT do usuário criado                |

---

## ✔️ Resumo rápido

### 🔧 Configure manualmente:

- `auth_service_url`
- `user_service_url`
- `adminEmail`
- `adminPassword`

---

### 🤖 Variáveis gerenciadas automaticamente pelos scripts:

- `userRecentCadastreId`
- `userRecentCadastreEmail`
- `userRecentCadastrePassword`
- `userRecentCadastreToken`
