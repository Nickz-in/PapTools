---
description: Bugs encontrados na página de Login / Recuperação de Senha
icon: user
---

# Login

## Recuperação de Senha

Ao fazer a recuperação de senha o campo está aceitando qualquer informação sem antes fazer um check se o e-mail é válido.\


<div><figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure> <figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure></div>

\
• A adição de um checkMail, se faz necessária para evitar e-mails inexistentes ou qualquer informação no campo. \
• Uma ideia de resolução, é fazer uma checagem no banco de dados primeiro se existe o e-mail enviado, caso não exista, retornar uma mensagem de erro logo abaixo informando: "E-mail inválido, tente novamente."\
\
Exemplo se o e-mail for inválido:\
![](<../.gitbook/assets/image (6).png>)\
\
Ao inserir um e-mail "válido", porém não cadastrado no banco:\
![](<../.gitbook/assets/image (7).png>)\
\
\
\
\
\
