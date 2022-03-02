# Notificações no Slack via Pipeline

## Motivação:
Usavamos as notificações integradas ao Slack, onde era possível ser notificado em caso de sucesso ou falha. Um dos problemas dessa solução, é a baixa quantidade de informações presentes nas notificações enviadas. Em uma reunião de chapter, elegemos as principais informações necessárias para visualizar em uma notificação.

<b> Informações Necessárias: </b>

* Número da Release
* Atalho para o App Center
* Atalho para a Release no GitHub

>>> Essa era a ideia inicial, atualizamos conforme desenvolvimento e adicionamos mais algumas informações que vocês irão ver mais pra frente
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

### Arquivo SH
Ao avaliar o código gerado pelo Block Kit Builder, percebemos que ele se torna extenso e inviável de manter o código bruto dentro de qualquer pipeline. Foi considerado então, desacoplar o código e inseri-ló dentro dos arquivos de ci do projeto, fazendo referência ao arquivo nas pipelines, mas não escrevendo diretamente toda a estrutura.

Assim ficou a estrutura:

![Folder Slack Script](https://user-images.githubusercontent.com/53791328/156242033-58d247e4-78c9-4483-8565-f4c29b159a03.png)

Foi necessário criar dois templates, um de sucesso e um de falha, pelo fato de que as informações necessárias em ambos mudam conforme o resultado da execução da pipeline.

<b>Estrutura de Notificação de Sucesso:</b>

>>> Além de contar com um diversas informações, ainda há um titulo e uma imagem de feedback

|Informações|Atalhos|
|---|---|
|Marca|Link para a Release|
|Versão|Link para o App Center|
|Ambiente||
|Origem||

<b>Estrutura de Notificação de Falha:</b>

>>> Além de contar com um diversas informações, ainda há um titulo e uma imagem de feedback

|Informações|Atalhos|
|---|---|
|Marca|Mais Informações|
|Versão||
|Ambiente||
|Origem||

Nessa etapa havia um problema, não seria correto deixar informações sensíveis, como <b>WebHook</b>, imagens de feedback (continha imagens de membros da equipe) e demais informações que além de sensiveis, ainda impediam o template de se tornar genérico.
Então a solução foi passar parametros para esse arquivos <b>.sh</b> e fazer ele montar a mensagem através desses parametros fornecidos.

>>> Pode acessar o arquivo [aqui](https://github.com/ArezzoCo/arezzoco-white-label-app/blob/develop/scripts/slack/job-success.sh)

![Variable Group](https://user-images.githubusercontent.com/53791328/156276322-4498e770-71a9-400b-8701-57993a6a5920.png)

Surgiu então outra dúvida: Onde vamos guardar estes dados considerados sensíveis?

Conseguimos usar o <b>Variable Groups</b> dentro da própria plataforma da [Azure](https://dev.azure.com/arezzosa/MOBILIDADE/_library?itemType=VariableGroups&view=VariableGroupView&variableGroupId=1&path=arezzo-staging-variables)

<b>Vantagens:</b>
* Podemos alterar os parametros passados sem alterações no código, sem merge
* Podemos ofuscar as informações sensíveis
* Tornamos o template mais genérico

Então em resumo, guardamos as informações sensíveis ou que podem váriar de app pra app dentro da plataforma da Azure e passamos via parâmetros para dentro do template que fica em um arquivo .sh dentro do projeto de cada App.

## Pipeline:

No contexto das pipelines, tinha duas questões para resolver:
* Como detectar sucesso ou falha?
* Como lidar com escopo nas pipelines

### Como detectar sucesso ou falha

Podemos usar as condições especificadas na documentação da [Azure](https://docs.microsoft.com/pt-br/azure/devops/pipelines/process/conditions?view=azure-devops&tabs=yaml).

![Condicao e Dependencia](https://github.com/cristiantariga/tech-talk/blob/main/notificacoes-slack-pipeline/images/condicao%20e%20dependencia.png?raw=true)

Onde informamos a condição (no nosso caso, sucesso ou falha) e todos os jobs que precisam atender essa condição, para só assim executarmos o job dependente, que seria o job da notificação.

Um ponto interessante que podemos observar, é que existindo dois jobs (um de sucesso e outro de falha), ele apenas executa o que obedecer a condição e dá um skip no job que não atendeu.

![Skip Sucesso e Falha](https://user-images.githubusercontent.com/53791328/156281431-bfae3b25-ec16-4bc0-85e5-b46eabb6b697.png)

### Escopo nas pipelines
Nos primeiros momentos da implementação da notificação nas pipelines, houve alguns problemas com escopo, onde valores extraidos/setados em um job, não eram acessiveis para outro.

Era necessário utilizar [outputs](https://docs.microsoft.com/pt-br/azure/devops/pipelines/process/set-variables-scripts?view=azure-devops&tabs=bash) para conseguir ter acesso a determinados valores de variaveis.

Foi definido ao setar a variavel no job <b>GetCurrentVersionName</b> o isoutput=true

```[task.setvariable variable=currentVersionName;isoutput=true]```

Depois de definir o output, conseguimos obter o valor setado nesse job, dentro do job de sucesso por exemplo:

```$[ dependencies.GetCurrentVersionName.outputs['getVersion.currentVersionName'] ]```

No caso, apontamos como dependecia o <b>GetCurrentVersionName</b> que é o job de origem desse valor, e pegamos o valor do output escolhido, no caso o <b>currentVersionName</b>
