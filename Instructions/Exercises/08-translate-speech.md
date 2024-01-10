---
lab:
  title: Traduzir Fala
  module: Module 8 - Translate speech with Azure AI Speech
---

# Traduzir Fala

O Fala de IA do Azure inclui uma API de tradução de fala que você pode usar para traduzir a linguagem falada. Por exemplo, suponha que você queira desenvolver um aplicativo tradutor que as pessoas possam usar ao viajar em lugares onde não falam o idioma local. Eles seriam capazes de dizer frases como "Onde é a estação?" ou "Preciso encontrar uma farmácia" em seu próprio idioma, e fazer com que ele os traduza para o idioma local.

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

1. No Visual Studio Code, no painel **Explorer**, navegue até a pasta **Labfiles/08-speech-translation** e expanda a pasta **CSharp** ou **Python** de acordo com a sua preferência de linguagem e a pasta **tradutor** que ela contém. Cada pasta contém os arquivos de linguagem específicos para o aplicativo no qual você integrará a funcionalidade de Fala de IA do Azure.
1. Clique com o botão direito do mouse na pasta **translator** que contém os arquivos de código e abra um terminal integrado. Em seguida, instale o pacote do SDK de Fala da IA do Azure executando o comando apropriado para sua preferência de linguagem:

    **C#**

    ```
    dotnet add package Microsoft.CognitiveServices.Speech --version 1.30.0
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-speech==1.30.0
    ```

1. No painel **Explorer**, na pasta **tradutor**, abra o arquivo de configuração da linguagem de sua preferência

    - **C#**: appsettings.json
    - **Python**: .env

1. Atualize os valores de configuração para incluir a **região** e uma **chave** do recurso de Fala de IA do Azure que você criou (disponível na página **Chaves e Ponto de Extremidade** para seu recurso de Fala de IA do Azure no portal do Azure).

    > **Observação**: certifique-se de adicionar a *região* para o seu recurso, <u>não</u> para o ponto de extremidade!

1. Salve o arquivo de configuração.

## Adicionar código para usar o SDK de Fala

1. Observe que a pasta **tradutor** contém um arquivo de código para o aplicativo cliente:

    - **C#**: Program.cs
    - **Python**: translator.py

    Abra o arquivo de código e, na parte superior, sob as referências de namespace existentes, localize o comentário **Importar namespaces**. Em seguida, sob este comentário, adicione o seguinte código específico de linguagem para importar os namespaces que você precisará para usar o SDK de Fala de IA do Azure:

    **C#**: Program.cs

    ```csharp
    // Import namespaces
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    using Microsoft.CognitiveServices.Speech.Translation;
    ```

    **Python**: translator.py

    ```python
    # Import namespaces
    import azure.cognitiveservices.speech as speech_sdk
    ```

1. Na função **Principal**, observe que o código para carregar a chave e a região do serviço de Fala de IA do Azure do arquivo de configuração já foi fornecido. Você deve usar essas variáveis para criar um **SpeechTranslationConfig** para seu recurso de Fala de IA do Azure, que você usará para traduzir a entrada falada. Adicione o seguinte código sob o comentário **Configurar tradução**:

    **C#**: Program.cs

    ```csharp
    // Configure translation
    translationConfig = SpeechTranslationConfig.FromSubscription(aiSvcKey, aiSvcRegion);
    translationConfig.SpeechRecognitionLanguage = "en-US";
    translationConfig.AddTargetLanguage("fr");
    translationConfig.AddTargetLanguage("es");
    translationConfig.AddTargetLanguage("hi");
    Console.WriteLine("Ready to translate from " + translationConfig.SpeechRecognitionLanguage);
    ```

    **Python**: translator.py

    ```python
    # Configure translation
    translation_config = speech_sdk.translation.SpeechTranslationConfig(ai_key, ai_region)
    translation_config.speech_recognition_language = 'en-US'
    translation_config.add_target_language('fr')
    translation_config.add_target_language('es')
    translation_config.add_target_language('hi')
    print('Ready to translate from',translation_config.speech_recognition_language)
    ```

1. Você usará o **SpeechTranslationConfig** para traduzir fala em texto, mas também usará um **SpeechConfig** para sintetizar traduções em fala. Adicione o seguinte código sob o comentário **Configurar fala**:

    **C#**: Program.cs

    ```csharp
    // Configure speech
    speechConfig = SpeechConfig.FromSubscription(aiSvcKey, aiSvcRegion);
    ```

    **Python**: translator.py

    ```python
    # Configure speech
    speech_config = speech_sdk.SpeechConfig(ai_key, ai_region)
    ```

1. Salve suas alterações e retorne ao terminal integrado para a pasta **tradutor** e digite o seguinte comando para executar o programa:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python translator.py
    ```

1. Se você estiver usando C#, poderá ignorar avisos sobre o uso do operador **await** em métodos assíncronos – corrigiremos isso mais tarde. O código deve exibir uma mensagem informando que está pronto para traduzir de en-US e solicitar um idioma de destino. Pressione ENTER para encerrar o programa.

## Implementar tradução de fala

Agora que você tem um **SpeechTranslationConfig** para o serviço de Fala de IA do Azure, você pode usar a API de tradução de Fala da IA do Azure para reconhecer e traduzir fala.

> **IMPORTANTE**: esta seção inclui instruções para dois procedimentos alternativos. Siga o primeiro procedimento se você tiver um microfone funcionando. Siga o segundo procedimento se quiser simular a entrada falada usando um arquivo de áudio.

### Se você tiver um microfone funcionando

1. Na função **Principal** do programa, observe que o código usa a função **Traduzir** para traduzir a entrada falada.
1. Na função **Traduzir**, no comentário **Traduzir fala**, adicione o código a seguir para criar um cliente **TranslationRecognizer** que possa ser usado para reconhecer e traduzir fala usando o microfone do sistema padrão para entrada.

    **C#**: Program.cs

    ```csharp
    // Translate speech
    using AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
    using TranslationRecognizer translator = new TranslationRecognizer(translationConfig, audioConfig);
    Console.WriteLine("Speak now...");
    TranslationRecognitionResult result = await translator.RecognizeOnceAsync();
    Console.WriteLine($"Translating '{result.Text}'");
    translation = result.Translations[targetLanguage];
    Console.OutputEncoding = Encoding.UTF8;
    Console.WriteLine(translation);
    ```

    **Python**: translator.py

    ```python
    # Translate speech
    audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
    translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config = audio_config)
    print("Speak now...")
    result = translator.recognize_once_async().get()
    print('Translating "{}"'.format(result.text))
    translation = result.translations[targetLanguage]
    print(translation)
    ```

    > **Observação** O código em seu aplicativo converte a entrada para todos os três idiomas em uma única chamada. Somente a tradução para o idioma específico é exibida, mas você pode recuperar qualquer uma das traduções especificando o código do idioma de destino na coleção de **traduções** do resultado.

1. Agora vá para a seção **Executar o programa** abaixo.

---

### Como alternativa, use a entrada de áudio de um arquivo

1. Na janela do terminal, digite o seguinte comando para instalar uma biblioteca que você pode usar para reproduzir o arquivo de áudio:

    **C#**: Program.cs

    ```csharp
    dotnet add package System.Windows.Extensions --version 4.6.0 
    ```

    **Python**: translator.py

    ```python
    pip install playsound==1.3.0
    ```

1. No arquivo de código do programa, nas importações de namespace existentes, adicione o seguinte código para importar a biblioteca que você acabou de instalar:

    **C#**: Program.cs

    ```csharp
    using System.Media;
    ```

    **Python**: translator.py

    ```python
    from playsound import playsound
    ```

1. Na função **Principal** do programa, observe que o código usa a função **Traduzir** para traduzir a entrada falada. Em seguida, na função **Traduzir**, sob o comentário **Traduzir fala**, adicione o código a seguir para criar um cliente **TranslationRecognizer** que pode ser usado para reconhecer e traduzir fala de um arquivo.

    **C#**: Program.cs

    ```csharp
    // Translate speech
    string audioFile = "station.wav";
    SoundPlayer wavPlayer = new SoundPlayer(audioFile);
    wavPlayer.Play();
    using AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
    using TranslationRecognizer translator = new TranslationRecognizer(translationConfig, audioConfig);
    Console.WriteLine("Getting speech from file...");
    TranslationRecognitionResult result = await translator.RecognizeOnceAsync();
    Console.WriteLine($"Translating '{result.Text}'");
    translation = result.Translations[targetLanguage];
    Console.OutputEncoding = Encoding.UTF8;
    Console.WriteLine(translation);
    ```

    **Python**: translator.py

    ```python
    # Translate speech
    audioFile = 'station.wav'
    playsound(audioFile)
    audio_config = speech_sdk.AudioConfig(filename=audioFile)
    translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config = audio_config)
    print("Getting speech from file...")
    result = translator.recognize_once_async().get()
    print('Translating "{}"'.format(result.text))
    translation = result.translations[targetLanguage]
    print(translation)
    ```

---

### Executar o programa

1. Salve suas alterações e retorne ao terminal integrado para a pasta **tradutor** e digite o seguinte comando para executar o programa:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python translator.py
    ```

1. Quando solicitado, digite um código de idioma válido (*fr*, *es* ou *hi*) e, em seguida, se estiver usando um microfone, fale claramente e diga "onde é a estação?" ou alguma outra frase que você possa usar ao viajar para o exterior. O programa deve transcrever sua entrada falada e traduzi-la para o idioma especificado (francês, espanhol ou hindi). Repita esse processo, tentando cada idioma suportado pelo aplicativo. Quando terminar, pressione ENTER para encerrar o programa.

    O TranslationRecognizer permite que você fale por cerca de 5 segundos. Se ele não detectar nenhuma entrada falada, ele produz um resultado "Sem correspondência". A tradução para Hindi nem sempre pode ser exibida corretamente na janela Console devido a problemas de codificação de caracteres.

> **OBSERVAÇÃO**: o código em seu aplicativo converte a entrada para todos os três idiomas em uma única chamada. Somente a tradução para o idioma específico é exibida, mas você pode recuperar qualquer uma das traduções especificando o código do idioma de destino na coleção de **traduções** do resultado.

## Sintetizar a tradução para a fala

Até agora, seu aplicativo traduz a entrada falada em texto; o que pode ser suficiente se você precisar pedir ajuda a alguém durante uma viagem. No entanto, seria melhor que a tradução fosse falada em voz alta em uma voz adequada.

1. Na função **Traduzir**, sob o comentário **Sintetizar tradução**, adicione o seguinte código para usar um cliente **SpeechSynthesizer** para sintetizar a tradução como fala através do alto-falante padrão:

    **C#**: Program.cs

    ```csharp
    // Synthesize translation
    var voices = new Dictionary<string, string>
                    {
                        ["fr"] = "fr-FR-HenriNeural",
                        ["es"] = "es-ES-ElviraNeural",
                        ["hi"] = "hi-IN-MadhurNeural"
                    };
    speechConfig.SpeechSynthesisVoiceName = voices[targetLanguage];
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(translation);
    if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
    {
        Console.WriteLine(speak.Reason);
    }
    ```

    **Python**: translator.py

    ```python
    # Synthesize translation
    voices = {
            "fr": "fr-FR-HenriNeural",
            "es": "es-ES-ElviraNeural",
            "hi": "hi-IN-MadhurNeural"
    }
    speech_config.speech_synthesis_voice_name = voices.get(targetLanguage)
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    speak = speech_synthesizer.speak_text_async(translation).get()
    if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

1. Salve suas alterações e retorne ao terminal integrado para a pasta **tradutor** e digite o seguinte comando para executar o programa:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python translator.py
    ```

1. Quando solicitado, insira um código de linguagem válido (*fr*, *es* ou *hi*) e, em seguida, fale claramente ao microfone e diga uma frase que você pode usar ao viajar para o exterior. O programa deve transcrever sua entrada falada e responder com uma tradução falada. Repita esse processo, tentando cada idioma suportado pelo aplicativo. Quando terminar, pressione **ENTER** para encerrar o programa.

> **OBSERVAÇÃO**
> *Neste exemplo, você usou um **SpeechTranslationConfig** para converter fala em texto e, em seguida, usou um **SpeechConfig** para sintetizar a tradução como fala. Você pode, de fato, usar o **SpeechTranslationConfig** para sintetizar a tradução diretamente, mas isso só funciona ao traduzir para um único idioma e resulta em um fluxo de áudio que normalmente é salvo como um arquivo em vez de enviado diretamente para um alto-falante.*

## Mais informações

Para obter mais informações sobre a API de tradução de Fala de IA do Azure, confira a [documentação de tradução de fala](https://learn.microsoft.com/azure/ai-services/speech-service/speech-translation).
