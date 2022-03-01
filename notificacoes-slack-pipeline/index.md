# Notificações no Slack via Pipeline

## Motivação:
Usavamos as notificações integradas ao Slack, onde era possível ser notificado em caso de sucesso ou falha. Um dos problemas dessa solução, é a baixa quantidade de informações presentes nas notificações enviadas. Em uma reunião de chapter, elegemos as principais informações necessárias para visualizar em uma notificação.

<b> Informações Necessárias: </b>

* Número da Release
* Atalho para o App Center
* Atalho para a Release no GitHub

## Notificação:

Ao invés de usarmos a integração padrão anexada ao Slack, partimos para as funcionalidades disponiveis da [API do Slack](https://api.slack.com/web).
O código usado para envio de uma mensagem personalizada, baseia se em um atributo <b>data</b> onde as informações iniciais indicam que é possível passar a mensagem desejada em formato de texto.

```curl -X POST -H 'Content-type: application/json' --data '{"text":"Minha mensagem"}' YOUR_WEBHOOK_URL```

Nesta etapa, havia duas dúvidas principais:
* Como criar um WebHook?
* O atributo data aceita somente texto?

### WebHook

Webhook é uma forma de recebimento de informações, que são passadas quando um evento acontece. Dessa forma, o webhook na prática, é a forma de receber informações entre dois sistemas de uma forma passiva. É uma maneira prática para um app ou sistema fornecer outras aplicações com informações em tempo real.

Para ter acesso ao WebHook no Slack, tive que criar um [App](https://api.slack.com/apps?new_app=1)  
