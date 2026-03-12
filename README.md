# 📚 Estudos — Meta Broker

Repositório destinado ao registro de estudos, anotações técnicas e experimentos relacionados à construção e integração de um **broker para serviços da Meta**, incluindo principalmente APIs de mensageria como **WhatsApp Cloud API**.

O objetivo é centralizar conhecimento sobre autenticação, webhooks, segurança, arquitetura e padrões de integração necessários para operar um serviço intermediário (broker) entre aplicações clientes e os serviços da Meta.

## 📖 Links e referencias:

Playground WhatsApp Flows: https://developers.facebook.com/docs/whatsapp/flows/playground?nav_ref=biz_unified_f3_login_page_to_dfc <br>
Introdução a webhooks: https://developers.facebook.com/docs/graph-api/webhooks/getting-started/#configure-webhooks-product <br>
Como gerar um token de acesso: https://developers.facebook.com/documentation/business-messaging/whatsapp/access-tokens#system-user-access-tokens <br>
Referencias de sintaxe de modelos de mensagem: https://developers.facebook.com/documentation/business-messaging/whatsapp/webhooks/reference/messages/text <br>
Conversa GPT: https://chatgpt.com/share/69b3148d-4228-800b-96b8-f4321264d08d

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




----------------------------------------------------------------

configuração de aplicativo meta em número de parceiro pelo business manager meta
A configuração de um aplicativo Meta em um número de parceiro (como um provedor de soluções ou agência) é feita através do Meta Business Manager (Gerenciador de Negócios) e do portal Meta for Developers. Este processo é essencial para quem utiliza a API oficial do WhatsApp (WABA) via parceiros comerciais. 
YouTube
YouTube
 +2
Passo a Passo para Configuração
Criação do Aplicativo no Meta for Developers:
Acesse o Meta for Developers e faça login com sua conta do Facebook.
Vá em Meus Aplicativos e clique em Criar aplicativo.
Selecione o tipo de aplicativo (geralmente "Negócios" ou "Outro") e o caso de uso WhatsApp.
Complete as configurações básicas, incluindo a URL da política de privacidade e termos de uso.
Vinculação do Aplicativo ao Portfólio Empresarial:
No Meta Business Suite, acesse as Configurações do negócio.
No menu lateral, vá em Contas > Aplicativos.
Clique em Adicionar e selecione Conectar um ID de aplicativo.
Insira o ID gerado no portal de desenvolvedores para vincular o app ao seu Business Manager.
Configuração do Número do WhatsApp com o Parceiro:
Em Configurações do negócio, vá em Contas > Contas do WhatsApp.
Clique em Adicionar conta do WhatsApp e siga as instruções para inserir o número de telefone (com DDI e DDD).
Importante: O número deve ser removido de qualquer aplicativo de celular (WhatsApp comum ou Business) antes de ser cadastrado na API.
Selecione o método de verificação (SMS ou ligação) para receber o código e confirmar a posse do número.
Atribuição do Parceiro:
Na seção de Contas do WhatsApp, selecione o número configurado.
Clique em Atribuir parceiros e insira a Identificação da empresa (ID) do parceiro que gerenciará o serviço.
Defina as permissões de acesso (gerenciar conta de anúncios, mensagens, etc.) e salve. 
YouTube
YouTube
 +6
Requisitos Importantes
Verificação da Empresa: É altamente recomendável que o seu portfólio empresarial (BM) esteja com o status de verificado para evitar limitações no envio de mensagens.
Forma de Pagamento: Certifique-se de ter um método de pagamento ativo no Business Manager para cobrir os custos das conversas iniciadas pela empresa. 
Notificações Inteligentes
Notificações Inteligentes
 +1
Você já possui o ID da empresa (Business ID) do parceiro que deseja adicionar?





undefined
undefined
undefined
11 sites
Como cadastar um número de WhatApp API na Meta ...
9 de abr. de 2025 — muito bem nesse vídeo eu vou mostrar para você como. fazer. a verificação do seu número de WhatsApp. API junto a meta Business Sui...


YouTube
·
Taylor Alcantara | Automação de Marketing

6:04
Como adicionar um parceiro a um conjunto de dados do seu portfólio ...
Como adicionar um parceiro a um conjunto de dados * Acesse as Configurações do Meta Business Suite. * Clique em Fontes de dados. *

Facebook
Meta Mudou Tudo! Como Criar o Novo App da API WhatsApp
5 de dez. de 2025 — fala galera beleza. a meta mudou de novo e se você trabalha com automação marketing ou tecnologia você já sabe. quando a meta mexe...


YouTube
·
Comunidade ZDG

12m
Mostrar tudo
o parceiro já possui um aplicativo, ao qual devo me vincular
Nesse caso, como o parceiro já possui o aplicativo, o processo é de concessão de acesso e não de criação. Você precisa autorizar que o aplicativo deles gerencie o seu número de WhatsApp dentro do seu ecossistema Meta.
Aqui estão os passos curtos para realizar essa vinculação:
Obtenha o ID da Empresa (Business ID) do Parceiro:
Solicite ao parceiro o identificador numérico do Gerenciador de Negócios deles.
Adicione o Parceiro no seu Business Manager:
Acesse as Configurações do Negócio no seu Meta Business Suite.
No menu lateral, vá em Usuários > Parceiros.
Clique em Adicionar > Conceder acesso a um parceiro a seus ativos.
Insira o ID do parceiro e avance.
Vincule o Ativo (WhatsApp) ao Aplicativo/Parceiro:
Na tela que abrir (ou em Contas > Contas do WhatsApp), selecione o seu número/conta do WhatsApp.
Ative a permissão de Gerenciar conta do WhatsApp (ou permissões específicas solicitadas pelo parceiro).
Clique em Salvar alterações.
Aprovação no App do Parceiro:
O parceiro receberá a notificação e fará a chamada da API usando o ID do aplicativo deles para "reivindicar" a permissão de envio de mensagens em seu nome.
Dica técnica: Se o parceiro estiver usando o modelo de Embedded Signup (Login Incorporado), você fará isso clicando em um botão dentro da plataforma deles, onde fará login no Facebook e selecionará sua conta comercial. Isso automatiza os passos acima.
O parceiro te enviou um link de integração ou pediu para você fazer o processo manualmente pelo Business Manager?




onde o parceiro encontra seu bussines id?
O parceiro pode encontrar o ID do Gerenciador de Negócios (também chamado de ID do Portfólio Empresarial) seguindo estas etapas no Meta Business Suite: 
Onde localizar o ID
Menu de Configurações: Acesse o menu lateral esquerdo e clique no ícone de engrenagem (Configurações).
Informações da Empresa: Role o menu lateral até o final e clique em Informações da empresa (ou "Informações do portfólio empresarial").
Exibição do ID: O número de identificação (geralmente com 15 ou 16 dígitos) aparecerá logo abaixo do nome da empresa no topo da página, rotulado como ID do Gerenciador de Negócios ou Identificação do portfólio empresarial. 
Facebook
Facebook
 +4
Método Alternativo (Pela URL)
Se ele já estiver logado no Gerenciador de Negócios, o ID também pode ser visualizado diretamente na barra de endereços do navegador. O número aparecerá logo após o termo business_id= na URL da página. 
Respond.io
Respond.io
 +1
Importante: Certifique-se de que ele está passando o ID do Gerenciador de Negócios e não o ID de uma Página ou de uma Conta de Anúncios específica, pois são números diferentes. 
Respond.io
Respond.io
 +1
O parceiro já te enviou esse ID ou você ainda está aguardando para prosseguir com a vinculação?





undefined
undefined
undefined
9 sites
How to Find Facebook Business Manager ID [May 2022] - Respond.io
31 de jan. de 2025 — How to Find Facebook Business ID: Browser Address Bar. Open Meta Business Suite and look at the URL in your browser address bar. Y...

Respond.io

Find Your Business Portfolio ID | Meta Business Help Center
Find your business ID. Your business portfolio ID is the unique number that corresponds to your business portfolio. You may need y...

Facebook
Como encontrar o número de identificação da conta de anúncios ...
O nome e o número de identificação da sua conta de anúncios são mostrados no menu suspenso de ativos de negócios na parte superior...

Facebook
Mostrar tudo
fazendo esse procedimento, o parceiro pode gerenciar meus números de whatsapp?
Sim, ao adicionar o parceiro e conceder acesso à sua Conta do WhatsApp (WABA), ele passará a ter permissão para gerenciar seus números através do aplicativo dele.
Mas é importante entender o que exatamente ele poderá fazer, dependendo das permissões que você der:
O que o parceiro poderá fazer:
Enviar e receber mensagens: Usar a API para automatizar o atendimento ou campanhas.
Gerenciar Templates: Criar e enviar modelos de mensagens (as "HSMs") para aprovação da Meta.
Ver métricas: Acompanhar o status de entrega e leitura das mensagens.
Configurar Webhooks: Conectar o número ao sistema de chat ou CRM dele.
O que ele NÃO pode fazer (a menos que você autorize):
Excluir sua conta comercial: O controle "mestre" do Business Manager continua sendo seu.
Acessar outros ativos: Ele só verá o que você selecionar especificamente (como a Conta do WhatsApp). Ele não terá acesso às suas Páginas do Facebook ou Contas de Anúncios, a menos que você também os vincule.
Dica de Segurança:
Na hora de vincular, o Meta Business Manager perguntará o nível de acesso. Para a maioria das integrações de API, você deve marcar a opção "Gerenciar conta do WhatsApp".
Uma dúvida importante: Esse parceiro é uma plataforma de atendimento (como um CRM/Chatbot) ou uma agência de marketing? Isso muda um pouco como você deve configurar as permissões de mensagens.




chatbot
Como se trata de um Chatbot, o parceiro precisa de acesso técnico total à API do WhatsApp para que as mensagens entrem e saiam do sistema dele automaticamente.
Ao conceder o acesso ao parceiro (usando o Business ID que ele te passar), você deve seguir estes detalhes específicos para o funcionamento do robô:
Atribuição de Ativos: Dentro das suas Configurações de Negócio, ao selecionar o parceiro, escolha Contas do WhatsApp.
Permissão de Controle Total: Marque a opção Gerenciar conta do WhatsApp (ou "Full Control"). Isso é necessário para que o chatbot dele possa:
Registrar o número na API.
Configurar os Webhooks (o "ouvido" do robô que percebe quando chega uma mensagem).
Enviar as respostas automáticas.
Templates de Mensagem: O parceiro também conseguirá criar e enviar para aprovação da Meta aqueles modelos de mensagem (notificações) que o chatbot usa para iniciar conversas.
Importante: Assim que você terminar a configuração no seu Business Manager, o parceiro geralmente precisa dar um "aceite" no painel dele para concluir a ponte entre o seu número e o código do robô.
Você já está com a tela de "Adicionar Ativos" aberta no seu Gerenciador de Negócios ou ainda vai solicitar o ID para ele?




eu como parceiro, posso pedir para um parceiro atribuir ativos dele para mim?
Sim, você pode. No ecossistema da Meta, a relação de parceria é de mão dupla: você pode tanto conceder acesso aos seus ativos quanto solicitar acesso aos ativos de outra empresa.
Como você é o parceiro que fornece o Chatbot, o fluxo ideal é você pedir para o seu cliente (o outro parceiro/empresa) te adicionar. Existem duas formas de fazer isso:
1. Solicitação Manual (Via Business Manager)
Você passa o seu ID do Gerenciador de Negócios para o cliente e ele te adiciona.
O que o cliente faz: Vai em Configurações do Negócio > Usuários > Parceiros > Adicionar > Conceder acesso a um parceiro.
O que você recebe: Uma notificação para aceitar a parceria e, em seguida, o cliente atribui a Conta do WhatsApp específica para o seu ID.
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




o cliente pode ter wabas sem ter um app?
Sim, o cliente pode e deve ter a WABA (WhatsApp Business Account) sem precisar criar um aplicativo próprio. [1, 2]
Na verdade, em modelos de parceria para Chatbot, o cenário padrão é exatamente este:
O Cliente tem a WABA e o Número: Ele detém a propriedade da conta de WhatsApp no Business Manager dele. [2, 3]
Você (Parceiro) tem o Aplicativo: O seu App ID (criado no portal Meta for Developers) é que contém a inteligência e as configurações da API. [1, 4]
A Vinculação: Quando o cliente te adiciona como parceiro e te dá acesso à WABA dele, o seu aplicativo "enxerga" o número dele e passa a disparar as mensagens. [4, 5]
Por que isso é bom para o cliente?
Ele não precisa lidar com configurações técnicas de desenvolvedor. [1]
Ele mantém a posse do número e do histórico de faturamento. [3]
Ele só precisa clicar em "Aprovar" na solicitação de ativos que você enviar. [6]
E para você (Parceiro/Chatbot)?
Você centraliza a manutenção do código em um único App ID. [4]
Você pode gerenciar múltiplos números de clientes diferentes usando o mesmo aplicativo. [4]
O cliente já tem o número verificado e a WABA criada no Business Manager dele?




eu, como parceiro, tenho um número de whatsapp cadastrado no meu próprio perfil empresarial, para testar, como vou fazer para receber mensagens e "escutar" que ela chegou, e responder?
Para você, como desenvolvedor do Chatbot, "escutar" e responder mensagens do seu próprio número exige configurar um Webhook e usar a API do WhatsApp Business.
Aqui está o roteiro técnico direto:
1. Configurar o Webhook (O "Ouvido")
O Webhook é uma URL do seu servidor que a Meta chamará toda vez que seu número receber uma mensagem.
No Meta for Developers, abra o seu Aplicativo.
No menu lateral, vá em WhatsApp > Configuração.
Em Webhook, clique em Editar.
Insira a URL de retorno (seu servidor HTTPS) e um Token de verificação (uma senha que você cria para validar a conexão).
Importante: Clique em Gerenciar e assine o campo messages. Isso faz a Meta avisar seu servidor quando chegar uma mensagem.
2. Receber a Mensagem (JSON)
Quando alguém enviar um "Oi" para o seu número, a Meta enviará um POST para sua URL com um JSON parecido com este:
json
{
  "object": "whatsapp_business_account",
  "entry": [{
      "changes": [{
          "value": {
            "messages": [{ "from": "5511999999999", "text": { "body": "Oi" } }]
          },
          "field": "messages"
      }]
  }]
}
Use o código com cuidado.

Seu código deve ler esse JSON para saber quem enviou e o que foi dito.
3. Responder a Mensagem (A "Voz")
Para responder, você deve fazer uma requisição POST para a API da Meta usando o seu Token de Acesso:
URL: https://graph.facebook.com
Header: Authorization: Bearer SEU_TOKEN_DE_ACESSO
Body (JSON):
json
{
  "messaging_product": "whatsapp",
  "to": "NUMERO_DO_CLIENTE",
  "type": "text",
  "text": { "body": "Olá! Eu sou o seu robô." }
}
Use o código com cuidado.

Onde pegar as informações no seu App:
ID do Número: Em WhatsApp > Configuração inicial (é um número de 15 dígitos).
Token de Acesso: Para testes rápidos, use o Token Temporário na mesma tela (ele dura 24h). Para produção, você precisará gerar um Token de Sistema nas Configurações do Negócio.
Dica de ouro: Use ferramentas como o Webhook.site ou ngrok para expor seu servidor local e ver os JSONs chegando em tempo real antes de programar o backend final.
Você já tem um servidor (Node, Python, PHP, etc.) rodando para receber esses dados ou quer uma recomendação de como subir um ambiente de teste?


------------------------------------------------------------------------------------

Arquitetura definida:

A empresa provedora de tecnologia possui um aplicativo Meta;
O aplicativo meta possui:
- App Id ()
- App Secret ()
- Webhook Verify Token (token para vinculo da aplicação provedora com a meta)
- URL do webhook (url do aplicativo web da empresa provedora, usado para checar se o webhook é valido, para a meta enviar eventos e se a aplicação está funcionando)
- 

------------------------------------------------------------------------------------
{
  "object": "whatsapp_business_account",
  "entry": [
    {
      "id": "102290129340398",
      "changes": [
        {
          "value": {
            "messaging_product": "whatsapp",
            "metadata": {
              "display_phone_number": "15550783881",
              "phone_number_id": "106540352242922"
            },
            "contacts": [
              {
                "profile": {
                  "name": "Sheena Nelson"
                },
                "wa_id": "16505551234"
              }
            ],
            "messages": [
              {
                "from": "16505551234",
                "id": "wamid.HBgLMTY1MDM4Nzk0MzkVAgASGBQzQTRBNjU5OUFFRTAzODEwMTQ0RgA=",
                "timestamp": "1749416383",
                "type": "text",
                "text": {
                  "body": "Does it come in another color?"
                }
              }
            ]
          },
          "field": "messages"
        }
      ]
    }
  ]
}




Esse é o "bilhete" (JSON) que a Meta envia para o seu servidor toda vez que alguém fala com o seu chatbot. Imagine que é um formulário de entrega com os seguintes campos:
1. Quem recebeu a mensagem (Sua empresa/Cliente)
id (o primeiro): É o ID da conta comercial (WABA) do seu cliente.
display_phone_number: O número de telefone que as pessoas veem no WhatsApp.
phone_number_id: Este é o RG do número. Você usará esse ID no seu código para dizer à Meta: "responda através deste número aqui".
2. Quem enviou a mensagem (O usuário final)
name ("Sheena Nelson"): O nome que a pessoa configurou no perfil do WhatsApp dela.
wa_id / from: O número de telefone da pessoa que mandou a mensagem (quem está falando com o robô).
3. Detalhes da conversa
id (dentro de messages): É o código único daquela mensagem específica. Serve para você não processar a mesma mensagem duas vezes ou para marcar como lida.
timestamp: O horário exato em que a mensagem foi enviada (em formato de segundos).
type ("text"): Diz que o que chegou foi um texto. Se fosse uma foto, aqui estaria "image".
body: O conteúdo da mensagem. É aqui que o seu robô lê o que a pessoa escreveu ("Isso vem em outra cor?").
Resumo para o seu banco de dados:
Ao receber esse objeto, seu sistema deve olhar para o phone_number_id para saber de qual cliente é aquela mensagem e para o from para saber com qual usuário o robô deve conversar.
