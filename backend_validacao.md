3. **Query Optimization**:
   ```sql
   -- Ao invés de COUNT(*)
   SELECT EXISTS(
     SELECT 1 FROM usuarios 
     WHERE LOWER(email) = LOWER($1) 
     LIMIT 1
   );
   ```

---

## 🚀 Deploy

### Checklist de Deploy

- [ ] Endpoints criados e testados
- [ ] Tabelas criadas no banco
- [ ] Índices criados
- [ ] Serviço de email configurado
- [ ] Variáveis de ambiente configuradas
- [ ] Rate limiting testado
- [ ] Logs configurados
- [ ] Testes de integração passando
- [ ] Documentação da API atualizada
- [ ] Time de frontend notificado

---

## 📞 Contato

**Dúvidas sobre a implementação?**
- Frontend Developer: Nicholas Souza
- Documentação completa: `VALIDACAO_EMAIL_RECUPERACAO_SENHA.md`

---

**Data**: 29/10/2025  
**Prioridade**: Alta 🔴  
**Estimativa**: 4-8 horas de desenvolvimento  
**Status**: Aguardando implementação
# 🔧 Requisitos Backend - Recuperação de Senha

> **Resumo Executivo**: O frontend implementou validação de email para recuperação de senha. Para funcionar completamente, o backend precisa implementar 2 novos endpoints.

---

## 🎯 Objetivo

Permitir que usuários recuperem sua senha apenas se o email estiver cadastrado no sistema, aumentando a segurança e evitando spam.

---

## 📋 Endpoints Necessários

### 1. Verificar se Email Existe

```http
GET /auth/verify-email?email={email}
```

**Descrição**: Verifica se um email está cadastrado no banco de dados

**Query Parameters**:
- `email` (string, obrigatório) - Email a ser verificado

**Headers**:
```
Content-Type: application/json
Accept: application/json
```

**Response Success (200)**:
```json
{
  "exists": true
}
```

**Response Email Não Encontrado (200)**:
```json
{
  "exists": false
}
```

**Regras de Negócio**:
- ✅ Buscar email de forma **case-insensitive** (user@test.com = USER@test.com)
- ✅ Considerar apenas usuários **ativos** (não deletados/bloqueados)
- ✅ Registrar log de auditoria
- ❌ **NÃO retornar dados do usuário** (segurança)

**SQL Exemplo**:
```sql
SELECT COUNT(*) as count 
FROM usuarios 
WHERE LOWER(email) = LOWER('usuario@email.com') 
  AND ativo = true 
  AND deletado = false;
```

---

### 2. Enviar Email de Recuperação

```http
POST /auth/password-recovery
```

**Descrição**: Gera código de recuperação e envia por email

**Headers**:
```
Content-Type: application/json
Accept: application/json
```

**Request Body**:
```json
{
  "email": "usuario@empresa.com"
}
```

**Response Success (200)**:
```json
{
  "success": true,
  "message": "Email de recuperação enviado com sucesso"
}
```

**Response Email Não Encontrado (404)**:
```json
{
  "success": false,
  "message": "Email não encontrado",
  "code": "EMAIL_NOT_FOUND"
}
```

**Response Limite Excedido (429)**:
```json
{
  "success": false,
  "message": "Limite de tentativas excedido. Tente novamente em 1 hora.",
  "code": "TOO_MANY_REQUESTS"
}
```

**Regras de Negócio**:
1. ✅ Gerar código aleatório de **6 dígitos**
2. ✅ Código expira em **15 minutos**
3. ✅ **Invalidar códigos anteriores** do mesmo usuário
4. ✅ Limite de **3 tentativas por hora** por usuário
5. ✅ Enviar email com template formatado
6. ✅ Registrar log de auditoria

---

## 🗄️ Banco de Dados

### Tabela: password_reset_codes

```sql
CREATE TABLE password_reset_codes (
  id SERIAL PRIMARY KEY,
  user_id INTEGER NOT NULL REFERENCES usuarios(id),
  codigo VARCHAR(6) NOT NULL,
  expira_em TIMESTAMP NOT NULL,
  ativo BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT NOW(),
  used_at TIMESTAMP NULL,
  ip_address VARCHAR(45) NULL,
  
  INDEX idx_user_id (user_id),
  INDEX idx_codigo (codigo),
  INDEX idx_ativo_expira (ativo, expira_em)
);
```

### Tabela: auditoria_recuperacao_senha (opcional mas recomendado)

```sql
CREATE TABLE auditoria_recuperacao_senha (
  id SERIAL PRIMARY KEY,
  acao VARCHAR(50) NOT NULL,
  user_id INTEGER NULL REFERENCES usuarios(id),
  email VARCHAR(255) NOT NULL,
  ip_address VARCHAR(45) NULL,
  success BOOLEAN DEFAULT false,
  error_message TEXT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  
  INDEX idx_email_created (email, created_at),
  INDEX idx_created_at (created_at)
);
```

---

## 💻 Exemplo de Implementação (Node.js)

### Endpoint 1: Verificar Email

```javascript
router.get('/auth/verify-email', async (req, res) => {
  const { email } = req.query;
  
  if (!email) {
    return res.status(400).json({ error: 'Email é obrigatório' });
  }
  
  const result = await db.query(
    `SELECT COUNT(*) as count 
     FROM usuarios 
     WHERE LOWER(email) = LOWER($1) 
       AND ativo = true 
       AND deletado = false`,
    [email]
  );
  
  const exists = result.rows[0].count > 0;
  
  // Log de auditoria
  await db.query(
    `INSERT INTO auditoria_recuperacao_senha 
     (acao, email, ip_address, success) 
     VALUES ($1, $2, $3, $4)`,
    ['VERIFICACAO_EMAIL', email, req.ip, true]
  );
  
  return res.json({ exists });
});
```

### Endpoint 2: Enviar Email

```javascript
router.post('/auth/password-recovery', async (req, res) => {
  const { email } = req.body;
  
  // 1. Buscar usuário
  const user = await db.query(
    `SELECT id, nome, email 
     FROM usuarios 
     WHERE LOWER(email) = LOWER($1) 
       AND ativo = true`,
    [email]
  );
  
  if (user.rows.length === 0) {
    return res.status(404).json({ 
      success: false, 
      code: 'EMAIL_NOT_FOUND' 
    });
  }
  
  const usuario = user.rows[0];
  
  // 2. Verificar limite (3 por hora)
  const tentativas = await db.query(
    `SELECT COUNT(*) as count 
     FROM password_reset_codes 
     WHERE user_id = $1 
       AND created_at > NOW() - INTERVAL '1 hour'`,
    [usuario.id]
  );
  
  if (tentativas.rows[0].count >= 3) {
    return res.status(429).json({ 
      success: false, 
      code: 'TOO_MANY_REQUESTS' 
    });
  }
  
  // 3. Invalidar códigos anteriores
  await db.query(
    'UPDATE password_reset_codes SET ativo = false WHERE user_id = $1',
    [usuario.id]
  );
  
  // 4. Gerar código de 6 dígitos
  const codigo = Math.floor(100000 + Math.random() * 900000).toString();
  
  // 5. Salvar código (expira em 15 min)
  await db.query(
    `INSERT INTO password_reset_codes 
     (user_id, codigo, expira_em, ip_address) 
     VALUES ($1, $2, NOW() + INTERVAL '15 minutes', $3)`,
    [usuario.id, codigo, req.ip]
  );
  
  // 6. Enviar email
  await emailService.send({
    to: usuario.email,
    subject: 'Recuperação de Senha',
    template: 'password-recovery',
    data: {
      nome: usuario.nome,
      codigo: codigo,
      validade: '15 minutos'
    }
  });
  
  // 7. Log
  await db.query(
    `INSERT INTO auditoria_recuperacao_senha 
     (acao, user_id, email, ip_address, success) 
     VALUES ($1, $2, $3, $4, $5)`,
    ['RECUPERACAO_SOLICITADA', usuario.id, email, req.ip, true]
  );
  
  return res.json({ success: true });
});
```

---

## 📧 Template de Email

### Sugestão de Template HTML

```html
<!DOCTYPE html>
<html>
<head>
  <style>
    body { font-family: Arial, sans-serif; }
    .container { max-width: 600px; margin: 0 auto; padding: 20px; }
    .header { background: #FFC722; padding: 20px; text-align: center; }
    .code { 
      font-size: 32px; 
      font-weight: bold; 
      color: #333; 
      letter-spacing: 5px;
      background: #f5f5f5;
      padding: 20px;
      text-align: center;
      margin: 20px 0;
    }
    .warning { color: #d32f2f; }
  </style>
</head>
<body>
  <div class="container">
    <div class="header">
      <h1>PapTools - Recuperação de Senha</h1>
    </div>
    
    <p>Olá, {{nome}}</p>
    
    <p>Recebemos uma solicitação para recuperar sua senha.</p>
    
    <p>Use o código abaixo no aplicativo:</p>
    
    <div class="code">{{codigo}}</div>
    
    <p class="warning">⚠️ Este código expira em {{validade}}</p>
    
    <p>Se você não solicitou esta recuperação, ignore este email.</p>
    
    <hr>
    <p style="color: #666; font-size: 12px;">
      Este é um email automático, não responda.
    </p>
  </div>
</body>
</html>
```

---

## 🧪 Testes

### Casos de Teste Obrigatórios

#### Endpoint 1: /auth/verify-email

| Teste | Input | Esperado |
|-------|-------|----------|
| Email existe | user@test.com | `{exists: true}` |
| Email não existe | nao@existe.com | `{exists: false}` |
| Case insensitive | USER@test.com | `{exists: true}` |
| Sem parâmetro | (vazio) | 400 Bad Request |
| Usuário inativo | inativo@test.com | `{exists: false}` |

#### Endpoint 2: /auth/password-recovery

| Teste | Cenário | Esperado |
|-------|---------|----------|
| Email válido | Primeira tentativa | 200 + email enviado |
| Email não existe | Email não cadastrado | 404 EMAIL_NOT_FOUND |
| Limite excedido | 4ª tentativa em 1h | 429 TOO_MANY_REQUESTS |
| Código expira | Após 15 minutos | Código inválido |
| Códigos antigos | Novo código gerado | Códigos antigos inativos |

---

## 🔒 Segurança

### Checklist de Segurança

- [ ] Emails armazenados com hash (ou criptografados)
- [ ] Códigos nunca retornados em logs
- [ ] Rate limiting implementado (3 tentativas/hora)
- [ ] Auditoria de todas as tentativas
- [ ] Proteção contra SQL Injection (usar prepared statements)
- [ ] Validação de email no backend também
- [ ] HTTPS obrigatório
- [ ] CORS configurado corretamente
- [ ] Timeout de 15 minutos para códigos

---

## 📊 Monitoramento

### Logs Importantes

```javascript
// Log toda tentativa de verificação
logger.info('Email verification', { 
  email, 
  exists, 
  ip: req.ip, 
  timestamp 
});

// Log todo envio de código
logger.info('Password recovery requested', { 
  userId, 
  email, 
  ip: req.ip, 
  timestamp 
});

// Log códigos expirados/inválidos
logger.warn('Expired code used', { 
  userId, 
  code, 
  expired_at 
});

// Log limite excedido
logger.warn('Rate limit exceeded', { 
  userId, 
  email, 
  attempts 
});
```

### Métricas Recomendadas

- Total de verificações de email por dia
- Total de recuperações solicitadas por dia
- Taxa de sucesso vs falha
- Emails mais verificados (detectar ataques)
- Tempo médio de resposta dos endpoints

---

## ⚡ Performance

### Otimizações

1. **Índices no banco**:
   - `usuarios.email` (LOWER(email))
   - `password_reset_codes.user_id`
   - `password_reset_codes.ativo, expira_em`

2. **Cache**:
   - Cachear emails inexistentes por 5 minutos (evitar queries repetidas)
   - Usar Redis para rate limiting


