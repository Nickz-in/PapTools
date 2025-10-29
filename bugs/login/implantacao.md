---
description: >-
  Uma forma encontrada para aplicar a solução, passível a modificações conforme
  a necessidade.
---

# Implantação

1.  ### **AuthDatasource**

    _Arquivo_**:** `lib/data/datasources/auth_datasource.dart`&#x20;

    #### O que foi adicionado:

    ```dart
    // Verifica se o email existe no banco de dados
    Future<bool> verifyEmailExists(String email) async {
      try {
        final response = await http.get(
          Uri.parse('${Environment.apiUrl}/auth/verify-email?email=$email'),
          headers: {
            'Content-Type': 'application/json',
            'Accept': 'application/json',
          },
        );

        if (response.statusCode == 200) {
          final result = json.decode(response.body);
          return result['exists'] ?? false;
        } else {
          SecureLogger.error('Erro ao verificar email: ${response.body}');
          return false;
        }
      } catch (e) {
        SecureLogger.error('Erro ao verificar email: $e');
        return false;
      }
    }

    // Envia solicitação de recuperação de senha
    Future<bool> sendPasswordRecoveryEmail(String email) async {
      try {
        final response = await http.post(
          Uri.parse('${Environment.apiUrl}/auth/password-recovery'),
          headers: {
            'Content-Type': 'application/json',
            'Accept': 'application/json',
          },
          body: json.encode({'email': email}),
        );

        if (response.statusCode == 200 || response.statusCode == 201) {
          return true;
        } else {
          SecureLogger.error('Erro ao enviar email: ${response.body}');
          return false;
        }
      } catch (e) {
    ```



    **Explicação:**

    * **`verifyEmailExists`**: Faz uma chamada GET para o backend verificar se o e-mail existe;
    * **`sendPasswordRecoveryEmail`**: Faz uma chamada POST para o backend enviar o código por e-mail.

***

2. ### PasswordRecoverViewModel

**Arquivo:** `lib/ui/modules/auth/viewmodels/password_recover_viewmodel.dart`

#### O que foi modificado/adicionado:

```dart
class PasswordRecoverViewModel extends ChangeNotifier {
  // ========== NOVOS ATRIBUTOS ==========
  final AuthDatasource _authDatasource = AuthDatasource.instance;
  String? errorMessage;  // Armazena mensagem de erro
  
  // ========== NOVOS MÉTODOS ==========
  
  /// Valida se o email tem formato correto
  bool isValidEmail(String email) {
    final emailRegex = RegExp(
      r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$',
    );
    return emailRegex.hasMatch(email);
  }
  
  /// Limpa a mensagem de erro
  void clearErrorMessage() {
    errorMessage = null;
    notifyListeners();
  }
  
  // ========== MÉTODO ATUALIZADO ==========
  
  Future<void> sendPasswordRecoverRequest() async {
    final email = emailController.text.trim();

    // PASSO 1: Limpar erro anterior
    errorMessage = null;
    notifyListeners();

    // PASSO 2: Validar campo vazio
    if (email.isEmpty) {
      errorMessage = 'Por favor, informe seu email';
      notifyListeners();
      return;
    }

    // PASSO 3: Validar formato
    if (!isValidEmail(email)) {
      errorMessage = 'Por favor, informe um email válido';
      notifyListeners();
      return;
    }

    // PASSO 4: Mostrar loading
    isSendingEmail = true;
    notifyListeners();

    try {
      // PASSO 5: Verificar se email existe no banco
      final emailExists = await _authDatasource.verifyEmailExists(email);

      if (!emailExists) {
        errorMessage = 'Email não encontrado. Verifique se o email está correto ou entre em contato com o suporte.';
        isSendingEmail = false;
        notifyListeners();
        return;
      }

      // PASSO 6: Enviar email de recuperação
      final emailSent = await _authDatasource.sendPasswordRecoveryEmail(email);

      if (!emailSent) {
        errorMessage = 'Erro ao enviar email de recuperação. Tente novamente mais tarde.';
        isSendingEmail = false;
        notifyListeners();
        return;
      }

      // PASSO 7: Sucesso - avançar para próximo estado
      state = SendingPasswordRecoverCodeState();
      isSendingEmail = false;
      errorMessage = null;
      notifyListeners();
      
    } catch (e) {
      errorMessage = 'Erro inesperado. Tente novamente mais tarde.';
      isSendingEmail = false;
      notifyListeners();
    }
  }
}
```

**Explicação do Fluxo:**

1. Recebe o email digitado pelo usuário;
2. Limpa erros anteriores;
3. Valida se não está vazio;
4. Valida formato com RegEx;
5. Mostra loading;
6. Verifica no back-end se e-mail existe;
7. Se existe, envia o código por email;
8. Só avança para próxima tela se tudo der certo.

***

3. ### **SendPasswordRecoverEmailStep**

**Arquivo:** `lib/ui/modules/auth/pages/password_recover/send_password_recover_email_step.dart`

#### O que foi modificado:

```dart
TextFormField(
  enabled: true,
  controller: viewModel.emailController,  // CONECTADO AO CONTROLLER
  
  onChanged: (v) {
    // Limpar erro quando usuário digita
    if (viewModel.errorMessage != null) {
      viewModel.clearErrorMessage();
    }
  },
  
  decoration: InputDecoration(
    hintText: "Email",
    errorText: viewModel.errorMessage,  // EXIBE ERRO
    errorMaxLines: 3,  // PERMITE MENSAGENS LONGAS
    
    // Bordas normais
    border: OutlineInputBorder(
      borderSide: const BorderSide(width: 1, color: Color(0xFFCBD5E1)),
      borderRadius: BorderRadius.circular(4),
    ),
    
    // BORDAS VERMELHAS QUANDO HÁ ERRO
    errorBorder: OutlineInputBorder(
      borderSide: const BorderSide(width: 1, color: Colors.red),
      borderRadius: BorderRadius.circular(4),
    ),
    focusedErrorBorder: OutlineInputBorder(
      borderSide: const BorderSide(width: 1, color: Colors.red),
      borderRadius: BorderRadius.circular(4),
    ),
  ),
  
  keyboardType: TextInputType.emailAddress,
),
```

**Explicação:**

* **Campo conectado**: Agora o campo usa `viewModel.emailController;`
* **Feedback visual**: Borda fica vermelha quando há erro;
* **Mensagem de erro**: Aparece abaixo do campo;
* **Limpeza automática**: Erro desaparece quando usuário digita.\
  \
