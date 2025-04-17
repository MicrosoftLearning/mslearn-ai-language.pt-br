---
lab:
  title: Reconhecer e sintetizar Fala (versão da Fábrica de IA do Azure)
  module: Module 4 - Create speech-enabled apps with Azure AI services
---

<!--
Possibly update to use standalone AI Service instead of Foundry?
-->

# Reconhecer e sintetizar fala

A **Fala de IA do Azure** é um serviço que fornece funcionalidade relacionada à fala, incluindo:

- Uma API de *reconhecimento de fala* que permite implementar o reconhecimento de fala (conversão de palavras faladas audíveis em texto).
- Uma API de *conversão de texto em fala* que permite implementar a sintetização da fala (convertendo texto em fala audível).

Neste exercício, você usará essas duas APIs para implementar um aplicativo de relógio falante.

> **OBSERVAÇÃO** Este exercício foi elaborado para ser feito no Azure Cloud Shell, onde não há suporte para acesso direto ao hardware de som do computador. Portanto, o laboratório usará arquivos de áudio para fluxos de entrada e saída de fala. O código para obter os mesmos resultados usando um microfone e alto-falante é fornecido para sua referência.

## Criar um projeto do Azure AI Foundry

Vamos começar criando um projeto da Fábrica de IA do Azure.

1. Em um navegador da Web, abra o [Portal da Fábrica de IA do Azure](https://ai.azure.com) em `https://ai.azure.com` e entre usando suas credenciais do Azure. Feche todas as dicas ou painéis de início rápido abertos na primeira vez que você entrar e, se necessário, use o logotipo da **Fábrica de IA do Azure** no canto superior esquerdo para navegar até a home page, que é semelhante à imagem a seguir:

    ![Captura de tela do portal do Azure AI Foundry.](./ai-foundry/media/ai-foundry-home.png)

1. Na home page, selecione **+Criar projeto**.
1. No assistente **Criar um projeto**, insira um nome de projeto adequado e, se um hub existente for sugerido, escolha a opção de criar um novo. Em seguida, examine os recursos do Azure que serão criados automaticamente para dar suporte ao hub e ao projeto.
1. Selecione **Personalizar** e especifique as seguintes configurações para o hub:
    - **Nome do hub**: *um nome para o hub*
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *criar ou selecionar um grupo de recursos*
    - **Localização**: escolha qualquer região disponível
    - **Conectar os Serviços de IA do Azure ou o OpenAI do Azure** – *Criar um novo recurso de Serviços de IA*
    - **Conectar a Pesquisa de IA do Azure**: *crie um novo recurso da Pesquisa de IA do Azure com um nome exclusivo*

1. Clique em **Avançar** e revise a configuração. Em seguida, selecione **Criar** e aguarde a conclusão do processo.
1. Quando o projeto for criado, feche todas as dicas exibidas e examine a página do projeto no Portal da Fábrica de IA do Azure, que deve ser semelhante à imagem a seguir:

    ![Captura de tela dos detalhes de um projeto IA do Azure no Portal da Fábrica de IA do Azure.](./ai-foundry/media/ai-foundry-project.png)

## Preparar e configurar o aplicativo de relógio falante

1. No Portal da Fábrica de IA do Azure, visualize a página **Visão geral** do seu projeto.
1. Na área **Detalhes do projeto**, observe a **cadeia de conexão do projeto** e o **local** do seu projeto. Você usará a cadeia de conexão para se conectar ao seu projeto em um aplicativo cliente e precisará do local para se conectar ao ponto de extremidade de Fala dos Serviços de IA do Azure.
1. Abra uma nova guia do navegador (mantendo o portal da Fábrica de IA do Azure aberto na guia existente). Em seguida, na nova guia, navegue até o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com`; efetue login com suas credenciais do Azure, se solicitado.
1. Use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um Cloud Shell no portal do Azure selecionando um ambiente do ***PowerShell***. O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure.

    > **Observação**: se você já criou um Cloud Shell que usa um ambiente *Bash*, alterne-o para o ***PowerShell***.

1. Na barra de ferramentas do Cloud Shell, no menu **Configurações**, selecione **Ir para a versão clássica** (isso é necessário para usar o editor de código).

    > **Dica**: conforme você colar comandos no cloudshell, a saída pode ocupar uma grande quantidade do espaço da tela. Você pode limpar a tela digitando o comando `cls` para facilitar o foco em cada tarefa.

1. No painel do PowerShell, insira os seguintes comandos para clonar o repositório GitHub para este exercício:

    ```
   rm -r mslearn-ai-language -f
   git clone https://github.com/microsoftlearning/mslearn-ai-language mslearn-ai-language
    ```

    ***Agora siga as etapas para a linguagem de programação escolhida.***

1. Após o repositório ser clonado, navegue até a pasta que contém os arquivos de código do aplicativo do relógio falante:  

    **Python**

    ```
   cd mslearn-ai-language/labfiles/07b-speech/python/speaking-clock
    ```

    **C#**

    ```
   cd mslearn-ai-language/labfiles/07b-speech/c-sharp/speaking-clock
    ```

1. No painel de linha de comando do Cloud Shell, digite o seguinte comando para instalar as bibliotecas que você usará:

    **Python**

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity azure-ai-projects azure-cognitiveservices-speech==1.42.0
    ```

    **C#**

    ```
   dotnet add package Azure.Identity
   dotnet add package Azure.AI.Projects --prerelease
   dotnet add package Microsoft.CognitiveServices.Speech --version 1.42.0
    ```

1. Digite o seguinte comando para editar o arquivo de configuração que foi fornecido:

    **Python**

    ```
   code .env
    ```

    **C#**

    ```
   code appsettings.json
    ```

    O arquivo é aberto em um editor de código.

1. No arquivo de código, substitua os espaços reservados **your_project_endpoint** e **your_location** pela cadeia de conexão e localização do seu projeto (copiada da página **Visão Geral** no portal da Fábrica de IA do Azure).
1. Depois de substituir os espaços reservados, use o comando **CTRL+S** para salvar suas alterações e, em seguida, use o comando **CTRL+Q** para fechar o editor de código, mantendo a linha de comando do Cloud Shell aberta.

## Adicionar código para usar o SDK de Fala de IA do Azure

> **Dica**: ao adicionar código, certifique-se de manter o recuo correto.

1. Digite o seguinte comando para editar o arquivo de código que foi fornecido:

    **Python**

    ```
   code speaking-clock.py
    ```

    **C#**

    ```
   code Program.cs
    ```

1. Na parte superior do arquivo de código, sob as referências de namespace existentes, localize o comentário **Import namespaces**. Em seguida, neste comentário, adicione o seguinte código específico de linguagem para importar os namespaces que você precisará para usar o SDK de Fala de IA do Azure com o recurso Serviços de IA do Azure no projeto da Fábrica de IA do Azure:

    **Python**

    ```python
   # Import namespaces
   from dotenv import load_dotenv
   from azure.ai.projects.models import ConnectionType
   from azure.identity import DefaultAzureCredential
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.projects import AIProjectClient
   import azure.cognitiveservices.speech as speech_sdk
    ```

    **C#**

    ```csharp
   // Import namespaces
   using Azure.Identity;
   using Azure.AI.Projects;
   using Microsoft.CognitiveServices.Speech;
   using Microsoft.CognitiveServices.Speech.Audio;
    ```

1. Na função **main**, no comentário **Get config settings**, observe que o código carrega a cadeia de conexão do projeto e o local que você definiu no arquivo de configuração.

1. Adicione o seguinte código no comentário **Get AI Speech endpoint and key from the project**:

    **Python**

    ```python
   # Get AI Services key from the project
   project_client = AIProjectClient.from_connection_string(
        conn_str=project_connection,
        credential=DefaultAzureCredential())

   ai_svc_connection = project_client.connections.get_default(
      connection_type=ConnectionType.AZURE_AI_SERVICES,
      include_credentials=True, 
    )

   ai_svc_key = ai_svc_connection.key

    ```

    **C#**

    ```csharp
   // Get AI Services key from the project
   var projectClient = new AIProjectClient(project_connection,
                        new DefaultAzureCredential());

   ConnectionResponse aiSvcConnection = projectClient.GetConnectionsClient().GetDefaultConnection(ConnectionType.AzureAIServices, true);

   var apiKeyAuthProperties = aiSvcConnection.Properties as ConnectionPropertiesApiKeyAuth;

   var aiSvcKey = apiKeyAuthProperties.Credentials.Key;
    ```

    Esse código se conecta ao projeto da Fábrica de IA do Azure, obtém seu recurso conectado padrão dos Serviços de IA e recupera a chave de autenticação necessária para usá-lo.

1. No comentário **Configure speech service**, adicione o código a seguir para usar a chave dos Serviços de IA e a região do projeto para configurar sua conexão com o ponto de extremidade de Fala dos Serviços de IA do Azure

   **Python**

    ```python
   # Configure speech service
   speech_config = speech_sdk.SpeechConfig(ai_svc_key, location)
   print('Ready to use speech service in:', speech_config.region)
    ```

    **C#**

    ```csharp
   // Configure speech service
   speechConfig = SpeechConfig.FromSubscription(aiSvcKey, location);
   Console.WriteLine("Ready to use speech service in " + speechConfig.Region);
    ```

1. Salve as alterações (*CTRL+S*), mas deixe o editor de código aberto.

## Executar o aplicativo

Até agora, o aplicativo apenas se conecta ao seu projeto da Fábrica de IA do Azure para recuperar os detalhes necessários para usar o serviço de Fala, mas é útil executá-lo e confirmar se ele funciona antes de adicionar a funcionalidade de fala.

1. Na linha de comando abaixo do editor de código, insira o seguinte comando da CLI do Azure para determinar a conta do Azure que está conectada para a sessão:

    ```
   az account show
    ```

    A saída JSON resultante incluirá detalhes da sua conta do Azure e da assinatura em que você está trabalhando (que deve ser a mesma assinatura na qual você criou seu projeto da Fábrica de IA do Azure).

    Seu aplicativo usa as credenciais do Azure para o contexto no qual ele é executado para autenticar a conexão com seu projeto. Em um ambiente de produção, o aplicativo pode ser configurado para ser executado usando uma identidade gerenciada. Nesse ambiente de desenvolvimento, ele usará suas credenciais de sessão autenticadas do Cloud Shell.

    > **Observação**: você pode entrar no Azure em seu ambiente de desenvolvimento usando o comando da CLI do Azure `az login`. Nesse caso, o Cloud Shell já se conectou usando as credenciais do Azure com as quais você entrou no portal; portanto, entrar explicitamente é desnecessário. Para saber mais sobre como usar a CLI do Azure para autenticar no Azure, consulte [Autenticar no Azure usando a CLI do Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli).

1. Na linha de comando, insira o seguinte comando específico da linguagem para executar o aplicativo de relógio falante:

    **Python**

    ```
   python speaking-clock.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. Se você estiver usando C#, poderá ignorar avisos sobre o uso do operador **await** em métodos assíncronos – corrigiremos isso mais tarde. O código deve exibir a região do recurso de serviço de fala que o aplicativo usará. Uma execução bem-sucedida indica que o aplicativo se conectou ao projeto da Fábrica de IA do Azure e recuperou a chave necessária para usar o serviço de Fala de IA do Azure.

## Adicionar código para reconhecer a fala

Agora que você tem um **SpeechConfig** para o serviço de fala no recurso Serviços de IA do Azure do projeto, você pode usar a API de **reconhecimento de fala** para reconhecer a fala e transcrevê-la para texto.

Neste procedimento, a entrada de fala é capturada de um arquivo de áudio, que você pode reproduzir aqui:

<video controls src="ai-foundry/media/Time.mp4" title="Que horas são?" width="150"></video>

1. Na função **Principal**, observe que o código usa a função **TranscribeCommand** para aceitar entrada falada. Em seguida, na função **TranscribeCommand**, sob o comentário **Configurar reconhecimento de fala**, adicione o código apropriado abaixo para criar um cliente **SpeechRecognizer** que possa ser usado para reconhecer e transcrever a fala de um arquivo de áudio:

    **Python**

    ```python
   # Configure speech recognition
   current_dir = os.getcwd()
   audioFile = current_dir + '/time.wav'
   audio_config = speech_sdk.AudioConfig(filename=audioFile)
   speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    ```

    **C#**

    ```csharp
   // Configure speech recognition
   string audioFile = "time.wav";
   using AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
   using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
    ```

1. Na função **TranscribeCommand**, sob o comentário **Processar entrada de fala**, adicione o seguinte código para ouvir a entrada falada, tomando cuidado para não substituir o código no fim da função que retorna o comando:

    **Python**

    ```python
   # Process speech input
   print("Listening...")
   speech = speech_recognizer.recognize_once_async().get()
   if speech.reason == speech_sdk.ResultReason.RecognizedSpeech:
       command = speech.text
       print(command)
   else:
       print(speech.reason)
       if speech.reason == speech_sdk.ResultReason.Canceled:
           cancellation = speech.cancellation_details
           print(cancellation.reason)
           print(cancellation.error_details)
    ```

    **C#**

    ```csharp
   // Process speech input
   Console.WriteLine("Listening...");
   SpeechRecognitionResult speech = await speechRecognizer.RecognizeOnceAsync();
   if (speech.Reason == ResultReason.RecognizedSpeech)
   {
       command = speech.Text;
       Console.WriteLine(command);
   }
   else
   {
       Console.WriteLine(speech.Reason);
       if (speech.Reason == ResultReason.Canceled)
       {
           var cancellation = CancellationDetails.FromResult(speech);
           Console.WriteLine(cancellation.Reason);
           Console.WriteLine(cancellation.ErrorDetails);
       }
   }
    ```

1. Salve as alterações (*CTRL+S*) e, em seguida, na linha de comando abaixo do editor de código, insira o seguinte comando para executar o programa:

    **Python**

    ```
   python speaking-clock.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. Revise a saída do aplicativo, que "ouvirá" a fala no arquivo de áudio e retornará uma resposta apropriada (observe que o Azure Cloud Shell pode estar em execução em um servidor que está em um fuso horário diferente do seu!)

    > **Dica**: se o SpeechRecognizer encontrar um erro, ele produzirá um resultado de "Cancelado". O código no aplicativo exibirá a mensagem de erro. A causa mais provável é uma região ou valor incorreto no arquivo de configuração.

## Sintetizar fala

Seu aplicativo de relógio de fala aceita entrada falada, mas na verdade não fala! Vamos corrigir isso adicionando código para sintetizar a fala.

Mais uma vez, devido às limitações de hardware do Cloud Shell, direcionaremos a saída de fala sintetizada para um arquivo.

1. Na função **Principal** do programa, observe que o código usa a função **TellTime** para informar ao usuário a hora atual.
1. Na função **TellTime**, sob o comentário **Configurar síntese de fala**, adicione o seguinte código para criar um cliente **SpeechSynthesizer** que pode ser usado para gerar saída falada:

    **Python**

    ```python
   # Configure speech synthesis
   output_file = "output.wav"
   speech_config.speech_synthesis_voice_name = "en-GB-RyanNeural"
   audio_config = speech_sdk.audio.AudioConfig(filename=output_file)
   speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config, audio_config,)
    ```

    **C#**

    ```csharp
   // Configure speech synthesis
   var outputFile = "output.wav";
   speechConfig.SpeechSynthesisVoiceName = "en-GB-RyanNeural";
   using var audioConfig = AudioConfig.FromWavFileOutput(outputFile);
   using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig, audioConfig);
    ```

1. Na função **TellTime**, sob o comentário **Sintetizar saída falada**, adicione o seguinte código para gerar saída falada, tomando cuidado para não substituir o código no final da função que imprime a resposta:

    **Python**

    ```python
   # Synthesize spoken output
   speak = speech_synthesizer.speak_text_async(response_text).get()
   if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
       print(speak.reason)
   else:
       print("Spoken output saved in " + outputFile)
    ```

    **C#**

    ```csharp
   // Synthesize spoken output
   SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(responseText);
   if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
   {
       Console.WriteLine(speak.Reason);
   }
   else
   {
       Console.WriteLine("Spoken output saved in " + outputFile);
   }
    ```

1. Salve as alterações (*CTRL+S*) e, em seguida, na linha de comando abaixo do editor de código, insira o seguinte comando para executar o programa:

   **Python**

    ```
   python speaking-clock.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. Revise a saída do aplicativo, o que deve indicar que a saída falada foi salva em um arquivo.
1. Se você tiver um reprodutor multimídia capaz de reproduzir arquivos de áudio .wav, na barra de ferramentas do painel do Cloud Shell, use o botão **Carregar/Baixar arquivos** para baixar o arquivo de áudio da pasta do aplicativo e reproduzi-lo:

    **Python**

    /home/*user*`/mslearn-ai-language/Labfiles/07b-speech/Python/speaking-clock/output.wav`

    **C#**

    /home/*user*`/mslearn-ai-language/Labfiles/07b-speech/C-Sharp/speaking-clock/output.wav`

    O arquivo terá um som como este:

    <video controls src="./ai-foundry/media/Output.mp4" title="O tempo é 2:15" width="150"></video>

## Usar a Linguagem de Marcação de Sintetização de Voz

O SSML (Linguagem de Marcação de Sintetização de Fala) permite personalizar a maneira como sua fala é sintetizada usando um formato baseado em XML.

1. Na função **TellTime**, substitua todo o código atual sob o comentário **Sintetizar saída falada** com o seguinte código (deixe o código sob o comentário **Imprimir a resposta**):

    **Python**

    ```python
   # Synthesize spoken output
   responseSsml = " \
       <speak version='1.0' xmlns='http://www.w3.org/2001/10/synthesis' xml:lang='en-US'> \
           <voice name='en-GB-LibbyNeural'> \
               {} \
               <break strength='weak'/> \
               Time to end this lab! \
           </voice> \
       </speak>".format(response_text)
   speak = speech_synthesizer.speak_ssml_async(responseSsml).get()
   if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
       print(speak.reason)
   else:
       print("Spoken output saved in " + outputFile)
    ```

   **C#**

    ```csharp
   // Synthesize spoken output
   string responseSsml = $@"
       <speak version='1.0' xmlns='http://www.w3.org/2001/10/synthesis' xml:lang='en-US'>
           <voice name='en-GB-LibbyNeural'>
               {responseText}
               <break strength='weak'/>
               Time to end this lab!
           </voice>
       </speak>";
   SpeechSynthesisResult speak = await speechSynthesizer.SpeakSsmlAsync(responseSsml);
   if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
   {
       Console.WriteLine(speak.Reason);
   }
   else
   {
        Console.WriteLine("Spoken output saved in " + outputFile);
   }
    ```

1. Salve suas alterações e retorne ao terminal integrado para a pasta **speaking-clock** e digite o seguinte comando para executar o programa:

    **Python**

    ```
   python speaking-clock.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. Revise a saída do aplicativo, o que deve indicar que a saída falada foi salva em um arquivo.
1. Novamente, se você tiver um reprodutor multimídia capaz de reproduzir arquivos de áudio .wav, na barra de ferramentas do painel do Cloud Shell, use o botão **Carregar/Baixar arquivos** para baixar o arquivo de áudio da pasta do aplicativo e reproduzi-lo:

    **Python**

    /home/*user*`/mslearn-ai-language/Labfiles/07b-speech/Python/speaking-clock/output.wav`

    **C#**

    /home/*user*`/mslearn-ai-language/Labfiles/07b-speech/C-Sharp/speaking-clock/output.wav`

    O arquivo terá um som como este:
    
    <video controls src="./ai-foundry/media/Output2.mp4" title="O tempo é 5:30. Hora de finalizar este laboratório." width="150"></video>

## Limpar

Se tiver terminado de explorar a Fala de IA do Azure, deverá excluir os recursos que criou neste exercício para evitar incorrer em custos desnecessários do Azure.

1. Retorne à guia do navegador que contém o portal do Azure (ou reabra o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com` em uma nova guia do navegador) e visualize o conteúdo do grupo de recursos onde você implantou os recursos usados neste exercício.
1. Na barra de ferramentas, selecione **Excluir grupo de recursos**.
1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.

## E se você tiver um microfone e um alto-falante?

Neste exercício, você usará arquivos de áudio para a entrada e saída de fala. Vamos ver como o código pode ser modificado para usar hardware de áudio.

### Uso do Reconhecimento de fala com um microfone

Se você tiver um microfone, poderá usar o seguinte código para capturar a entrada de fala para reconhecimento de fala:

**Python**

```python
# Configure speech recognition
audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
print('Speak now...')

# Process speech input
speech = speech_recognizer.recognize_once_async().get()
if speech.reason == speech_sdk.ResultReason.RecognizedSpeech:
    command = speech.text
    print(command)
else:
    print(speech.reason)
    if speech.reason == speech_sdk.ResultReason.Canceled:
        cancellation = speech.cancellation_details
        print(cancellation.reason)
        print(cancellation.error_details)

```

**C#**

```csharp
// Configure speech recognition
using AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
Console.WriteLine("Speak now...");

SpeechRecognitionResult speech = await speechRecognizer.RecognizeOnceAsync();
if (speech.Reason == ResultReason.RecognizedSpeech)
{
    command = speech.Text;
    Console.WriteLine(command);
}
else
{
    Console.WriteLine(speech.Reason);
    if (speech.Reason == ResultReason.Canceled)
    {
        var cancellation = CancellationDetails.FromResult(speech);
        Console.WriteLine(cancellation.Reason);
        Console.WriteLine(cancellation.ErrorDetails);
    }
}
```

> **Observação**: o microfone padrão do sistema é a entrada de áudio padrão, então você também pode simplesmente omitir o AudioConfig completamente!

### Uso da síntese de voz com um alto-falante

Se você tiver um alto-falante, poderá usar o código a seguir para sintetizar a fala.

**Python**

```python
response_text = 'The time is {}:{:02d}'.format(now.hour,now.minute)

# Configure speech synthesis
speech_config.speech_synthesis_voice_name = "en-GB-RyanNeural"
audio_config = speech_sdk.audio.AudioOutputConfig(use_default_speaker=True)
speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config, audio_config)

# Synthesize spoken output
speak = speech_synthesizer.speak_text_async(response_text).get()
if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
    print(speak.reason)
```

**C#**

```csharp
var now = DateTime.Now;
string responseText = "The time is " + now.Hour.ToString() + ":" + now.Minute.ToString("D2");

// Configure speech synthesis
speechConfig.SpeechSynthesisVoiceName = "en-GB-RyanNeural";
using var audioConfig = AudioConfig.FromDefaultSpeakerOutput();
using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig, audioConfig);

// Synthesize spoken output
SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(responseText);
if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
{
    Console.WriteLine(speak.Reason);
}
```

> **Observação**: o alto-falante padrão do sistema é a saída de áudio padrão, então você também pode simplesmente omitir o AudioConfig completamente!

## Mais informações

Para obter mais informações sobre como usar as APIs de **Reconhecimento de Fala** e **Conversão de texto em fala**, consulte a [documentação de Reconhecimento de Fala](https://learn.microsoft.com/azure/ai-services/speech-service/index-speech-to-text) e [Documentação de conversão de texto em fala](https://learn.microsoft.com/azure/ai-services/speech-service/index-text-to-speech).
