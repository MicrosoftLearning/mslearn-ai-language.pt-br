---
lab:
  title: Reconhecimento e sintetização de fala
  module: Module 4 - Create speech-enabled apps with Azure AI services
---

# Como usar o reconhecimento de fala e a sintetização de voz

A **Fala de IA do Azure** é um serviço que fornece funcionalidade relacionada à fala, incluindo:

- Uma API de *reconhecimento de fala* que permite implementar o reconhecimento de fala (conversão de palavras faladas audíveis em texto).
- Uma API de *conversão de texto em fala* que permite implementar a sintetização da fala (convertendo texto em fala audível).

Neste exercício, você usará essas duas APIs para implementar um aplicativo de relógio falante.

> **OBSERVAÇÃO** Este exercício requer que você esteja usando um computador com alto-falantes/fones de ouvido. Para uma melhor experiência, um microfone também é necessário. Alguns ambientes virtuais hospedados podem ser capazes de capturar áudio do microfone local, mas se isso não funcionar (ou se você não tiver um microfone), você poderá usar um arquivo de áudio fornecido para entrada de fala. Siga as instruções cuidadosamente, pois você precisará escolher diferentes opções, dependendo se estiver usando um microfone ou o arquivo de áudio.

## Provisionar um recurso de *Fala de IA do Azure*

Caso ainda não tenha um na sua assinatura, provisione um recurso de **Fala de IA do Azure**.

1. Abra o portal do Azure em `https://portal.azure.com` usando a conta Microsoft associada à sua assinatura do Azure.
1. No campo de pesquisa na parte superior, procure **serviços de IA do Azure** e pressione **Enter** e selecione **Criar** em **Serviço de fala** nos resultados.
1. Crie um recurso com as seguintes configurações:
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *escolher ou criar um grupo de recursos*
    - **Região**: *escolha uma região disponível*
    - **Nome**: *insira um nome exclusivo*
    - **Tipo de preço**: selecione **F0** (*gratuito*), ou **S** (*Standard*) se F não estiver disponível.
    - **Aviso de IA Responsável**: concordar
1. Selecione **Examinar + criar**.
1. Aguarde a conclusão da implantação e acesse o recurso implantado.
1. Vizualize a página **Chaves e Ponto de Extremidade**. Você precisará das informações nesta página mais adiante no exercício.

## Prepare-se para desenvolver um aplicativo no Visual Studio Code

Você desenvolverá seu aplicativo de fala usando o Visual Studio Code. Os arquivos de código para seu aplicativo foram fornecidos em um repositório do GitHub.

> **Dica**: se você já clonou o repositório **mslearn-ai-language**, abra-o no Visual Studio Code. Caso contrário, siga estas etapas para cloná-lo em seu ambiente de desenvolvimento.

1. Inicie o Visual Studio Code.
1. Abra a paleta (SHIFT+CTRL+P) e execute um comando **Git: Clone** para clonar o repositório `https://github.com/MicrosoftLearning/mslearn-ai-language` para uma pasta local (não importa a pasta).
1. Depois que o repositório for clonado, abra a pasta no Visual Studio Code.
1. Aguarde enquanto arquivos adicionais são instalados para que haja suporte aos projetos com o código C# no repositório.

    > **Observação**: se você for solicitado a adicionar os ativos necessários para compilar e depurar, selecione **Agora não**.

## Configurar seu aplicativo

Os aplicativos para C# e Python foram fornecidos. Ambos os aplicativos apresentam a mesma funcionalidade. Primeiro, você concluirá algumas partes importantes do aplicativo para habilitar o uso do recurso de Fala de IA do Azure.

1. No Visual Studio Code, no painel **Explorer**, navegue até a pasta **Labfiles/07-speech** e expanda a pasta **CSharp** ou **Python** dependendo de sua preferência de linguagem e a pasta **speaking-clock** que ela contém. Cada pasta contém os arquivos de linguagem específicos para o aplicativo no qual você integrará a funcionalidade de Fala de IA do Azure.
1. Clique com o botão direito do mouse na pasta **speaking-clock** que contém seus arquivos de código e abra um terminal integrado. Em seguida, instale o pacote do SDK de Fala da IA do Azure executando o comando apropriado para sua preferência de linguagem:

    **C#**

    ```
    dotnet add package Microsoft.CognitiveServices.Speech --version 1.30.0
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-speech==1.30.0
    ```

1. No painel **Explorer**, na pasta **speaking-clock**, abra o arquivo de configuração do idioma de sua preferência

    - **C#**: appsettings.json
    - **Python**: .env

1. Atualize os valores de configuração para incluir a **região** e uma **chave** do recurso de Fala de IA do Azure que você criou (disponível na página **Chaves e Ponto de Extremidade** para seu recurso de Fala de IA do Azure no portal do Azure).

    > **Observação**: certifique-se de adicionar a *região* para o seu recurso, <u>não</u> para o ponto de extremidade!

1. Salve o arquivo de configuração.

## Adicionar código para usar o SDK de Fala de IA do Azure

1. Observe que a pasta **speaking-clock** contém um arquivo de código para o aplicativo cliente:

    - **C#**: Program.cs
    - **Python**: speaking-clock.py

    Abra o arquivo de código e, na parte superior, sob as referências de namespace existentes, localize o comentário **Importar namespaces**. Em seguida, sob este comentário, adicione o seguinte código específico de linguagem para importar os namespaces que você precisará para usar o SDK de Fala de IA do Azure:

    **C#**: Program.cs

    ```csharp
    // Import namespaces
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    ```

    **Python**: speaking-clock.py

    ```python
    # Import namespaces
    import azure.cognitiveservices.speech as speech_sdk
    ```

1. Na função **Principal**, observe que o código para carregar a chave de serviço e a região do arquivo de configuração já foi fornecido. Você deve usar essas variáveis para criar um **SpeechConfig** para seu recurso de Fala de IA do Azure. Adicione o seguinte código sob o comentário **Configurar serviço de fala**:

    **C#**: Program.cs

    ```csharp
    // Configure speech service
    speechConfig = SpeechConfig.FromSubscription(aiSvcKey, aiSvcRegion);
    Console.WriteLine("Ready to use speech service in " + speechConfig.Region);
    
    // Configure voice
    speechConfig.SpeechSynthesisVoiceName = "en-US-AriaNeural";
    ```

    **Python**: speaking-clock.py

    ```python
    # Configure speech service
    speech_config = speech_sdk.SpeechConfig(ai_key, ai_region)
    print('Ready to use speech service in:', speech_config.region)
    ```

1. Salve suas alterações e retorne ao terminal integrado para a pasta **speaking-clock** e digite o seguinte comando para executar o programa:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. Se você estiver usando C#, poderá ignorar avisos sobre o uso do operador **await** em métodos assíncronos – corrigiremos isso mais tarde. O código deve exibir a região do recurso de serviço de fala que o aplicativo usará.

## Adicionar código para reconhecer a fala

Agora que você tem um **SpeechConfig** para o serviço de fala em seu recurso de Fala de IA do Azure, você pode usar a API de **reconhecimento de fala** para reconhecer a fala e transcrevê-la para texto.

> **IMPORTANTE**: esta seção inclui instruções para dois procedimentos alternativos. Siga o primeiro procedimento se você tiver um microfone funcionando. Siga o segundo procedimento se quiser simular a entrada falada usando um arquivo de áudio.

### Se você tiver um microfone funcionando

1. Na função **Principal** do programa, observe que o código usa a função **TranscribeCommand** para aceitar entrada falada.
1. Na função **TranscribeCommand**, no comentário **Configurar reconhecimento de fala**, adicione o código apropriado abaixo para criar um cliente **SpeechRecognizer** que possa ser usado para reconhecer e transcrever fala usando o microfone do sistema padrão:

    **C#**

    ```csharp
    // Configure speech recognition
    using AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
    using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
    Console.WriteLine("Speak now...");
    ```

    **Python**

    ```python
    # Configure speech recognition
    audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
    speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    print('Speak now...')
    ```

1. Agora avance para a seção **Adicionar código para processar o comando transcrito** abaixo.

---

### Como alternativa, use a entrada de áudio de um arquivo

1. Na janela do terminal, digite o seguinte comando para instalar uma biblioteca que você pode usar para reproduzir o arquivo de áudio:

    **C#**

    ```
    dotnet add package System.Windows.Extensions --version 4.6.0 
    ```

    **Python**

    ```
    pip install playsound==1.3.0
    ```

1. No arquivo de código do programa, nas importações de namespace existentes, adicione o seguinte código para importar a biblioteca que você acabou de instalar:

    **C#**: Program.cs

    ```csharp
    using System.Media;
    ```

    **Python**: speaking-clock.py

    ```python
    from playsound import playsound
    ```

1. Na função **Principal**, observe que o código usa a função **TranscribeCommand** para aceitar entrada falada. Em seguida, na função **TranscribeCommand**, sob o comentário **Configurar reconhecimento de fala**, adicione o código apropriado abaixo para criar um cliente **SpeechRecognizer** que possa ser usado para reconhecer e transcrever a fala de um arquivo de áudio:

    **C#**: Program.cs

    ```csharp
    // Configure speech recognition
    string audioFile = "time.wav";
    SoundPlayer wavPlayer = new SoundPlayer(audioFile);
    wavPlayer.Play();
    using AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
    using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
    ```

    **Python**: speaking-clock.py

    ```python
    # Configure speech recognition
    current_dir = os.getcwd()
    audioFile = current_dir + '\\time.wav'
    playsound(audioFile)
    audio_config = speech_sdk.AudioConfig(filename=audioFile)
    speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    ```

---

### Adicionar código para processar o comando transcrito

1. Na função **TranscribeCommand**, sob o comentário **Processar entrada de fala**, adicione o seguinte código para ouvir a entrada falada, tomando cuidado para não substituir o código no fim da função que retorna o comando:

    **C#**: Program.cs

    ```csharp
    // Process speech input
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

    **Python**: speaking-clock.py

    ```python
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

1. Salve suas alterações e retorne ao terminal integrado para a pasta **speaking-clock** e digite o seguinte comando para executar o programa:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. Se estiver usando um microfone, fale claramente e diga "que horas são?". O programa deve transcrever sua entrada falada e exibir a hora (com base na hora local do computador onde o código está sendo executado, que pode não ser a hora correta onde você está).

    O SpeechRecognizer da a você cerca de 5 segundos para falar. Se ele não detectar nenhuma entrada falada, ele produz um resultado "Sem correspondência".

    Se o SpeechRecognizer encontrar um erro, ele produzirá um resultado de "Cancelado". O código no aplicativo exibirá a mensagem de erro. A causa mais provável é uma chave ou região incorreta no arquivo de configuração.

## Sintetizar fala

Seu aplicativo de relógio de fala aceita entrada falada, mas na verdade não fala! Vamos corrigir isso adicionando código para sintetizar a fala.

1. Na função **Principal** do programa, observe que o código usa a função **TellTime** para informar ao usuário a hora atual.
1. Na função **TellTime**, sob o comentário **Configurar síntese de fala**, adicione o seguinte código para criar um cliente **SpeechSynthesizer** que pode ser usado para gerar saída falada:

    **C#**: Program.cs

    ```csharp
    // Configure speech synthesis
    speechConfig.SpeechSynthesisVoiceName = "en-GB-RyanNeural";
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    ```

    **Python**: speaking-clock.py

    ```python
    # Configure speech synthesis
    speech_config.speech_synthesis_voice_name = "en-GB-RyanNeural"
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    ```

    > **OBSERVAÇÃO** A configuração de áudio padrão usa o dispositivo de áudio padrão do sistema para saída, portanto, você não precisa fornecer um **AudioConfig**. Se você precisar redirecionar a saída de áudio para um arquivo, poderá usar um **AudioConfig** com um caminho de arquivo para fazer isso.

1. Na função **TellTime**, sob o comentário **Sintetizar saída falada**, adicione o seguinte código para gerar saída falada, tomando cuidado para não substituir o código no final da função que imprime a resposta:

    **C#**: Program.cs

    ```csharp
    // Synthesize spoken output
    SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(responseText);
    if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
    {
        Console.WriteLine(speak.Reason);
    }
    ```

    **Python**: speaking-clock.py

    ```python
    # Synthesize spoken output
    speak = speech_synthesizer.speak_text_async(response_text).get()
    if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

1. Salve suas alterações e retorne ao terminal integrado para a pasta **speaking-clock** e digite o seguinte comando para executar o programa:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. Quando solicitado, fale claramente ao microfone e diga "que horas são?". O programa deve falar, dizendo a hora.

## Usar uma voz diferente

Seu aplicativo de relógio de fala usa uma voz padrão, que você pode alterar. O serviço de fala suporta uma variedade de vozes *padrão*, bem como vozes *neurais* mais semelhantes às humanas. Você também pode criar vozes *personalizadas*.

> **Observação**: para obter uma lista de vozes neurais e padrão, consulte [Galeria de Vozes](https://speech.microsoft.com/portal/voicegallery) no Speech Studio.

1. Na função **TellTime**, sob o comentário **Configurar síntese de fala**, modifique o código da seguinte forma para especificar uma voz alternativa antes de criar o cliente **SpeechSynthesizer**:

   **C#**: Program.cs

    ```csharp
    // Configure speech synthesis
    speechConfig.SpeechSynthesisVoiceName = "en-GB-LibbyNeural"; // change this
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    ```

    **Python**: speaking-clock.py

    ```python
    # Configure speech synthesis
    speech_config.speech_synthesis_voice_name = 'en-GB-LibbyNeural' # change this
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    ```

1. Salve suas alterações e retorne ao terminal integrado para a pasta **speaking-clock** e digite o seguinte comando para executar o programa:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. Quando solicitado, fale claramente ao microfone e diga "que horas são?". O programa deve falar na voz especificada, informando a hora.

## Usar a Linguagem de Marcação de Sintetização de Voz

O SSML (Linguagem de Marcação de Sintetização de Fala) permite personalizar a maneira como sua fala é sintetizada usando um formato baseado em XML.

1. Na função **TellTime**, substitua todo o código atual sob o comentário **Sintetizar saída falada** com o seguinte código (deixe o código sob o comentário **Imprimir a resposta**):

   **C#**: Program.cs

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
    ```

    **Python**: speaking-clock.py

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
    ```

1. Salve suas alterações e retorne ao terminal integrado para a pasta **speaking-clock** e digite o seguinte comando para executar o programa:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. Quando solicitado, fale claramente ao microfone e diga "que horas são?". O programa deve falar na voz especificada no SSML (substituindo a voz especificada no SpeechConfig), informando a hora e, depois de uma pausa, dizer que é hora de terminar este laboratório – e é!

## Mais informações

Para obter mais informações sobre como usar as APIs de **Reconhecimento de Fala** e **Conversão de texto em fala**, consulte a [documentação de Reconhecimento de Fala](https://learn.microsoft.com/azure/ai-services/speech-service/index-speech-to-text) e [Documentação de conversão de texto em fala](https://learn.microsoft.com/azure/ai-services/speech-service/index-text-to-speech).
