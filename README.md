# Chatbot de Clima do Telegram 🌦️

Projeto do Bot de Clima no Telegram criado utilizando **n8n**! Este workflow conecta a sua conta do Telegram a um bot que informa a temperatura atual de qualquer cidade via a API do **OpenWeather**, além de contar com a melhoria opcional das respostas com **Google Gemini**.

## 🚀 Como Funciona

1. O bot recebe o nome de uma cidade (ex: "São Paulo,SP,BR") via uma mensagem do Telegram.
2. O workflow remove excessos de espaço e formata a pesquisa para a chamada na API de clima.
3. Aciona a OpenWeather API consultando a temperatura.
4. Gera uma resposta em caso de sucesso (com arredondamentos) ou emite uma mensagem de erro padronizada (`❌ Cidade não encontrada. Use o formato Cidade,UF,BR`).
5. **Integração opcional de Inteligência Artificial:** Aciona o Google Gemini para reescrever a mensagem de maneira mais humana. Caso o Gemini não receba a credencial, um gerador estático (fallback) envia a resposta com assertividade.

## 📥 Como importar o workflow

Dentro do seu painel do n8n:
1. No menu esquerdo, acesse **Workflows** e clique em **Add Workflow**.
2. No menu superior direito da tela de edição do workflow, acesse **[•••] > Import from file**.
3. Selecione o arquivo `workflow-chatbot-telegram.json` incluído neste repositório.
4. Aguarde o fluxo carregar em sua tela.

## 🔐 Configurando Credenciais

Para que o projeto funcione adequadamente, você precisará preencher as credenciais diretamente no painel do N8N.

### 1. Telegram Bot
1. Crie seu bot com o `@BotFather` no Telegram e copie o Token gerado.
2. Acesse no painel esquerdo do n8n: **Credentials > Add Credential**.
3. Procure por `Telegram API` e clique para criar.
4. Na tela de configuração desta credencial, no campo **Access Token**, cole o token gerado diretamente.
5. Salve a credencial, por exemplo com o nome "Telegram Weather Bot".
6. **Atenção:** É perfeitamente normal que o n8n mostre um campo com o texto "Select Credential" nos nós do Telegram. Basta você clicar nesse *dropdown* nos 3 nós (Telegram Trigger, Envia Sucesso e Envia Erro) e selecionar a credencial "Telegram Weather Bot" que você acabou de criar.

### 2. OpenWeather API
1. Crie uma conta no `home.openweathermap.org` e pegue sua API Key.
2. No N8N, abra o nó **"OpenWeather API"** e, em **Authentication**, mude para **Generic Credential Type**.
3. Em **Generic Auth Type**, selecione **Query Auth**.
4. Clique para criar uma nova credencial e renomeie-a para `Openweather Api Key` (ou outro nome de sua preferência).
5. Na janela da credencial, defina o **Name** como `appid` e no **Value** cole o seu TOKEN do OpenWeather. Salve e selecione essa credencial no nó.

### 3. Google Gemini API (Opcional)
Esse passo foi configurado com um Node Langchain para atuar como formatador natural:
1. Crie ou recupere sua chave de API gratuitamente em `aistudio.google.com/app/apikey`.
2. No n8n em: **Credentials > Add Credential > Google API**. Insira a credencial gerada.
3. Feito isso, conecte a credencial criada no nó do **Google Gemini**.

## ✅ Testando e Executando o Chatbot

Para começar as consultas em fase de testes (Development/Testing):

1. **Ative a escuta (listening):** Clique em **"Execute Workflow"** na parte inferior da tela do seu n8n.
2. **Envie a Cidade:** No seu aplicativo Telegram, vá até a conversa com o bot recém-criado e envie uma cidade para testar.
   - **Exemplo de Envio:** `Belo Horizonte,MG,BR` ou `São Paulo,SP,BR`.
3. **O que esperar de retorno no Sucesso:** Em média de 2 segundos, pelo menos o nó deverá carregar e responder com sucesso: 
   _🌤 A temperatura em São Paulo é de 25°C_ (Podendo vir reescrito de forma mais descontraída caso a API do Gemini esteja configurada).
4. **O que esperar de retorno no Erro:** Caso você envie uma cidade fictícia (ex: `LugarNenhum`), o bot enviará a mensagem padrão:
   _❌ Cidade não encontrada. Use o formato Cidade,UF,BR (ex.: São Paulo,SP,BR)._

**Para manter online 24/7:**
Quando os testes derem certo, lembre-se de ativar o gatilho principal marcando a chave **Active** (no canto superior direito do workflow). Assim ele responderá sempre que uma pessoa chamar, sem precisar clicar no botão de teste.

---

> **⚠️ Nota Técnica: O Problema das Variáveis de Ambiente no N8N**
> 
> Durante o desenvolvimento deste projeto, constatei que não é possível injetar variáveis de ambiente do Docker diretamente no *Credentials Manager* utilizando expressões (ex: `={{ $env.TELEGRAM_BOT_TOKEN }}`). 
> 
> Isso ocorre porque o núcleo de segurança do N8N avalia propositalmente essas expressões de ambiente como `undefined` no escopo das credenciais geradas pela UI, visando impedir ataques de roubo de injeção em instâncias hospedadas em nuvem. Para contornar cenários em que as credenciais devem obrigatoriamente derivar de um contêiner automatizado ou `.env`, a solução sustentável (não demonstrada aqui) seria utilizar o recurso `CREDENTIALS_OVERWRITE_DATA` do n8n dentro do `docker-compose.yml`. Como esta via exige alteração avançada de variáveis, a abordagem simplificada escolhida foi a inserção manual orientada nas etapas acima.
