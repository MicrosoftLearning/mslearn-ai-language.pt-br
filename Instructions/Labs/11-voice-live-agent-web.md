---
lab:
  title: Desenvolver um agente de voz com o Voice Live da IA do Azure
  description: Saiba como criar um aplicativo Web para habilitar interações de voz em tempo real com um agente do Voice Live da IA do Azure.
---

# Desenvolver um agente de voz com o Voice Live da IA do Azure

Neste exercício, você irá completar um aplicativo web em Python com Flask que permite interações de voz em tempo real com um agente. Adicione o código para inicializar a sessão e manipule eventos de sessão. Você irá usar um script de implantação que implementa o modelo de IA, cria uma imagem do aplicativo no Registro de Contêiner do Azure (ACR) usando tarefas do ACR e, em seguida, cria uma instância dos Serviço de Aplicativo do Azure que obtém essa imagem. Para testar o aplicativo, você precisará de um dispositivo de áudio com recursos de microfone e alto-falante.

Embora este exercício seja baseado em Python, você pode desenvolver aplicativos semelhantes para outros SDKs específicos da linguagem, incluindo:

- [Biblioteca de clientes VoiceLive do Azure para .NET](https://www.nuget.org/packages/Azure.AI.VoiceLive/)

Tarefas realizadas neste exercício:

* Baixar os arquivos base do aplicativo
* Adicionar código para completar o aplicativo Web
* Examinar a base geral do código
* Atualizar e executar o script de implantação
* Exibir e testar o aplicativo

Este exercício levará aproximadamente **30** minutos para ser concluído.

## Iniciar o Azure Cloud Shell e baixar os arquivos

Nesta seção do exercício, você irá baixar um arquivo compactado que contém os arquivos base do aplicativo.

1. No navegador, vá par o portal do Azure [https://portal.azure.com](https://portal.azure.com). Faça login com suas credenciais do Azure se for solicitado.

1. Use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um novo Cloud Shell no portal do Azure, selecionando um ambiente do ***Bash***. O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure.

    > **Observação**: se você já criou uma Cloud Shell que usa um ambiente *PowerShell*, troque-o pelo ***Bash***.

1. Na barra de ferramentas do Cloud Shell, no menu **Configurações**, selecione **Ir para a versão clássica** (isso é necessário para usar o editor de código).

1. Execute o comando a seguir no shell **Bash** para baixar e descompactar os arquivos do exercício. O segundo comando também irá alterar para o diretório dos arquivos do exercício.

    ```bash
    wget https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/downloads/python/voice-live-web.zip
    ```

    ```
    unzip voice-live-web.zip && cd voice-live-web
    ```

## Adicionar código para completar o aplicativo Web

Agora que os arquivos do exercício estão baixados, a próxima etapa é adicionar código para completar o aplicativo. As etapas a seguir são executadas no Cloud Shell. 

>**Dica:** redimensione o Cloud Shell para exibir mais informações e código arrastando a borda superior. Você também pode usar os botões minimizar e maximizar para alternar entre o Cloud Shell e a interface do portal principal.

Execute o comando a seguir para alterar o diretório *src* antes de continuar com o exercício.

```bash
cd src
```

### Adicionar código para implementar o assistente de Voice Live

Nesta seção, você irá adicionar código para implementar o assistente de Voice Live. O método **\_\_init\_\_** inicializa o assistente de voz armazenando os parâmetros de conexão do VoiceLive do Azure (ponto de extremidade, credenciais, modelo, voz e instruções do sistema) e configurando variáveis de estado em tempo de execução para gerenciar o ciclo de vida da conexão e lidar com interrupções do usuário durante as conversas. O método **start** importa os componentes necessários do SDK do VoiceLive do Azure, que serão usados para estabelecer a conexão WebSocket e configurar a sessão de voz em tempo real.

1. Execute o comando a seguir para abrir o arquivo *flask_app.py* para edição.

    ```bash
    code flask_app.py
    ```

1. Pesquise o comentário **# BEGIN VOICE LIVE ASSISTANT IMPLEMENTATION - ALIGN CODE WITH COMMENT** no código. Copie o código abaixo e insira-o logo abaixo do comentário. Certifique-se de verificar o recuo.

    ```python
    def __init__(
        self,
        endpoint: str,
        credential,
        model: str,
        voice: str,
        instructions: str,
        state_callback=None,
    ):
        # Store Azure Voice Live connection and configuration parameters
        self.endpoint = endpoint
        self.credential = credential
        self.model = model
        self.voice = voice
        self.instructions = instructions
        
        # Initialize runtime state - connection established in start()
        self.connection = None
        self._response_cancelled = False  # Used to handle user interruptions
        self._stopping = False  # Signals graceful shutdown
        self.state_callback = state_callback or (lambda *_: None)

    async def start(self):
        # Import Voice Live SDK components needed for establishing connection and configuring session
        from azure.ai.voicelive.aio import connect  # type: ignore
        from azure.ai.voicelive.models import (
            RequestSession,
            ServerVad,
            AzureStandardVoice,
            Modality,
            InputAudioFormat,
            OutputAudioFormat,
        )  # type: ignore
    ```

1. Insira **ctrl+s** para salvar suas alterações e mantenha o editor aberto para a próxima seção.

### Adicionar código para implementar o assistente de Voice Live

Nesta seção, você irá adicionar código para configurar a sessão do Voice Live. Isso especifica as modalidades (a API não tem suporte para somente áudio), as instruções do sistema que definem o comportamento do assistente, a voz de TTS do Azure para respostas, o formato de áudio para os fluxos de entrada e saída, e a detecção de atividades de voz (VAD) no servidor, que determina como o modelo identifica quando os usuários começam e param de falar.

1. Pesquise o comentário **# BEGIN CONFIGURE VOICE LIVE SESSION - ALIGN CODE WITH COMMENT** no código. Copie o código abaixo e insira-o logo abaixo do comentário. Certifique-se de verificar o recuo.

    ```python
    # Configure VoiceLive session with audio/text modalities and voice activity detection
    session_config = RequestSession(
        modalities=[Modality.TEXT, Modality.AUDIO],
        instructions=self.instructions,
        voice=voice_cfg,
        input_audio_format=InputAudioFormat.PCM16,
        output_audio_format=OutputAudioFormat.PCM16,
        turn_detection=ServerVad(threshold=0.5, prefix_padding_ms=300, silence_duration_ms=500),
    )
    await conn.session.update(session=session_config)
    ```

1. Insira **ctrl+s** para salvar suas alterações e mantenha o editor aberto para a próxima seção.

### Adicionar código para manipular eventos de sessão

Nesta seção, você irá adicionar o código para incluir manipuladores de eventos da sessão do Voice Live. Os manipuladores de eventos respondem aos principais eventos da sessão do VoiceLive durante o ciclo de vida da conversa: **_handle_session_updated** sinaliza quando a sessão está pronta para receber entrada do usuário; **_handle_speech_started** detecta quando o usuário começa a falar e implementa a lógica de interrupção, interrompendo qualquer reprodução de áudio do assistente em andamento e cancelando respostas em progresso para permitir um fluxo de conversa natural; e **_handle_speech_stopped** lida com o momento em que o usuário termina de falar, iniciando o processamento da entrada pelo assistente.

1. Pesquise o comentário **# BEGIN HANDLE SESSION EVENTS - ALIGN CODE WITH COMMENT** no código. Copie o código abaixo e insira-o logo abaixo do comentário, certificando-se de verificar o recuo.

    ```python
    async def _handle_event(self, event, conn, verbose=False):
        """Handle Voice Live events with clear separation by event type."""
        # Import event types for processing different Voice Live server events
        from azure.ai.voicelive.models import ServerEventType
        
        event_type = event.type
        if verbose:
            _broadcast({"type": "log", "level": "debug", "event_type": str(event_type)})
        
        # Route Voice Live server events to appropriate handlers
        if event_type == ServerEventType.SESSION_UPDATED:
            await self._handle_session_updated()
        elif event_type == ServerEventType.INPUT_AUDIO_BUFFER_SPEECH_STARTED:
            await self._handle_speech_started(conn)
        elif event_type == ServerEventType.INPUT_AUDIO_BUFFER_SPEECH_STOPPED:
            await self._handle_speech_stopped()
        elif event_type == ServerEventType.RESPONSE_AUDIO_DELTA:
            await self._handle_audio_delta(event)
        elif event_type == ServerEventType.RESPONSE_AUDIO_DONE:
            await self._handle_audio_done()
        elif event_type == ServerEventType.RESPONSE_DONE:
            # Reset cancellation flag but don't change state - _handle_audio_done already did
            self._response_cancelled = False
        elif event_type == ServerEventType.ERROR:
            await self._handle_error(event)

    async def _handle_session_updated(self):
        """Session is ready for conversation."""
        self.state_callback("ready", "Session ready. You can start speaking now.")

    async def _handle_speech_started(self, conn):
        """User started speaking - handle interruption if needed."""
        self.state_callback("listening", "Listening… speak now")
        
        try:
            # Stop any ongoing audio playback on the client side
            _broadcast({"type": "control", "action": "stop_playback"})
            
            # If assistant is currently speaking or processing, cancel the response to allow interruption
            current_state = assistant_state.get("state")
            if current_state in {"assistant_speaking", "processing"}:
                self._response_cancelled = True
                await conn.response.cancel()
                _broadcast({"type": "log", "level": "debug", 
                          "msg": f"Interrupted assistant during {current_state}"})
            else:
                _broadcast({"type": "log", "level": "debug", 
                          "msg": f"User speaking during {current_state} - no cancellation needed"})
        except Exception as e:
            _broadcast({"type": "log", "level": "debug", 
                      "msg": f"Exception in speech handler: {e}"})

    async def _handle_speech_stopped(self):
        """User stopped speaking - processing input."""
        self.state_callback("processing", "Processing your input…")

    async def _handle_audio_delta(self, event):
        """Stream assistant audio to clients."""
        if self._response_cancelled:
            return  # Skip cancelled responses
            
        # Update state when assistant starts speaking
        if assistant_state.get("state") != "assistant_speaking":
            self.state_callback("assistant_speaking", "Assistant speaking…")
        
        # Extract and broadcast Voice Live audio delta as base64 to WebSocket clients
        audio_data = getattr(event, "delta", None)
        if audio_data:
            audio_b64 = base64.b64encode(audio_data).decode("utf-8")
            _broadcast({"type": "audio", "audio": audio_b64})

    async def _handle_audio_done(self):
        """Assistant finished speaking."""
        self._response_cancelled = False
        self.state_callback("ready", "Assistant finished. You can speak again.")

    async def _handle_error(self, event):
        """Handle Voice Live errors."""
        error = getattr(event, "error", None)
        message = getattr(error, "message", "Unknown error") if error else "Unknown error"
        self.state_callback("error", f"Error: {message}")

    def request_stop(self):
        self._stopping = True
    ```

1. Insira **ctrl+s** para salvar suas alterações e mantenha o editor aberto para a próxima seção.

### Examinar o código no aplicativo

Até agora, você adicionou código ao aplicativo para implementar o agente e manipular eventos do agente. Reserve alguns minutos para revisar o código completo e os comentários, a fim de compreender melhor como o aplicativo está gerenciando o estado do cliente e suas operações.

1. Quando terminar, pressione **ctrl+q** para sair do editor. 

## Atualizar e executar o script de implantação

Nesta seção, você irá fazer uma pequena alteração no script de implantação **azdeploy.sh** e, em seguida, executará a implantação. 

### Atualizar o script de implantação

Há apenas dois valores que você deve alterar na parte superior do script de implantação **azdeploy.sh**. 

* O valor **rg** especifica o grupo de recursos que irá conter a implantação. Você pode aceitar o valor padrão ou inserir seu próprio valor se precisar implantar em um grupo de recursos específico.

* O valor **location** define a região para a implantação. O modelo *gpt-4o* usado neste exercício pode ser implantado em outras regiões, mas pode haver limites em regiões específicas. Se a implantação falhar na região escolhida, experimente **eastus2** ou **swedencentral**. 

    ```
    rg="rg-voicelive" # Replace with your resource group
    location="eastus2" # Or a location near you
    ```

1. Execute os comandos a seguir no Cloud Shell para começar a editar o script de implantação.

    ```bash
    cd ~/voice-live-web
    ```
    
    ```bash
    code azdeploy.sh
    ```

1. Atualize os valores de **rg** e **location** conforme suas necessidades, em seguida pressione **ctrl+s** para salvar as alterações e **ctrl+q** para sair do editor.

### Executar o script de implantação

O script de implantação implementa o modelo de IA e cria os recursos necessários no Azure para executar um aplicativo conteinerizado no Serviço de Aplicativo.

1. Execute o comando a seguir no Cloud Shell para começar a implantar os recursos do Azure e o aplicativo.

    ```bash
    bash azdeploy.sh
    ```

1. Selecione a **opção 1** para a implantação inicial.

    A implantação deve ser concluída em 5 a 10 minutos. Durante a implantação, você pode ser solicitado a fornecer as seguintes informações ou realizar as seguintes ações:
    
    * Se for solicitado que você se autentique no Azure, siga as instruções apresentadas a você.
    * Se for solicitado que você selecione uma assinatura, use as setas para destacar sua assinatura e pressione **Enter**. 
    * É provável que você veja alguns avisos durante a implantação; eles podem ser ignorados.
    * Se a implantação falhar durante a implantação do modelo de IA, altere a região no script de implantação e tente novamente. 
    * As regiões no Azure podem ficar congestionadas em determinados momentos, o que pode afetar o tempo das implantações. Se a implantação falhar após a implantação do modelo, execute novamente o script de implantação.

## Exibir e testar o aplicativo

Quando a implantação for concluída, a mensagem "Deployment complete!" aparecerá no shell, juntamente com um link para o aplicativo Web. Você pode selecionar esse link ou navegar até o recurso do Serviço de Aplicativo e iniciar o aplicativo por lá. Pode levar alguns minutos para o aplicativo carregar. 

1. Selecione o botão **Iniciar sessão** para se conectar ao modelo.
1. Você será solicitado a conceder ao aplicativo acesso aos seus dispositivos de áudio.
1. Comece a falar com o modelo quando o aplicativo solicitar que você comece a falar.

Solucionar problemas:

* Se o aplicativo informar variáveis de ambiente ausentes, reinicie-o no Serviço de Aplicativo.
* Se você notar mensagens excessivas de *audio chunk* no log exibido pelo aplicativo, selecione **Parar sessão** e, em seguida, inicie a sessão novamente. 
* Se o aplicativo não funcionar de forma alguma, verifique se todo o código foi adicionado corretamente e se o recuo está correto. Se você precisar fazer alterações, execute novamente a implantação e selecione a **opção 2** para apenas atualizar a imagem.

## Limpar os recursos

Execute o seguinte comando no Cloud Shell para remover todos os recursos implantados neste exercício. Você será solicitado a confirmar a exclusão do recurso.

```
azd down --purge
```