---
description: Validação de Email para Recuperação de Senha - Documentação Completa
---

# Validação

## Como funcionava?

Antes da implementação, o fluxo de recuperação de senha tinha uma **falha de segurança crítica**:

```
1. Usuário acessa "Recuperar senha"
2. Digita qualquer texto no campo de email (ex: "abcdef", "teste@inexistente.com")
3. Sistema aceita SEM VALIDAÇÃO
4. Usuário é levado para tela de código
5. Obviamente, nenhum código chega
6. Talvez possa chegar código de validação em e-mails válidos, por não haver checagem.
```

**Problemas identificados:**

* ❌ Nenhuma validação de formato de e-mail;
* ❌ Nenhuma verificação se o e-mail existe no banco;
* Sem validação, qualquer e-mail válido, (talvez) receberia o código de recuperação.&#x20;



### ✅ Sugestão de correção:

```
1. Usuário acessa "Recuperar senha"
2. Digita o e-mail
3. Sistema faz validações:
   a) Formato do e-mail está correto? se sim, próximo
   b) Email existe no banco de dados?
4. Se NÃO: avisa que não foi encontrado, contate suporte. Aguardar e-mail válido.
5. Se SIM: envia código e avança para próxima tela.
```

**Benefícios esperados:**

* Validação de formato com RegEx;
* Verificação em tempo real no banco;
* Segurança, apenas usuários cadastrados podem solicitar;
* Feedback claro e específico;
* Melhor experiência do usuário.

\
\
\
\
\
