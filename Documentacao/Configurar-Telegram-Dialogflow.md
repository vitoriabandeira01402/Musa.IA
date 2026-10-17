# Configurar Telegram + Dialogflow ES

Este guia liga uma conversa no Telegram ao Dialogflow ES usando n8n. O fluxo usa o ID do chat como sessão do Dialogflow para manter o contexto de cada pessoa separado.

```text
Telegram → n8n Telegram Trigger → Dialogflow ES detectIntent → n8n Telegram → resposta no mesmo chat
```

O workflow importável está em [`integracoes/musa-telegram-dialogflow.json`](../integracoes/musa-telegram-dialogflow.json). Ele vem desativado e sem credenciais. É necessário concluir os passos abaixo antes de ativá-lo.

## 1. Criar o bot do Telegram

1. Abra o Telegram e converse com [@BotFather](https://t.me/BotFather).
2. Envie `/newbot`, escolha um nome e um username disponível.
3. Guarde o token em local seguro. **Não coloque o token no GitHub, no workflow JSON ou nesta conversa.**

## 2. Preparar o Dialogflow ES

1. Crie ou selecione um projeto no [Google Cloud Console](https://console.cloud.google.com/).
2. Ative a **Dialogflow API** no projeto.
3. Crie um agente no [Dialogflow ES](https://dialogflow.cloud.google.com/) usando português do Brasil (`pt-BR`).
4. Configure os intents usando os blocos prontos da próxima seção. Em cada intent, cole as frases em **Training phrases** (uma por linha) e o texto indicado em **Responses → Text response**.
5. Anote o **Project ID** do Google Cloud. O ID é usado na URL do nó HTTP do workflow.

> O Dialogflow ES faz interpretação de linguagem por intents e entidades; ele não é, por si só, um modelo generativo nem um catálogo musical. As respostas precisam ser configuradas no agente. Para recomendações dinâmicas de catálogo, será necessário adicionar uma fonte de músicas e regras de recomendação ao fluxo.

## Intents prontos para copiar

No console, abra **Intents → Create Intent**, dê o nome indicado e adicione as frases de treinamento. Em **Responses**, escolha **Default**, adicione **Text Response** e cole a resposta. Salve cada intent.

### `BoasVindas_Musa`

**Training phrases:**

```text
Oi
Olá
Oi Musa
Tudo bem?
Quero descobrir músicas novas
```

**Text response:**

```text
Oi! Eu sou a Musa.IA 🎵 Posso sugerir caminhos musicais a partir do seu estilo ou de uma música que você goste. Você prefere MPB, indie, pop, rock ou R&B?
```

### `Recomendacao_MPB`

**Training phrases:**

```text
Me recomenda MPB
Quero ouvir música brasileira
Gosto de MPB suave
Me indica uma música brasileira tranquila
Quero descobrir artistas de MPB
```

**Text response:**

```text
Se você está no clima de MPB suave, experimente Partilhar, do Rubel; Velha Infância, dos Tribalistas; ou O Vento, do Los Hermanos. Qual dessas combina mais com você?
```

### `Recomendacao_Indie`

**Training phrases:**

```text
Me recomenda indie
Quero um indie leve para relaxar
Gosto de rock alternativo
Me indica uma banda indie
Quero descobrir músicas indie parecidas com Cranberries
```

**Text response:**

```text
Para explorar indie e dream pop, experimente Bags, da Clairo; Fade Into You, do Mazzy Star; ou Dreams, do The Cranberries. Depois me conta qual você curtiu mais.
```

### `Recomendacao_Pop_RnB`

**Training phrases:**

```text
Me recomenda pop
Quero ouvir R&B
Gosto de SZA
Quero um pop tranquilo
Me indica uma música com uma batida suave
```

**Text response:**

```text
Se você gosta de pop e R&B suave, experimente Snooze ou Good Days, da SZA, e Bags, da Clairo. Quer continuar por esse clima ou tentar outro estilo?
```

### `Musica_Parecida`

**Training phrases:**

```text
Quero músicas parecidas com Linger do The Cranberries
Me indica algo parecido com Dreams do Fleetwood Mac
Gosto de Bags da Clairo, o que mais posso ouvir?
Quero descobrir músicas com o clima de Fade Into You
Que música combina com Partilhar do Rubel?
```

Em cada frase, selecione com o mouse o trecho que identifica a música ou artista, associe a entidade **`@sys.any`** e dê ao parâmetro o nome **`musica_referencia`**. Faça essa marcação nas frases individualmente para que o parâmetro seja preenchido.

**Text response:**

```text
Anotei "$musica_referencia" como referência 🎵 Nesta configuração inicial ainda não tenho um catálogo musical conectado para calcular semelhanças reais. Para começar, experimente Fade Into You, do Mazzy Star; Dreams, do The Cranberries; ou Bags, da Clairo. Qual caminho você prefere?
```

### `Default Fallback Intent`

Abra o intent de fallback padrão do agente e substitua o texto de resposta por:

```text
Ainda não consegui encontrar o caminho musical 🎶 Você pode me dizer um estilo, como MPB ou indie, ou escrever “quero músicas parecidas com” e o nome de uma faixa.
```

> **Importante:** essas respostas fazem o Dialogflow classificar mensagens reais, mas as listas acima são fixas. A entity `@sys.any` só captura o nome que a pessoa escreveu; ela não pesquisa áudio nem calcula similaridade. Para recomendações personalizadas de verdade, será preciso adicionar fulfillment no Dialogflow/n8n e conectá-lo a um catálogo ou serviço de recomendação.

## 3. Configurar as credenciais no n8n

Use uma instância n8n acessível pela internet com HTTPS. O Telegram precisa alcançar o webhook público do n8n; uma instalação local sem túnel público não recebe atualizações.

1. Importe [`musa-telegram-dialogflow.json`](../integracoes/musa-telegram-dialogflow.json) em **Workflows → Import from File**.
2. No nó **Telegram Trigger**, crie ou selecione uma credencial **Telegram API** e informe o token do BotFather. Use a mesma credencial no nó **Responder no Telegram**.
3. No nó **Dialogflow ES - Detect Intent**, substitua `SEU_PROJECT_ID` na URL pelo Project ID do Google Cloud.
4. No Google Cloud, crie uma service account com permissão **Dialogflow API Client** no projeto e gere uma chave JSON. No n8n, crie a credencial **Google Service Account API** com o e-mail e a chave privada dessa service account. Ative **Set up for use in HTTP Request node** e informe o escopo `https://www.googleapis.com/auth/dialogflow`.
5. Selecione essa credencial Google no nó **Dialogflow ES - Detect Intent**. Tokens e chaves ficam guardados nas credenciais do n8n, não no arquivo exportado.
6. Salve e ative o workflow. O Telegram permite apenas um webhook ativo por bot; não use o mesmo token em outro workflow com Telegram Trigger.

## 4. Testar

1. Abra o link `https://t.me/SEU_BOT_USERNAME`, substituindo o trecho pelo username criado no BotFather.
2. Toque em **Start** e envie uma mensagem de texto, por exemplo: “Gosto de MPB suave, me sugira músicas”.
3. Verifique a execução no n8n e confirme que a resposta configurada no fulfillment do Dialogflow chega ao mesmo chat.

O site da Musa.IA ainda contém apenas uma demonstração local no navegador. Para que o botão do site abra o bot, atualize o link Telegram depois de criar o username. Não coloque token de bot ou chave de serviço na página pública.

## Referências oficiais

- [Telegram Trigger no n8n](https://docs.n8n.io/integrations/builtin/trigger-nodes/n8n-nodes-base.telegramtrigger/)
- [Telegram no n8n](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.telegram/)
- [Credencial Google Service Account no n8n](https://docs.n8n.io/integrations/builtin/credentials/google/service-account/)
- [Dialogflow ES detectIntent API](https://cloud.google.com/dialogflow/es/docs/reference/rest/v2/projects.agent.sessions/detectIntent)
