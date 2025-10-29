- Confirmar status do plantão na API
- Checar se já existe plantão ativo

**5. Notificações não aparecem**
- Verificar permissão de notificação
- Confirmar FCM token válido
- Checar inscrição nos tópicos corretos

---

## 📝 Changelog

### Versão 1.0.11+18 (Atual)

**Adicionado:**
- Sistema de autenticação SSO
- Revogação de token no logout
- Detecção de dispositivos comprometidos
- Suporte a perfil PME
- Calculadora de ofertas

**Melhorado:**
- Validações de formulário
- Tratamento de erros
- Performance de carregamento
- UX de plantões

**Corrigido:**
- Crash ao fazer logout
- Duplicação de feeds
- Vazamento de memória em mapas

---

## 👥 Contribuindo

### Como Contribuir

1. Fork do projeto
2. Crie uma branch (`git checkout -b feature/AmazingFeature`)
3. Commit suas mudanças (`git commit -m 'Add some AmazingFeature'`)
4. Push para a branch (`git push origin feature/AmazingFeature`)
5. Abra um Pull Request

### Code Review

- Todo PR deve ter ao menos 1 aprovação
- Executar testes antes de merge
- Seguir padrões de código estabelecidos
- Documentar mudanças significativas

---

## 📞 Suporte

Para dúvidas e suporte técnico:
- Email: suporte@paptools.com.br
- Telefone: (XX) XXXX-XXXX
- FAQ: Disponível no menu "Suporte" do app

---

## 📄 Licença

Propriedade da Claro Brasil.
Todos os direitos reservados.

---

**Última atualização:** 29/10/2025
**Versão da documentação:** 1.0
**Autor:** Equipe PapTools

#### OAuthService
```dart
- authenticate()              // Inicia fluxo OAuth
- exchangeCode(code)          // Troca código por token
- introspectToken(token)      // Valida token
- revokeToken(token)          // Revoga token
```

**Fluxo OAuth2:**
1. App abre browser com URL de autorização
2. Usuário faz login no IdP (Identity Provider)
3. IdP redireciona de volta com código
4. App troca código por access token
5. Token é usado nas requisições API

### Revoke Token no Logout

Ao fazer logout, o token SSO é revogado:

```dart
final ssoToken = AppModel.ssoAccessToken;
if (ssoToken != null && ssoToken.isNotEmpty) {
  await oauthService.revokeToken(ssoToken);
}
```

---

## 🔔 Notificações

### Firebase Cloud Messaging

**Localização**: `lib/data/repository/notification_repository.dart`

#### Inicialização

```dart
NotificationRepository.instance.initialize()
  ├─ Solicita permissão de notificação
  ├─ Configura handlers de mensagem
  │    ├─ onMessage (app em foreground)
  │    ├─ onMessageOpenedApp (notif clicada)
  │    └─ onBackgroundMessage
  └─ Retorna FCM token
```

#### Configuração por Perfil

```dart
configureNotificationsForProfile(profileName)
  ├─ Inscreve em tópicos específicos
  │    ├─ "all_users"
  │    ├─ "pap_sales_promoter"
  │    ├─ "bcc_consultant"
  │    └─ etc.
  └─ Remove inscrições antigas
```

#### Tipos de Notificação

**Data Message:**
```json
{
  "title": "Novo comunicado",
  "body": "Confira as novidades",
  "data": {
    "type": "feed",
    "feedId": "123"
  }
}
```

**Notification Message:**
```json
{
  "notification": {
    "title": "Título",
    "body": "Mensagem"
  }
}
```

#### Local Notifications

Usado para exibir notificações quando app está em foreground:

```dart
flutter_local_notifications
  ├─ Canal Android
  ├─ Ícone e som customizados
  └─ Ação ao clicar
```

---

## 🗺️ Geolocalização

### Geolocator

**Serviços de GPS:**

```dart
LocationDatasource
  ├─ getCurrentPosition()
  │    ├─ accuracy: high
  │    ├─ timeLimit: 10s
  │    └─ Retorna Position
  │
  ├─ getLastKnownPosition()
  │    └─ Retorna última posição (cache)
  │
  ├─ checkPermission()
  │    └─ LocationPermission status
  │
  └─ requestPermission()
       └─ Solicita ao usuário
```

### Uso em Plantões

Ao fazer check-in em plantão:
1. Solicita permissão de localização
2. Obtém posição atual
3. Envia para API com dados do plantão
4. Atualiza AppModel.latitude/longitude

### Google Maps

Usado em:
- Visualização de condomínios no mapa
- Navegação para clientes na rota
- Marcadores de pontos de interesse

```dart
GoogleMap(
  markers: Set<Marker>.of(markers),
  initialCameraPosition: CameraPosition(
    target: LatLng(latitude, longitude),
    zoom: 15,
  ),
)
```

---

## 🎨 Componentes Reutilizáveis

### Widgets Globais

**Localização**: `lib/ui/widgets/`

#### PapToolsErrorMessageWidget
- Exibe mensagens de erro consistentes
- Ícone de erro
- Mensagem customizável

#### PapToolsErrorPopDialog
- Dialog de erro
- Título e mensagem
- Botão de fechar

#### StartupToolCardWidget
- Card de utilitário na home
- Ícone + Título
- onTap action
- Usado no grid de ferramentas

### Bottom Navigation Bar

**Localização**: `lib/ui/modules/home/widgets/bottom_navigation_bar_widget.dart`

Navegação entre abas:
- Ícone de Home
- Ícone de Perfil
- Indicador de aba ativa

---

## 🔧 Configurações e Ambiente

### Environment

**Localização**: `lib/utils/config/environment.dart`

```dart
enum EnvironmentType {
  development,
  staging,
  production
}

Environment.setEnvironment(EnvironmentType.development);

Environment.current
  ├─ apiBaseUrl
  ├─ isDevelopment
  ├─ isStaging
  └─ isProduction
```

### OAuth Config

```dart
enum OAuthEnvironment {
  homolog,
  production
}

OAuthConfig.setEnvironment(OAuthEnvironment.homolog);
```

### Firebase Options

**Localização**: `lib/firebase_options.dart`

Configurações geradas pelo FlutterFire CLI:
- API Keys
- Project ID
- Messaging Sender ID
- App ID

---

## 📊 Sistema de Eventos

### Event Dispatcher

**Localização**: `lib/core/event/event_dispatcher.dart`

Permite comunicação desacoplada entre módulos:

```dart
EventDispatcher.instance
  ├─ register(EventHandler)      // Registra handler
  ├─ dispatch(Event)             // Dispara evento
  └─ unregister(EventHandler)    // Remove handler
```

### Eventos de Plantão

**OrderlyInitializedEvent:**
- Disparado: Quando plantões são carregados
- Handler: Atualiza UI da lista de plantões

**OrderlyCheckedInEvent:**
- Disparado: Quando check-in é realizado
- Handler: Atualiza status do plantão, habilita ferramentas

**OrderlyCheckedOutEvent:**
- Disparado: Quando check-out é realizado
- Handler: Finaliza plantão, desabilita ferramentas

**OrderlyActivitiesInitializedEvent:**
- Disparado: Quando atividades são carregadas
- Handler: Exibe checklist

**OrderlyActivitiesUpdatedEvent:**
- Disparado: Quando atividade é marcada/desmarcada
- Handler: Atualiza progresso

**OrderlyActivitiesResetedEvent:**
- Disparado: Quando plantão é finalizado
- Handler: Limpa atividades

### Exemplo de Uso

```dart
// Registrar handler
EventDispatcher.instance.register(
  OrderlyCheckedInEventHandler()
);

// Disparar evento
EventDispatcher.instance.dispatch(
  OrderlyCheckedInEvent(orderlyId: "123")
);
```

---

## 📱 Principais Fluxos de Usuário

### Fluxo: Venda PAP (Vendedor)

```
1. Login → Home
2. Clica em "Rotas"
3. Visualiza lista de clientes
4. Seleciona cliente
5. Clica em "Realizar Venda"
6. Preenche formulário:
   - Seleciona produtos
   - Informa dados do cliente
   - Adiciona observações
7. Captura foto do documento (opcional)
8. Revisa resumo
9. Confirma venda
10. Sistema envia para API
11. Exibe confirmação
12. Retorna para lista de clientes
```

### Fluxo: Plantão BCC (Consultor)

```
1. Login → Home
2. Clica em "Plantão"
3. Visualiza plantões:
   - Agendados
   - Em andamento
   - Histórico
4. Clica em "Novo Plantão"
5. Seleciona condomínio
6. Escolhe data/hora
7. Confirma agendamento
8. No dia do plantão:
   - Clica em "Iniciar Plantão"
   - Sistema solicita localização
   - Realiza check-in
9. Durante o plantão:
   - Acessa "Ações" (checklist)
   - Marca atividades concluídas
   - Registra vendas via "Venda"
10. Ao terminar:
    - Clica em "Finalizar Plantão"
    - Confirma check-out
    - Sistema calcula métricas
11. Visualiza resumo do plantão
```

### Fluxo: Visualizar Resultados

```
1. Home → Clica em "Resultados"
2. Seleciona período:
   - Dia
   - Semana
   - Mês
3. Visualiza métricas:
   - Total de vendas
   - Metas atingidas
   - Conversão
   - Ticket médio
4. Visualiza gráficos:
   - Evolução temporal
   - Por produto
   - Por região
5. Visualiza ranking:
   - Posição atual
   - Top performers
6. Compartilha resultados (opcional)
```

### Fluxo: Negociações

```
1. Home → Clica em "Negociações"
2. Visualiza lista de propostas:
   - Em análise
   - Aprovadas
   - Rejeitadas
3. Seleciona negociação
4. Visualiza detalhes:
   - Cliente
   - Produtos
   - Valores
   - Status
   - Histórico
5. Opções disponíveis:
   - Editar (se em rascunho)
   - Cancelar
   - Visualizar documentos
6. Recebe notificação de mudança de status
```

---

## 🧪 Guia de Desenvolvimento

### Setup do Ambiente

**Pré-requisitos:**
- Flutter 3.9.0+
- Dart SDK 3.9.0+
- Android Studio / Xcode
- Firebase CLI

**Instalação:**
```bash
# Clone o repositório
git clone <repo-url>
cd hands

# Instale dependências
flutter pub get

# Configure Firebase
flutterfire configure

# Execute
flutter run
```

### Estrutura de Branches

```
main/master      → Produção
develop          → Desenvolvimento
feature/xxx      → Novas funcionalidades
bugfix/xxx       → Correções
hotfix/xxx       → Correções urgentes
```

### Padrões de Código

**Nomenclatura:**
- Classes: `PascalCase`
- Arquivos: `snake_case.dart`
- Variáveis/Funções: `camelCase`
- Constantes: `UPPER_SNAKE_CASE`

# 📱 Documentação Completa do PapTools

## 📋 Índice
1. [Visão Geral](#visão-geral)
2. [Arquitetura](#arquitetura)
3. [Funcionalidades Principais](#funcionalidades-principais)
4. [Estrutura de Módulos](#estrutura-de-módulos)
5. [Perfis de Usuário](#perfis-de-usuário)
6. [Fluxo de Autenticação](#fluxo-de-autenticação)
7. [Páginas e Componentes](#páginas-e-componentes)
8. [Repositórios e Serviços](#repositórios-e-serviços)
9. [Gerenciamento de Estado](#gerenciamento-de-estado)
10. [Segurança](#segurança)
11. [Notificações](#notificações)
12. [Guia de Desenvolvimento](#guia-de-desenvolvimento)

---

## 🎯 Visão Geral

### O que é o PapTools?

**PapTools** é um aplicativo mobile desenvolvido em Flutter para a equipe de vendas da Claro. O aplicativo oferece diferentes funcionalidades baseadas no perfil do usuário, permitindo gerenciar vendas, rotas, plantões, negociações e muito mais.

### Principais Características

- ✅ **Multi-perfil**: Suporta diferentes perfis de usuário (Vendedor PAP, Consultor BCC, Supervisor, etc.)
- ✅ **Autenticação SSO**: Sistema de login com OAuth2 e autenticação biométrica
- ✅ **Offline-first**: Funciona parcialmente sem conexão com internet
- ✅ **Geolocalização**: Rastreamento de posição para visitas e plantões
- ✅ **Notificações Push**: Firebase Cloud Messaging integrado
- ✅ **Segurança**: Detecção de dispositivos comprometidos (root/jailbreak)
- ✅ **Feeds de Comunicação**: Sistema de notícias e comunicados
- ✅ **Dashboard**: Resultados, metas e rankings em tempo real

### Tecnologias Utilizadas

| Tecnologia | Versão | Descrição |
|------------|--------|-----------|
| Flutter | 3.9.0+ | Framework principal |
| Dart | 3.9.0+ | Linguagem de programação |
| Firebase | - | Messaging, Crashlytics, Analytics |
| OAuth2 | - | Autenticação SSO |
| SQLite | - | Banco de dados local |
| Google Maps | - | Mapas e geolocalização |

---

## 🏗️ Arquitetura

O projeto segue uma **arquitetura em camadas** baseada em Clean Architecture:

```
lib/
├── core/              # Componentes centrais
│   ├── event/         # Sistema de eventos
│   ├── gps/           # Serviços de GPS
│   ├── http/          # Cliente HTTP
│   ├── oauth2/        # Autenticação OAuth2
│   ├── profile/       # Perfis de acesso
│   ├── security/      # Segurança e validações
│   └── types/         # Tipos e enums
│
├── data/              # Camada de dados
│   ├── datasources/   # Fontes de dados (API, Local)
│   ├── models/        # Modelos de dados
│   ├── repository/    # Repositórios
│   └── services/      # Serviços auxiliares
│
├── ui/                # Camada de apresentação
│   ├── modules/       # Módulos da aplicação
│   ├── orquestrators/ # Orquestradores de fluxo
│   ├── security/      # Telas de segurança
│   └── widgets/       # Componentes reutilizáveis
│
└── utils/             # Utilitários
    ├── config/        # Configurações
    └── extensions/    # Extensions Dart
```

### Fluxo de Dados

```
┌─────────────┐
│    Widget   │
└──────┬──────┘
       │
       ↓
┌─────────────┐
│  ViewModel  │ (Gerencia estado e lógica de apresentação)
└──────┬──────┘
       │
       ↓
┌─────────────┐
│ Repository  │ (Orquestra datasources)
└──────┬──────┘
       │
       ↓
┌─────────────┐
│ Datasource  │ (Comunica com API/DB)
└──────┬──────┘
       │
       ↓
┌─────────────┐
│  API/DB     │
└─────────────┘
```

---

## 🚀 Funcionalidades Principais

### 1. Autenticação e Segurança

#### Login
- Login com email e senha
- Autenticação SSO (Single Sign-On) com OAuth2
- Autenticação biométrica (fingerprint/face)
- Verificação de permissões de acesso
- Armazenamento seguro de credenciais

#### Segurança do Dispositivo
- Detecção de root (Android)
- Detecção de jailbreak (iOS)
- Verificação de integridade do dispositivo
- Bloqueio de dispositivos comprometidos

### 2. Sistema de Feeds

**Comunicados e Notícias**
- Carrossel de feeds na home
- Visualização detalhada de comunicados
- Listagem completa de feeds
- Filtros por data e categoria
- Notificações push de novos comunicados

### 3. Perfis de Usuário

O aplicativo identifica automaticamente o perfil e exibe funcionalidades específicas:

#### a) PAP Vendedor / PME
**Funcionalidades:**
- 📍 **Rotas**: Gerenciamento de carteira de clientes com rotas
- 🛒 **Registro Avulso**: Registrar vendas fora da rota
- 📊 **Resultados**: Dashboard de vendas e metas
- 📅 **Calendário**: Agenda de visitas
- 📄 **Folheto Virtual**: Catálogo de produtos
- 🤝 **Negociações**: Acompanhamento de propostas
- 🧮 **Calculadora de Ofertas**: Simulação de planos

#### b) Consultor BCC (Premium/Condomínios)
**Funcionalidades:**
- 🏢 **Carteira**: Gestão de condomínios
- 🚩 **Plantão**: Sistema de check-in/out em plantões
- 💰 **Venda**: Registro de vendas em plantões
- ✅ **Ações**: Atividades do plantão (checklist)
- 🎯 **Oportunidades**: Mailing de prospects
- 📞 **Atendimento**: Registro de atendimentos
- 📱 **Leads URA**: Gestão de leads telefônicos
- 📊 **Resultados**: Métricas e rankings
- 📅 **Calendário**: Agendamentos
- 📄 **Folheto Virtual**: Catálogo
- 🤝 **Negociações**: Propostas em andamento
- 🧮 **Calculadora**: Simulação de ofertas

#### c) Consultor Híbrido
Combina funcionalidades de PAP e BCC

#### d) Supervisor PAP
**Funcionalidades:**
- 📊 **Dashboard**: Visão gerencial da equipe
- 📈 **Resultados**: Métricas de vendedores
- 👥 **Ranking**: Classificação de vendedores
- 📅 **Calendário**: Visão da equipe
- 🤝 **Negociações**: Acompanhamento geral

#### e) PAP Filial / PAP GN
Funcionalidades de gestão regional e nacional

---

## 📁 Estrutura de Módulos

### Módulo: Auth (Autenticação)

**Localização**: `lib/ui/modules/auth/`

#### Páginas:
- **`login.dart`**: Tela de login principal
  - Campos: Email e senha
  - Validações em tempo real
  - Botão "Esqueci minha senha"
  - Opção de login SSO (quando disponível)
  - Marca d'água de ambiente (homolog/dev)
  
- **`splash.dart`**: Tela de carregamento inicial
  - Carrega dados do usuário
  - Verifica permissões
  - Redireciona para home ou login
  
- **`password_recover/`**: Recuperação de senha
  - Envio de email de recuperação
  - Validação de código
  - Redefinição de senha

#### ViewModel:
- **`LoginViewModel`**
  - Gerencia estado do login
  - Valida credenciais
  - Integra com OAuth2
  - Armazena dados localmente
  
**Funções principais:**
```dart
- initialize()              // Inicializa dados salvos
- validateLogin()           // Valida campos de login
- verifyUserSSOAccess()     // Verifica se usuário tem acesso SSO
- login()                   // Executa login
- _restoreStoredUserData()  // Restaura dados offline
- clear()                   // Limpa estado
```

---

### Módulo: Home

**Localização**: `lib/ui/modules/home/`

#### Páginas:
- **`pap_tools_home_page.dart`**: Página principal
  - AppBar com foto e nome do usuário
  - Botão de notificações
  - PageView com duas abas (Home/Perfil)
  - Bottom Navigation Bar
  - Background personalizado

- **`pap_tools_startup_page.dart`**: Conteúdo da aba Home
  - Feed de comunicados (carrossel)
  - Alerta de permissão de localização
  - Grid de utilitários (ferramentas)
  - Pull-to-refresh

#### ViewModel:
- **`HomeViewModel`**
  - Controla índice da página atual
  - Gerencia navegação entre abas
  
**Propriedades:**
```dart
- pageIndex: int                    // Índice da página (0=Home, 1=Perfil)
- currentPageIsPerfil: bool         // Se está na aba de perfil
- changeIndex(int index)            // Muda de página
```

#### Tools Builder

**Localização**: `lib/ui/modules/home/tools-builder/`

Sistema que constrói dinamicamente os utilitários baseado no perfil:

- **`tools_builder.dart`**: Factory principal
  - Identifica o perfil do usuário
  - Retorna lista de widgets específicos
  - Registra event handlers quando necessário

- **`pap_sales_promoter_tools.dart`**: Ferramentas do vendedor PAP
- **`bcc_consultant_tools.dart`**: Ferramentas do consultor BCC
- **`hybrid_consultant_tools.dart`**: Ferramentas híbridas
- **`pap_supervisor_tools.dart`**: Ferramentas de supervisor
- **`pap_gn_tools.dart`**: Ferramentas GN
- **`pap_filial_tools.dart`**: Ferramentas Filial

---

### Módulo: Feeds

**Localização**: `lib/ui/modules/feeds/`

#### Páginas:
- **`feeds_carousel_slider_page.dart`**: Carrossel de comunicados
  - Auto-play configurável
  - Indicadores de página
  - Link "Ver todos"
  - Cards clicáveis

- **`feed_detail_page.dart`**: Detalhes do comunicado
  - Título completo
  - Descrição expandida
  - Data de publicação
  - Botão compartilhar

- **`all_feeds_list_page.dart`**: Lista completa
  - Todos os feeds disponíveis
  - Scroll infinito
  - Ordenação por data

#### ViewModel:
- **`FeedViewModel`**
  - Carrega feeds da API
  - Gerencia estado de loading
  - Controla carrossel
  - Verifica permissões de localização

**Funções:**
```dart
- initialize()                      // Carrega feeds
- thereIsNoFeedsToShow(): bool      // Verifica se há feeds
- isLocationPermissionDenied(): bool // Checa permissão GPS
```

---

### Módulo: User Profile

**Localização**: `lib/ui/modules/user-profile/`

#### Páginas:
- **`pap_tools_user_profile_page.dart`**: Perfil do usuário
  - Avatar circular
  - Nome e email do usuário
  - Cards de Suporte e Sobre o App
  - Seção "Preferências"
  - Botão de logout

**Componentes:**
```dart
- _buildUtilityCard()      // Cards de suporte/sobre
- _buildWidgetExitApp()    // Botão de sair
```

**Fluxo de Logout:**
1. Revoga token SSO (se existir)
2. Reseta índice da home
3. Navega para tela de login
4. Remove todas as rotas anteriores

#### Widgets:
- **`profile_support_dialog_widget.dart`**: Dialog de suporte
  - Links de contato
  - Email/telefone
  - FAQ

- **`profile_about_app_dialog_widget.dart`**: Dialog sobre o app
  - Versão do aplicativo
  - Termos de uso
  - Política de privacidade

---

### Módulo: Profiles (Funcionalidades por Perfil)

**Localização**: `lib/ui/modules/profiles/`

#### General (Compartilhado entre perfis)

**Calculator** - `general/calculator/`
- Calculadora de ofertas
- Simulação de planos
- Cálculo de descontos

**Calendar** - `general/calendar/`
- Agenda de visitas
- Eventos agendados
- Sincronização com sistema

**Dashboard** - `general/dashboard/`
- Métricas de vendas
- Gráficos de desempenho
- Ranking de vendedores
- Metas e resultados

**Folheto Virtual** - `general/folheto-virtual/`
- Catálogo digital de produtos
- Detalhes de ofertas
- Compartilhamento

**Negotiations** - `general/negotiations/`
- Lista de negociações
- Status de propostas
- Histórico de negociações
- Detalhamento de cada proposta

**Rejected Sale** - `general/rejected-sale/`
- Vendas rejeitadas
- Motivos de rejeição
- Possibilidade de reavaliação

---

#### BCC Premium Consultants

**Customer Service** - `bcc-premium-consultants/customer_service/`
- Registro de atendimentos
- Histórico de interações
- Classificação de atendimento

**Leads URA** - `bcc-premium-consultants/leads-ura/`
- Lista de leads telefônicos
- Detalhes do lead
- Ações disponíveis
- Status de conversão

**Mailing** - `bcc-premium-consultants/mailing/`
- Lista de oportunidades
- Filtros por status
- Detalhes de prospects
- Ações de contato
- Popup de registro de venda

**Orderly (Plantão)** - `bcc-premium-consultants/orderly/`

Sistema complexo de gerenciamento de plantões:

**Páginas:**
- `orderly_page.dart`: Lista de plantões
  - Plantões agendados
  - Plantão atual (em andamento)
  - Histórico de plantões
  - Criar novo plantão
  
**ViewModel:**
- `OrderlyViewModel`: Gerencia todo o ciclo do plantão
  
**Funcionalidades:**
```dart
- initialize()                  // Carrega plantões
- checkIn()                     // Inicia plantão
- checkOut()                    // Finaliza plantão
- hasAlreadyOneOrderlyOnGoing() // Verifica plantão ativo
- getCurrentOrderly()           // Retorna plantão atual
```

**Widgets:**
- `orderly_card_widget.dart`: Card de plantão
- `orderly_scheduled_list.dart`: Lista de agendados
- `finished_orderly_widget.dart`: Plantões finalizados
- `schedule_created_widget.dart`: Confirma agendamento
- `orderly_scheduler_widget.dart`: Agendar novo plantão

**Events:**
Sistema de eventos para sincronização:
- `OrderlyInitializedEventHandler`
- `OrderlyCheckedInEventHandler`
- `OrderlyCheckedOutEventHandler`

---

**Orderly Activities** - `bcc-premium-consultants/orderly-activities/`

Ações/checklist durante o plantão:

**Funcionalidades:**
- Lista de atividades obrigatórias
- Check/uncheck de atividades
- Progresso do plantão
- Validação de conclusão

**Events:**
- `OrderlyActivitiesInitializedEventHandler`
- `OrderlyActivitiesUpdatedEventHandler`
- `OrderlyActivitiesResetedEventHandler`

---

**Sale (Venda BCC)** - `bcc-premium-consultants/sale/`

Fluxo de venda premium com múltiplas etapas:

**Páginas:**
- `bcc_premium_sale_page.dart`: Container principal
- Steps:
  - `bcc_premium_info_step_page.dart`: Informações do cliente
  - `bcc_premium_sale_product_step_page.dart`: Seleção de produtos
  - `bcc_premium_sale_summary_step_page.dart`: Resumo e confirmação

**ViewModel:**
- Gerencia navegação entre steps
- Valida cada etapa
- Envia venda para API

---

**Wallet (Carteira)** - `bcc-premium-consultants/wallet/`

Gerenciamento de condomínios:

**Funcionalidades:**
- Lista de condomínios atribuídos
- Detalhes do condomínio
- Contatos do síndico
- Histórico de visitas
- Mapa de localização

---

#### PAP Sales Promoter

**PAP Results** - `pap-sales-promoter/pages/pap-results/`
- Dashboard de vendas
- Metas mensais
- Comparativo com período anterior
- Gráficos de desempenho

**PAP Route** - `pap-sales-promoter/pages/pap-route/`
- Lista de clientes na rota
- Mapa com localização
- Status de visita
- Navegação GPS

**Sale (Venda PAP)** - `pap-sales-promoter/pages/sale/`
- Formulário simplificado de venda
- Seleção de produtos
- Dados do cliente
- Registro de não-venda
- Motivos de recusa

---

## 👤 Perfis de Usuário

### Sistema de Perfis

**Localização**: `lib/core/profile/pap_tools_access_profile.dart`

O sistema identifica automaticamente o perfil através de:
- Canal de acesso (Premium, Condomínios, PAP Indireto)
- Dashboard de visualização (Filial, GN)
- Tipo de parceiro (PME)

### Perfis Disponíveis:

#### 1. PAPSalesPromoterProfile
**Critérios:**
- `mobileAcesso == 1`
- `canalPapindireto == 1`

**Acesso:**
- Rotas
- Registro avulso
- Resultados
- Calendário
- Folheto virtual
- Negociações
- Calculadora

---

#### 2. BCCConsultantProfile
**Critérios:**
- `mobileAcesso == 1`
- `canalPremium == 1` OU `canalCondiminios == 1`

**Acesso:**
- Carteira
- Plantão
- Venda premium
- Ações de plantão
- Oportunidades (Mailing)
- Atendimento
- Leads URA
- Resultados
- Calendário
- Folheto virtual
- Negociações
- Calculadora

**Event Handlers registrados:**
- OrderlyInitializedEventHandler
- OrderlyCheckedInEventHandler
- OrderlyCheckedOutEventHandler
- OrderlyActivitiesInitializedEventHandler
- OrderlyActivitiesUpdatedEventHandler
- OrderlyActivitiesResetedEventHandler

---

#### 3. HybridConsultantProfile
**Critérios:**
- `mobileAcesso == 1`
- `canalPapindireto == 1` E `canalCondiminios == 1`

**Acesso:**
Combinação de funcionalidades PAP + BCC

---

#### 4. PAPSupervisorProfile
**Critérios:**
- Supervisor de equipe

**Acesso:**
- Dashboard gerencial
- Resultados da equipe
- Ranking
- Negociações

---

#### 5. PAPGNProfile
**Critérios:**
- `dashVendaVisita == 'filial'`

**Acesso:**
- Dashboard nacional
- Métricas agregadas

---

#### 6. PAPFilialProfile
**Critérios:**
- `dashVendaVisita == 'gn'`

**Acesso:**
- Dashboard de filial
- Métricas regionais

---

#### 7. PMEProfile
**Critérios:**
- `descricaotipoparceiro == 'pme'`

**Acesso:**
- Funcionalidades similares ao PAP Sales Promoter

---

### Factory de Perfis

```dart
PapToolsAccessProfile profile = 
    PapToolsAccessProfileFactory.create(userInfo);
```

O factory analisa os dados do usuário e retorna o perfil apropriado.
Lança exceção se não encontrar perfil válido.

---

## 🔐 Fluxo de Autenticação

### 1. Inicialização do App

```
main.dart
  ├─ Inicializa Firebase
  ├─ Configura Crashlytics
  ├─ Verifica segurança do dispositivo
  └─ Inicia PapToolsAppWidget
       └─ Verifica dispositivo comprometido
            ├─ SIM: Mostra CompromisedDeviceWarningPage
            └─ NÃO: Navega para PageLogin
```

### 2. Tela de Login

**Fluxo Padrão:**

```
PageLogin (login.dart)
  ├─ LoginViewModel.initialize()
  │    └─ Carrega credenciais salvas (se existir)
  │
  ├─ Usuário preenche email/senha
  ├─ LoginViewModel.validateLogin()
  │    └─ Valida formato de email
  │    └─ Valida senha não vazia
  │
  └─ Ao clicar em "Entrar"
       ├─ Verifica se tem conexão
       │    └─ SEM conexão: Tenta login offline
       │    └─ COM conexão: Continua fluxo
       │
       ├─ LoginViewModel.verifyUserSSOAccess()
       │    └─ Verifica se usuário tem SSO habilitado
       │         ├─ TEM SSO: Exibe botão SSO
       │         └─ SEM SSO: Continua login tradicional
       │
       └─ LoginViewModel.login()
            ├─ AuthDatasource.userAccess()
            ├─ Salva credenciais localmente
            ├─ Identifica perfil do usuário
            ├─ AppModel.setUserAccess()
            └─ Navega para PageSplash
```

**Fluxo SSO (OAuth2):**

```
_oAuth2Authenticate()
  ├─ OAuthService.authenticate()
  │    └─ Abre browser com URL de autenticação
  │    └─ Usuário faz login no SSO
  │    └─ Retorna com código
  │
  ├─ OAuthService.exchangeCode(code)
  │    └─ Troca código por access token
  │
  ├─ Salva ssoAccessToken
  └─ LoginViewModel.login()
```

### 3. Splash Screen

```
PageSplash (splash.dart)
  ├─ Verifica permissões
  │    ├─ Localização
  │    ├─ Câmera
  │    └─ Notificações
  │
  ├─ Solicita permissões faltantes
  ├─ Carrega dados iniciais
  └─ Navega para HandsHomePage
```

### 4. Home Autenticada

```
HandsHomePage
  ├─ Exibe nome e perfil do usuário
  ├─ Carrega ferramentas do perfil
  ├─ Inicializa feeds
  └─ Ativa notificações para o perfil
```

---

## 📦 Repositórios e Serviços

### Repositórios

**Localização**: `lib/data/repository/`

#### CondomíniumRepository
```dart
- getCondominiums()           // Lista condomínios da carteira
- getCondominiumDetails(id)   // Detalhes do condomínio
- updateCondominium()         // Atualiza informações
```

#### FeedsRepository
```dart
- getFeeds()                  // Busca comunicados
- markAsRead(feedId)          // Marca como lido
```

#### MailingRepository
```dart
- getMailingList()            // Lista de oportunidades
- getMailingDetails(id)       // Detalhes da oportunidade
- registerAction()            // Registra ação no mailing
```

#### NegotiationRepository
```dart
- getNegotiations()           // Lista negociações
- getNegotiationDetails(id)   // Detalhes da negociação
- updateStatus()              // Atualiza status
```

#### LeadsUraRepository
```dart
- getLeads()                  // Lista leads telefônicos
- getLeadDetails(id)          // Detalhes do lead
- registerContact()           // Registra contato
```

#### OrderlySummaryRepository
```dart
- getOrderlySummary()         // Resumo de plantões
- createOrderly()             // Cria novo plantão
- checkIn()                   // Check-in em plantão
- checkOut()                  // Check-out de plantão
```

#### WalletRepository
```dart
- getWallet()                 // Carteira de clientes
- updateCustomer()            // Atualiza cliente
```

#### ProductsRepository
```dart
- getProducts()               // Lista produtos disponíveis
- getProductDetails(id)       // Detalhes do produto
- getOffers()                 // Ofertas ativas
```

#### NotificationRepository
```dart
- initialize()                           // Inicializa FCM
- configureNotificationsForProfile()     // Configura por perfil
- getToken()                             // Retorna FCM token
- subscribeToTopic(topic)                // Inscreve em tópico
- unsubscribeFromTopic(topic)            // Remove inscrição
```

#### CustomerServiceRepository
```dart
- getServiceHistory()         // Histórico de atendimentos
- registerService()           // Registra atendimento
```

---

### Serviços

**Localização**: `lib/data/services/`

#### ConnectionService
```dart
- initialize()                // Monitora conexão
- hasConnection: bool         // Status da conexão
```

#### LocalAuthService
```dart
- canAuthenticate()           // Verifica biometria disponível
- authenticate()              // Autentica com biometria
```

#### SharedPreferenceService
```dart
- getUserCredentials()        // Recupera credenciais
- saveUserCredentials()       // Salva credenciais
- getUserData()               // Recupera dados do usuário
- saveUserData()              // Salva dados do usuário
- hasUserData()               // Verifica se tem dados
- clear()                     // Limpa tudo
```

---

### Datasources

**Localização**: `lib/data/datasources/`

#### AuthDatasource
```dart
- verifyUserSSOAccess(email)  // Verifica SSO
- userAccess(email, password) // Login tradicional
- refreshToken()              // Atualiza token
```

#### LocationDatasource
```dart
- getCurrentPosition()        // Posição atual
- getLastKnownPosition()      // Última posição conhecida
- checkPermission()           // Verifica permissão
- requestPermission()         // Solicita permissão
```

---

## 🔄 Gerenciamento de Estado

### Padrão Utilizado

O app usa **ChangeNotifier** com **Provider pattern**:

```dart
class ExampleViewModel extends ChangeNotifier {
  bool _isLoading = false;
  
  bool get isLoading => _isLoading;
  
  void setLoading(bool value) {
    _isLoading = value;
    notifyListeners(); // Notifica listeners
  }
}
```

### Singleton Pattern

Todos os ViewModels seguem o padrão Singleton:

```dart
class HomeViewModel extends ChangeNotifier {
  HomeViewModel._();
  
  static HomeViewModel? _instance;
  
  static HomeViewModel get instance {
    _instance ??= HomeViewModel._();
    return _instance!;
  }
}
```

### ListenableBuilder

Na UI, usamos `ListenableBuilder` para reagir a mudanças:

```dart
ListenableBuilder(
  listenable: viewModel,
  builder: (context, child) {
    return Text(viewModel.someProperty);
  },
)
```

### AppModel - Estado Global

**Localização**: `lib/data/models/app_model.dart`

Singleton que armazena estado global:

```dart
AppModel.instance
  ├─ userAccess          // Dados de acesso
  ├─ userInfo            // Informações do usuário
  ├─ profile             // Perfil de acesso
  ├─ token               // Token de autenticação
  ├─ ssoAccessToken      // Token SSO
  ├─ guidIdOperador      // GUID do operador
  ├─ latitude/longitude  // Posição atual
  └─ hasConnection       // Status de conexão
```

**Métodos:**
```dart
- setUser(UserInfo)                    // Define usuário
- setHandisProfile(PapToolsAccessProfile) // Define perfil
- setUserAccess(UserAccess)            // Define acesso
- setCurrentPosition(Position)         // Atualiza posição
- setHasConnection(bool)               // Atualiza conexão
```

---

## 🛡️ Segurança

### Device Security Checker

**Localização**: `lib/core/security/device_security_checker.dart`

Verifica integridade do dispositivo:

```dart
DeviceSecurityChecker.performSecurityCheck()
  ├─ Verifica root (Android)
  ├─ Verifica jailbreak (iOS)
  ├─ Verifica developer mode
  └─ Retorna SecurityResult
       ├─ isDeviceCompromised: bool
       └─ reasons: List<String>
```

### CompromisedDeviceWarningPage

Se dispositivo comprometido:
- Bloqueia acesso ao app
- Exibe mensagem de segurança
- Não permite prosseguir

### Autenticação OAuth2

**Localização**: `lib/core/oauth2/`

#### OAuthConfig
```dart
- authorizationUrl    // URL de autorização
- tokenUrl            // URL para trocar código
- introspectUrl       // URL para validar token
- revokeUrl           // URL para revogar token
- redirectUri         // URI de callback
- urlScheme           // Scheme do app
```
**ViewModels:**
- Sempre estender `ChangeNotifier`
- Implementar padrão Singleton
- Chamar `notifyListeners()` após mudanças de estado
- Dispor recursos no `dispose()`

**Widgets:**
- Preferir `StatelessWidget` quando possível
- Usar `const` constructors
- Separar widgets grandes em componentes menores

**Repositories:**
- Um repositório por entidade
- Métodos assíncronos retornam `Future<Result<T>>`
- Tratar erros com try/catch

### Adicionar Novo Perfil

1. **Definir perfil em** `pap_tools_access_profile.dart`:
```dart
class NewProfile extends PapToolsAccessProfile {}
```

2. **Atualizar factory**:
```dart
static PapToolsAccessProfile create(UserInfo user) {
  // ... lógica de identificação
  if (condition) {
    return NewProfile();
  }
}
```

3. **Criar tools builder** em `tools-builder/new_profile_tools.dart`:
```dart
class NewProfileTools {
  static List<Widget> build(BuildContext context) => [
    StartupToolCardWidget(/* ... */),
  ];
}
```

4. **Atualizar** `tools_builder.dart`:
```dart
if (_app.profile is NewProfile) {
  return NewProfileTools.build(context);
}
```

### Adicionar Nova Funcionalidade

1. **Criar módulo** em `lib/ui/modules/profiles/[perfil]/[funcionalidade]/`
2. **Estrutura padrão**:
```
funcionalidade/
  ├── pages/
  │   └── funcionalidade_page.dart
  ├── viewmodels/
  │   └── funcionalidade_viewmodel.dart
  └── widgets/
      └── funcionalidade_widget.dart
```

3. **Criar repository** (se necessário):
```dart
// lib/data/repository/funcionalidade_repository.dart
class FuncionalidadeRepository {
  Future<Result<Data>> getData() async {
    // implementação
  }
}
```

4. **Adicionar ao tools builder**:
```dart
StartupToolCardWidget(
  onTap: () {
    Navigator.push(
      context,
      MaterialPageRoute(
        builder: (context) => FuncionalidadePage()
      ),
    );
  },
  icon: LucideIcons.icon,
  title: 'Nome',
),
```

### Debugging

**Logs:**
```dart
import 'package:logger/logger.dart';

final logger = Logger();

logger.d('Debug');
logger.i('Info');
logger.w('Warning');
logger.e('Error');
```

**Firebase Crashlytics:**
```dart
FirebaseCrashlytics.instance.recordError(
  error,
  stackTrace,
  fatal: true,
);
```

**Debug de Estado:**
```dart
@override
void notifyListeners() {
  print('State changed: ${toString()}');
  super.notifyListeners();
}
```

### Testes

**Unit Tests:** `test/unit/`
```dart
void main() {
  group('LoginViewModel', () {
    test('validateLogin should validate email', () {
      // arrange
      final viewModel = LoginViewModel();
      viewModel.controllerEmail.text = 'invalid';
      
      // act
      viewModel.validateLogin();
      
      // assert
      expect(viewModel.loginValidation.isValid, false);
    });
  });
}
```

**Integration Tests:** `test/integration/`

---

## 📚 Referências

### Documentação Externa

- [Flutter Docs](https://docs.flutter.dev/)
- [Firebase Flutter](https://firebase.google.com/docs/flutter/setup)
- [Google Maps Flutter](https://pub.dev/packages/google_maps_flutter)
- [OAuth 2.0](https://oauth.net/2/)

### Bibliotecas Principais

| Biblioteca | Versão | Uso |
|-----------|--------|-----|
| flutter_localizations | SDK | Localização PT-BR |
| http | 1.2.0 | Requisições HTTP |
| provider | 6.0.5 | State management |
| google_maps_flutter | 2.10.0 | Mapas |
| firebase_messaging | 16.0.0 | Push notifications |
| shared_preferences | 2.2.3 | Storage local |
| sqflite | 2.4.1 | SQLite |
| geolocator | 14.0.2 | GPS |
| camera | 0.11.0 | Câmera |
| local_auth | 2.2.0 | Biometria |
| flutter_web_auth_2 | 4.1.0 | OAuth2 |

---

## 🔍 Troubleshooting

### Problemas Comuns

**1. Erro de autenticação SSO**
- Verificar OAuthConfig environment
- Confirmar redirect URI configurado
- Validar certificados SSL

**2. Permissão de localização negada**
- Solicitar novamente nas configurações
- Widget DisabledLocationPermissionWidget orienta usuário

**3. Feeds não carregam**
- Verificar conexão de internet
- Conferir token de autenticação
- Validar endpoint da API

**4. Plantão não inicia**
- Verificar permissão de localização

