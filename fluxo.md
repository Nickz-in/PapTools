# 🎨 Fluxo Visual - Recuperação de Senha

## 📱 Jornada do Usuário

```
┌─────────────────────────────────────────────────────────────────┐
│                    TELA DE LOGIN                                 │
│                                                                  │
│  ┌──────────────────────────────────────────────────┐           │
│  │  Email: [________________]                        │           │
│  │  Senha: [________________]                        │           │
│  │                                                    │           │
│  │  [         Entrar          ]                      │           │
│  │                                                    │           │
│  │  >>> Esqueci minha senha <<<  👈 USUÁRIO CLICA    │           │
│  └──────────────────────────────────────────────────┘           │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│              TELA DE RECUPERAÇÃO - PASSO 1                       │
│                                                                  │
│  Informe seu email para receber o código                        │
│                                                                  │
│  ┌──────────────────────────────────────────────────┐           │
│  │  Email: [usuario@empresa.com___]                 │           │
│  └──────────────────────────────────────────────────┘           │
│                                                                  │
│  [         Enviar          ]  👈 USUÁRIO CLICA                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    VALIDAÇÃO FRONTEND                            │
│                                                                  │
│  ✓ Campo vazio?           ────────► ❌ Erro: "Informe email"    │
│  ✓ Formato válido?        ────────► ❌ Erro: "Email inválido"   │
│  ✓ Formato OK             ────────► ✅ Continua                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    CHAMADA API #1                                │
│                                                                  │
│  GET /auth/verify-email?email=usuario@empresa.com               │
│                                                                  │
│  BACKEND VERIFICA NO BANCO:                                      │
│  SELECT * FROM usuarios WHERE email = 'usuario@empresa.com'     │
│                                                                  │
│  ┌─────────────────┐              ┌─────────────────┐           │
│  │  Existe?        │─── SIM ───►  │ {exists: true}  │           │
│  │                 │              └─────────────────┘           │
│  │                 │              ┌─────────────────┐           │
│  │                 │─── NÃO ───►  │ {exists: false} │           │
│  └─────────────────┘              └─────────────────┘           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                    ┌─────────┴──────────┐
                    │                    │
               EXISTE                NÃO EXISTE
                    │                    │
                    ▼                    ▼
        ┌──────────────────┐  ┌──────────────────────┐
        │  Continua...     │  │  MOSTRA ERRO         │
        │                  │  │  "Email não          │
        │                  │  │   encontrado"        │
        │                  │  │                      │
        │                  │  │  PARA AQUI ❌        │
        └────────┬─────────┘  └──────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────────┐
│                    CHAMADA API #2                                │
│                                                                  │
│  POST /auth/password-recovery                                    │
│  Body: { "email": "usuario@empresa.com" }                       │
│                                                                  │
│  BACKEND:                                                        │
│  1. Gera código: 123456                                          │
│  2. Salva no banco com expiração de 15 min                       │
│  3. Envia email                                                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    EMAIL ENVIADO                                 │
│                                                                  │
│  Para: usuario@empresa.com                                       │
│  Assunto: Recuperação de Senha                                   │
│                                                                  │
│  ┌────────────────────────────────────────────┐                 │
│  │ Olá, João                                  │                 │
│  │                                            │                 │
│  │ Seu código de recuperação é:               │                 │
│  │                                            │                 │
│  │          [ 1 2 3 4 5 6 ]                   │                 │
│  │                                            │                 │
│  │ Válido por 15 minutos                      │                 │
│  └────────────────────────────────────────────┘                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│              TELA DE RECUPERAÇÃO - PASSO 2                       │
│                                                                  │
│  Digite o código enviado para seu email                          │
│                                                                  │
│  ┌──────────────────────────────────────────────────┐           │
│  │  Código: [_ _ _ _ _ _]                           │           │
│  └──────────────────────────────────────────────────┘           │
│                                                                  │
│  [        Validar Código        ]                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│              TELA DE RECUPERAÇÃO - PASSO 3                       │
│                                                                  │
│  Digite sua nova senha                                           │
│                                                                  │
│  ┌──────────────────────────────────────────────────┐           │
│  │  Nova Senha: [________________]                  │           │
│  │  Confirmar:  [________________]                  │           │
│  └──────────────────────────────────────────────────┘           │
│                                                                  │
│  [     Alterar Senha     ]                                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
                    ✅ SENHA ALTERADA!
```

---

## 🔄 Diagrama de Sequência Técnico

```
┌─────────┐  ┌──────┐  ┌───────────┐  ┌──────────┐  ┌──────────┐
│ Usuário │  │  UI  │  │ ViewModel │  │Datasource│  │ Backend  │
└────┬────┘  └───┬──┘  └─────┬─────┘  └────┬─────┘  └────┬─────┘
     │           │            │             │             │
     │ Digita    │            │             │             │
     │  email    │            │             │             │
     │───────────►            │             │             │
     │           │            │             │             │
     │ Clica     │            │             │             │
     │ "Enviar"  │            │             │             │
     │───────────►            │             │             │
     │           │            │             │             │
     │           │sendPassword│             │             │
     │           │ Recover()  │             │             │
     │           │────────────►             │             │
     │           │            │             │             │
     │           │            │ 1. Valida  │             │
     │           │            │   formato  │             │
     │           │            │────────►   │             │
     │           │            │            │             │
     │           │            │ 2. verify  │             │
     │           │            │   Email()  │             │
     │           │            │────────────►             │
     │           │            │            │             │
     │           │            │            │ GET /auth   │
     │           │            │            │ /verify-    │
     │           │            │            │  email      │
     │           │            │            │─────────────►
     │           │            │            │             │
     │           │            │            │ SELECT FROM │
     │           │            │            │  usuarios   │
     │           │            │            │             │
     │           │            │            │{exists:true}│
     │           │            │            │◄─────────────
     │           │            │            │             │
     │           │            │   true     │             │
     │           │            │◄────────────             │
     │           │            │            │             │
     │           │            │ 3. send    │             │
     │           │            │  Recovery  │             │
     │           │            │  Email()   │             │
     │           │            │────────────►             │
     │           │            │            │             │
     │           │            │            │POST /auth   │
     │           │            │            │/password-   │
     │           │            │            │ recovery    │
     │           │            │            │─────────────►
     │           │            │            │             │
     │           │            │            │ 1. Gera     │
     │           │            │            │   código    │
     │           │            │            │             │
     │           │            │            │ 2. Salva no │
     │           │            │            │    banco    │
     │           │            │            │             │
     │           │            │            │ 3. Envia    │
     │           │            │            │    email    │
     │           │            │            │             │
     │           │            │            │ {success:   │
     │           │            │            │   true}     │
     │           │            │            │◄─────────────
     │           │            │            │             │
     │           │            │   true     │             │
     │           │            │◄────────────             │
     │           │            │            │             │
     │           │            │ 4. Muda    │             │
     │           │            │   estado   │             │
     │           │            │────────►   │             │
     │           │            │            │             │
     │           │ Atualiza   │            │             │
     │           │    UI      │            │             │
     │           │◄────────────            │             │
     │           │            │            │             │
     │ Mostra    │            │            │             │
     │ próxima   │            │            │             │
     │  tela     │            │            │             │
     │◄───────────            │            │             │
     │           │            │            │             │
```

---

## ⚠️ Fluxos de Erro

### Erro 1: Email Inválido (Validação Frontend)

```
Usuário digita: "emailsemarroba"
         │
         ▼
   ┌─────────────┐
   │  ViewModel  │
   │  valida     │ ───► isValidEmail() ───► FALSE
   │  formato    │
   └─────────────┘
         │
         ▼
   ┌─────────────┐
   │  errorMsg = │
   │  "Email     │
   │   inválido" │
   └─────────────┘
         │
         ▼
   ┌─────────────┐
   │  UI mostra  │
   │  erro com   │
   │  borda      │
   │  vermelha   │
   └─────────────┘
         │
         ▼
    PARA AQUI ❌
   (não faz chamada API)
```

### Erro 2: Email Não Cadastrado

```
Usuário digita: "naoexiste@test.com"
         │
         ▼
   ┌─────────────┐
   │  Formato OK │ ✓
   └─────────────┘
         │
         ▼
   ┌─────────────┐
   │GET /verify  │
   │   -email    │
   └─────────────┘
         │
         ▼
   ┌─────────────┐
   │  Backend:   │
   │  SELECT...  │
   │  COUNT = 0  │
   └─────────────┘
         │
         ▼
   ┌─────────────┐
   │  Response:  │
   │{exists:false}│
   └─────────────┘
         │
         ▼
   ┌─────────────┐
   │  errorMsg = │
   │  "Email não │
   │  encontrado"│
   └─────────────┘
         │
         ▼
   ┌─────────────┐
   │  UI mostra  │
   │  erro       │
   └─────────────┘
         │
         ▼
    PARA AQUI ❌
   (não envia email)
```

### Erro 3: Limite de Tentativas Excedido

```
Usuário já tentou 3x em 1 hora
         │
         ▼
   ┌─────────────┐
   │  Formato OK │ ✓
   └─────────────┘
         │
         ▼
   ┌─────────────┐
   │  Email      │ ✓
   │  existe     │
   └─────────────┘
         │
         ▼
   ┌─────────────┐
   │POST         │
   │/password-   │
   │ recovery    │
   └─────────────┘
         │
         ▼
   ┌─────────────┐
   │  Backend:   │
   │  Verifica   │
   │  tentativas │
   │  = 3        │
   └─────────────┘
         │
         ▼
   ┌─────────────┐
   │  Response:  │
   │  429 TOO    │
   │  MANY       │
   │  REQUESTS   │
   └─────────────┘
         │
         ▼
   ┌─────────────┐
   │  errorMsg = │
   │  "Limite    │
   │  excedido"  │
   └─────────────┘
         │
         ▼
    PARA AQUI ❌
   (aguardar 1 hora)
```

---

## 🎯 Estados da Aplicação

```
┌─────────────────────────────────────────────────┐
│         PASSWORD RECOVER STATE MACHINE          │
└─────────────────────────────────────────────────┘

     ┌──────────────────────────────────┐
     │ SendingPasswordRecoverRequest    │ ◄── Estado Inicial
     │                                  │
     │ - Campo de email                 │
     │ - Validação                      │
     └──────────────┬───────────────────┘
                    │
         ┌──────────┴──────────┐
         │                     │
      SUCESSO               ERRO
         │                     │
         ▼                     ▼
┌─────────────────┐    ┌─────────────┐
│SendingPassword  │    │ Permanece   │
│RecoverCodeState │    │ no mesmo    │
│                 │    │ estado com  │
│ - Campo código  │    │ errorMsg    │
│ - Validação     │    └─────────────┘
└────────┬────────┘
         │
      SUCESSO
         │
         ▼
┌─────────────────┐
│CreatingNewPass  │
│wordState        │
│                 │
│ - Nova senha    │
│ - Confirmação   │
└─────────────────┘
```

---

## 📊 Tabelas no Banco

### Estrutura de Dados

```
┌─────────────────────────────────────────────────┐
│             TABELA: usuarios                    │
├──────────────┬───────────────┬──────────────────┤
│ id           │ INTEGER       │ PRIMARY KEY      │
│ nome         │ VARCHAR(255)  │                  │
│ email        │ VARCHAR(255)  │ UNIQUE, INDEXED  │
│ senha_hash   │ VARCHAR(255)  │                  │
│ ativo        │ BOOLEAN       │ DEFAULT true     │
│ deletado     │ BOOLEAN       │ DEFAULT false    │
│ created_at   │ TIMESTAMP     │                  │
└──────────────┴───────────────┴──────────────────┘
              ▲
              │ (FK)
              │
┌─────────────┴───────────────────────────────────┐
│       TABELA: password_reset_codes              │
├──────────────┬───────────────┬──────────────────┤
│ id           │ SERIAL        │ PRIMARY KEY      │
│ user_id      │ INTEGER       │ FK -> usuarios   │
│ codigo       │ VARCHAR(6)    │ "123456"         │
│ expira_em    │ TIMESTAMP     │ NOW() + 15min    │
│ ativo        │ BOOLEAN       │ DEFAULT true     │
│ created_at   │ TIMESTAMP     │ NOW()            │
│ used_at      │ TIMESTAMP     │ NULL             │
│ ip_address   │ VARCHAR(45)   │                  │
└──────────────┴───────────────┴──────────────────┘

┌─────────────────────────────────────────────────┐
│   TABELA: auditoria_recuperacao_senha           │
├──────────────┬───────────────┬──────────────────┤
│ id           │ SERIAL        │ PRIMARY KEY      │
│ acao         │ VARCHAR(50)   │ "VERIFICACAO_..." │
│ user_id      │ INTEGER       │ FK -> usuarios   │
│ email        │ VARCHAR(255)  │                  │
│ ip_address   │ VARCHAR(45)   │                  │
│ success      │ BOOLEAN       │                  │
│ created_at   │ TIMESTAMP     │ NOW()            │
└──────────────┴───────────────┴──────────────────┘
```

### Exemplo de Dados

```
password_reset_codes:
┌────┬─────────┬────────┬─────────────────────┬───────┬─────────────────────┐
│ id │ user_id │ codigo │ expira_em           │ ativo │ created_at          │
├────┼─────────┼────────┼─────────────────────┼───────┼─────────────────────┤
│  1 │   123   │ 456789 │ 2025-10-29 14:45:00 │ true  │ 2025-10-29 14:30:00 │
│  2 │   123   │ 123456 │ 2025-10-29 15:00:00 │ false │ 2025-10-29 14:45:00 │◄─ Invalidado
│  3 │   123   │ 789012 │ 2025-10-29 15:15:00 │ true  │ 2025-10-29 15:00:00 │◄─ Ativo atual
└────┴─────────┴────────┴─────────────────────┴───────┴─────────────────────┘
```

---

## 🔒 Camadas de Segurança

```
┌─────────────────────────────────────────────────────────────┐
│                    SEGURANÇA EM CAMADAS                      │
└─────────────────────────────────────────────────────────────┘

Camada 1: VALIDAÇÃO FRONTEND
├─ Formato de email (RegEx)
├─ Campo não vazio
└─ Feedback imediato ao usuário
         │
         ▼
Camada 2: VERIFICAÇÃO NO BANCO
├─ Email existe?
├─ Usuário ativo?
└─ Não revela informações sensíveis
         │
         ▼
Camada 3: RATE LIMITING
├─ Máximo 3 tentativas por hora
├─ Proteção contra spam
└─ Proteção contra força bruta
         │
         ▼
Camada 4: CÓDIGO TEMPORÁRIO
├─ Expira em 15 minutos
├─ Códigos antigos invalidados
└─ Uso único
         │
         ▼
Camada 5: AUDITORIA
├─ Log de todas tentativas
├─ IP tracking
└─ Análise de padrões suspeitos
```

---

## 📝 Resumo Final

### O que o Frontend faz:
✅ Valida formato do email  
✅ Verifica se email existe (via API)  
✅ Envia solicitação de recuperação (via API)  
✅ Mostra feedback claro ao usuário  
✅ Gerencia estados da aplicação  

### O que o Backend precisa fazer:
⏳ Endpoint GET /auth/verify-email  
⏳ Endpoint POST /auth/password-recovery  
⏳ Tabela password_reset_codes  
⏳ Sistema de envio de email  
⏳ Rate limiting (3 tentativas/hora)  
⏳ Logs de auditoria  

---

**Documentação criada em:** 29/10/2025  
**Versão:** 1.0

