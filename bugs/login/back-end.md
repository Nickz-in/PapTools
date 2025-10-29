# Back-end

## Ligando tudo no back

```
Método: GET
URL: /auth/verify-email
Query Parameters: email (string, obrigatório)
Headers: 
  - Content-Type: application/json
  - Accept: application/json
```

#### Request

```
GET /auth/verify-email?email=usuario@exemplo.com HTTP/1.1
Host: api.seudominio.com
Content-Type: application/json
Accept: application/json
```

#### Response Success (200 OK)

```json
{
  "exists": true
}
```

#### Response Email Não Encontrado (200 OK)

```json
{
  "exists": false
}
```

#### Response Error (500 Internal Server Error)

```json
{
  "error": "Database connection failed",
  "message": "Erro ao verificar email"
}
```

#### Regras de Negócio

1. Buscar no banco de dados se existe um usuário com este e-mail;
2. E-mail deve ser comparado de forma case-insensitive;
3. &#x20;Considerar apenas usuários ativos (não deletados/bloqueados);
4. NÃO revelar informações sensíveis (por segurança);
5. Log de todas as tentativas (para auditoria).

***

### Enviar Email de Recuperação

#### Especificação Técnica

```
Método: POST
URL: /auth/password-recovery
Headers:
  - Content-Type: application/json
  - Accept: application/json
Body: JSON
```

#### Request

```json
POST /auth/password-recovery HTTP/1.1
Host: api.seudominio.com
Content-Type: application/json
Accept: application/json

{
  "email": "usuario@exemplo.com"
}
```

#### Response Success (200 OK)

```json
{
  "success": true,
  "message": "Email de recuperação enviado com sucesso"
}
```

#### Response Error - Email Não Encontrado (404 Not Found)

```json
{
  "success": false,
  "message": "Email não encontrado",
  "code": "EMAIL_NOT_FOUND"
}
```

#### Response Error - Falha no Envio (500 Internal Server Error)

```json
{
  "success": false,
  "message": "Erro ao enviar email",
  "code": "EMAIL_SEND_FAILED"
}
```

***

⚠️ IMPORTANTE - Ação Necessária no Backend:\
Para que a funcionalidade trabalhe completamente, o backend precisa implementar 2 endpoints:

```
1. GET /auth/verify-email?email={email}
   Resposta: { "exists": true/false }

2. POST /auth/password-recovery
   Body: { "email": "usuario@email.com" }
   Resposta: 200 OK
```

## Testes e Validação

### Checklist de Testes

#### Frontend

* [ ] Campo vazio - deve mostrar erro "Por favor, informe seu email"
* [ ] Email inválido (sem @) - deve mostrar erro de formato
* [ ] Email inválido (sem domínio) - deve mostrar erro de formato
* [ ] Email válido mas não existe - deve mostrar "Email não encontrado"
* [ ] Email válido e existe - deve enviar e avançar
* [ ] Erro de rede - deve mostrar mensagem apropriada
* [ ] Loading aparece durante validação
* [ ] Borda vermelha aparece em erros
* [ ] Erro desaparece ao digitar

#### Backend

* [ ] GET /auth/verify-email retorna 200 com exists:true para email existente
* [ ] GET /auth/verify-email retorna 200 com exists:false para email inexistente
* [ ] POST /auth/password-recovery envia email com código
* [ ] Código expira após 15 minutos
* [ ] Limite de 3 tentativas por hora funciona
* [ ] Códigos antigos são invalidados
* [ ] Logs de auditoria são criados
* [ ] Case-insensitive funciona (USER@test.com = user@test.com)

### Casos de Teste

#### Teste 1: Email Válido

```
Input: "usuario@empresa.com"
Esperado: 
- Validação de formato: PASS
- Backend verifica: exists = true
- Email enviado: SUCCESS
- Resultado: Avança para próxima tela
```

#### Teste 2: Email Inválido

```
Input: "emailsemarroba"
Esperado:
- Validação de formato: FAIL
- Mensagem: "Por favor, informe um email válido"
- Resultado: NÃO avança
```

#### Teste 3: Email Não Cadastrado

```
Input: "naoexiste@test.com"
Esperado:
- Validação de formato: PASS
- Backend verifica: exists = false
- Mensagem: "Email não encontrado..."
- Resultado: NÃO avança
```

## &#x20;Mensagens de Erro

### Todas as Mensagens Implementadas

| Situação         | Mensagem                                                                                     | Quando Aparece                   |
| ---------------- | -------------------------------------------------------------------------------------------- | -------------------------------- |
| Campo vazio      | "Por favor, informe seu email"                                                               | Usuário clica enviar sem digitar |
| Formato inválido | "Por favor, informe um email válido"                                                         | Email não tem @ ou domínio       |
| Email não existe | "Email não encontrado. Verifique se o email está correto ou entre em contato com o suporte." | Backend retorna exists:false     |
| Erro ao enviar   | "Erro ao enviar email de recuperação. Tente novamente mais tarde."                           | Falha na API de envio            |
| Erro genérico    | "Erro inesperado. Tente novamente mais tarde."                                               | Exception não tratada            |

***
