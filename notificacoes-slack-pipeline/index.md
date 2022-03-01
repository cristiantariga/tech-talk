# Notificações no Slack via Pipeline

## Motivação:
Usavamos as notificações integradas ao Slack, onde era possível ser notificado em caso de sucesso ou falha. Um dos problemas dessa solução, é a baixa quantidade de informações presentes nas notificações enviadas. Em uma reunião de chapter, elegemos as principais informações necessárias para visualizar em uma notificação.

<b> Informações Necessárias: </b>

* Número da Release
* Atalho para o App Center
* Atalho para a Release no GitHub

![Relato Chatper](https://github.com/cristiantariga/tech-talk/blob/main/notificacoes-slack-pipeline/images/relato%20chapter.png?raw=true)

## Notificação:

Ao invés de usarmos a integração padrão anexada ao Slack, partimos para as funcionalidades disponiveis da [API do Slack](https://api.slack.com/web).
O código usado para envio de uma mensagem personalizada, baseia se em um atributo <b>data</b> onde as informações iniciais indicam que é possível passar a mensagem desejada em formato de texto.

```curl -X POST -H 'Content-type: application/json' --data '{"text":"Minha mensagem"}' YOUR_WEBHOOK_URL```

Nesta etapa, havia duas dúvidas principais:
* Como criar um WebHook?
* O atributo data aceita somente texto?

### WebHook

Webhook é uma forma de recebimento de informações, que são passadas quando um evento acontece. Dessa forma, o webhook na prática, é a forma de receber informações entre dois sistemas de uma forma passiva. É uma maneira prática para um app ou sistema fornecer outras aplicações com informações em tempo real.

Para ter acesso ao WebHook no Slack, tive que criar um novo [App](https://api.slack.com/apps?new_app=1), escolhendo um nome e workspace no qual deseja que o app tenha permissão para utilizar. Lembrando que o nome escolhido aparecerá como nome do remetente da notificação (exemplo: Narutão)

![Criar novo App](https://github.com/cristiantariga/tech-talk/blob/main/notificacoes-slack-pipeline/images/criar%20app.png?raw=true)

Além disso, tive que habilitar a utilização de WebHooks, logo na sequência:

![Habilitar WebHook](https://github.com/cristiantariga/tech-talk/blob/main/notificacoes-slack-pipeline/images/ativar%20webhook.png?raw=true)

Após habilitar, é possível criar um novo WebHook, escolhendo o canal que será enviado a mensagem, dentro do workspace anteriormente escolhido.
Agora o código citado anteriormente, teve o trecho YOUR_WEBHOOK_URL automaticamente substituido pela verdadeira URL do WebHook.

![WebHook URL](https://github.com/cristiantariga/tech-talk/blob/main/notificacoes-slack-pipeline/images/url%20do%20webhook.png?raw=true)

Nessa etapa, uma das minhas perguntas iniciais já foi respondida:
- [X] Como criar um WebHook?
- [ ] O atributo data aceita somente texto?

### Data
Ao ler a [documentação](https://api.slack.com/messaging/interactivity#getting_started) sobre mensagens, descobri que o <b>data</b> aceita objetos muito mais complexos.
Além disso, uma ferramenta que me auxiliou muito foi a [Block Kit Builder](), onde é possível montar a mensagem de maneira modular, acrescentando botões, imagens, divisores e muito mais, de maneira interativa e simplificada.

![Block Kit Builder](https://github.com/cristiantariga/tech-talk/blob/main/notificacoes-slack-pipeline/images/block%20kit%20builder.png?raw=true)

Nesse momento, as duas dúvidas iniciais foram respondidas:
- [X] Como criar um WebHook?
- [X] O atributo data aceita somente texto?
