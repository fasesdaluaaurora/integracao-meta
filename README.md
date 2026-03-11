# 📚 Estudos — Meta Broker

Repositório destinado ao registro de estudos, anotações técnicas e experimentos relacionados à construção e integração de um **broker para serviços da Meta**, incluindo principalmente APIs de mensageria como **WhatsApp Cloud API**.

O objetivo é centralizar conhecimento sobre autenticação, webhooks, segurança, arquitetura e padrões de integração necessários para operar um serviço intermediário (broker) entre aplicações clientes e os serviços da Meta.

## 📖 Links e referencias:

Playground WhatsApp Flows: https://developers.facebook.com/docs/whatsapp/flows/playground?nav_ref=biz_unified_f3_login_page_to_dfc


---

## 🧱 Conceito de Broker

O **Meta Broker** funciona como uma camada intermediária responsável por:

- Gerenciar autenticação com a Meta
- Receber e validar **webhooks**
- Encaminhar eventos para broker
- Gerenciar múltiplos aplicativos Meta

Arquitetura simplificada:

Meta Platforms
│
│ Webhooks / API
▼
Broker (middleware)
│
│ API interna
▼
Aplicação integradora


---

## 🔐 Segurança

- Validação de **webhook tokens** 

- **mTLS (Mutual TLS)**, com certificado único por cliente, validado na AWS, através de LoadBalancer.

- Assinatura de requisições
- Rotação de tokens
- Controle de acesso por aplicação

---

## 🔔 Webhooks

Os webhooks são o principal mecanismo de comunicação assíncrona utilizado pela Meta.

Tópicos estudados:

- Verificação inicial do webhook (`hub.challenge`)

Verficado através de rota, onde o token é único por cliente e salvo em banco, ao receber a requisição, o broker busca no banco, se o webhook informado existe em um cliente ativo e confirma, retornando o valor de challenge com 200 ok.
  
```bash
curl -X GET \
"https://meuservico.com/webhook?hub.mode=subscribe&hub.verify_token=TOKEN&hub.challenge=1234"
```

- Estrutura de payload
- Eventos de mensagens
- Retry e idempotência
- Verificação de integridade

---

## 🪪 Autenticação

Tipos de autenticação utilizados pelas APIs da Meta:

- **Access Tokens**
- **App Tokens**
- **System User Tokens**
- **JWT interno do broker**
- Validação de tokens por cliente

---

## ⚙️ Multi-tenant

### DÚVIDAS:

- A URL de callback deve ser única por cliente, passando id? ex: https://www.meubroker.com/webhook/{id_cliente} -> isso permite identificação do cliente no banco para consultas mais seguras.
- O perfil empresarial é único por cliente?

O broker precisa suportar múltiplos clientes. 


curl -v "https://www.frasea.ai:8443/webhook?hub.mode=subscribe&hub.challenge=123456789&hub.verify_token=wpp_webhook_40bb8701f2a1df8b4060431d3bf3dab1e308de1fd3769b28abc38138ef24066c" - get webhook validation



---------------------------------------------------------------------------------------------------------------------------------------

Modelo A — Cliente paga diretamente à Meta (recomendado)

Cada cliente possui:

Cliente
 └── Business Manager
      └── Payment Method
           └── WABA



Modelo B — Você paga e repassa

Estrutura:

Seu Business Manager
 └── Payment Method
      └── WABAs dos clientes


1. Como a Meta cobra mensagens

A cobrança do WhatsApp ocorre por conversa, vinculada ao WABA do cliente.

Estrutura de cobrança:

WABA
 └── Phone Number
      └── Conversations
           ├── Service conversation
           ├── Utility conversation
           ├── Marketing conversation
           └── Authentication conversation

Portanto, o custo sempre pertence ao WABA, não ao App.


7. Dica importante para brokers

Sempre salve:

waba_id
phone_number_id
display_phone_number

Porque:

templates são por WABA

mensagens são por phone_number_id


1. Estrutura das contas

Após o onboarding correto, a estrutura fica assim:

Seu Business Manager
    │
    └── Meta App (Frasea)
            │
            │ autorizado a acessar
            ▼
Cliente Business Manager
        │
        └── WhatsApp Business Account (WABA)
                │
                └── Phone Number

Pontos importantes:

o método de pagamento fica no Business Manager do cliente

você não tem acesso financeiro

o cliente não tem acesso ao seu App

apenas ocorre delegação de permissões



---------------------------------------------------------------------------------------------------------------------------------------

Caminho para o cliente permitir acesso do broker a sua conta:

Configurações -> Contas -> Aplicativos -> Seleciona aplicativo -> Atribuir parceiro -> cadastrar o id da Empresa que representa o Broker (id do portifólio empresarial é encontrado em "Informações da empresa" -> Identificação do portfólio empresarial: {id})

IPM Sistemas
Identificação do portfólio empresarial: 173564040404468



2. Solicitação Ativa (Você pede o acesso)
Se você já tem o ID do Gerenciador de Negócios do seu cliente, você pode tomar a iniciativa:
No seu Business Manager, vá em Configurações do Negócio.
Acesse Usuários > Parceiros.
Clique em Adicionar e escolha Solicitar ativos de um parceiro para trabalhar em nome dele.
Insira o ID do cliente, o seu e-mail de contato e selecione quais ativos você precisa (neste caso, a Conta do WhatsApp com permissão de "Gerenciar").
O cliente receberá uma solicitação formal no painel dele e precisará apenas clicar em Aprovar.
Por que isso é melhor para um Chatbot?
Como parceiro técnico, ao ser adicionado dessa forma, você consegue vincular o seu Aplicativo (App ID) à conta de WhatsApp do cliente. Isso permite que o seu sistema use o token de acesso para ler e enviar mensagens via API sem que o cliente precise lidar com códigos ou configurações técnicas complexas.
Você já tem o Business ID do seu cliente para enviar essa solicitação agora?


---------------------------------------------------------------------------------------------------------------------------------------
o processo atual será:

o cliente deve possuir uma conta WABA (WhatsApp Business Account) e o Número.

o broker (parceiro) tem o App. O App ID (criado no portal Meta for Developers) é que contém a inteligência e as configurações da API. 

A Vinculação: Quando o cliente te adiciona como parceiro e te dá acesso à WABA dele, o seu aplicativo "enxerga" o número dele e passa a disparar as mensagens.







