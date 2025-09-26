---
lab:
  title: Traduzir Fala
  description: Traduza fala entre idiomas de fala para fala e implemente no seu próprio aplicativo.
---

# Traduzir Fala

O Fala de IA do Azure inclui uma API de tradução de fala que você pode usar para traduzir a linguagem falada. Por exemplo, suponha que você queira desenvolver um aplicativo tradutor que as pessoas possam usar ao viajar em lugares onde não falam o idioma local. Eles seriam capazes de dizer frases como "Onde é a estação?" ou "Preciso encontrar uma farmácia" em seu próprio idioma, e fazer com que ele os traduza para o idioma local. Neste exercício, você usará o SDK de Fala de IA do Azure para Python para criar um aplicativo simples com base neste exemplo.

Embora este exercício seja baseado em Python, você pode desenvolver aplicativos de tradução de fala usando vários SDKs específicos da linguagem, incluindo:

- [SDK de Fala de IA do Azure para Python](https://pypi.org/project/azure-cognitiveservices-speech/)
- [SDK de Fala de IA do Azure para .NET](https://www.nuget.org/packages/Microsoft.CognitiveServices.Speech)
- [SDK de Fala de IA do Azure para JavaScript](https://www.npmjs.com/package/microsoft-cognitiveservices-speech-sdk)

Este exercício levará aproximadamente **30** minutos.

> **OBSERVAÇÃO** Este exercício foi elaborado para ser feito no Azure Cloud Shell, onde não há suporte para acesso direto ao hardware de som do computador. Portanto, o laboratório usará arquivos de áudio para fluxos de entrada e saída de fala. O código para obter os mesmos resultados usando um microfone e alto-falante é fornecido para sua referência.

## Criar um recurso de Fala de IA do Azure

Vamos começar criando um recurso da Fala de IA do Azure.

1. Abra o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com` usando a conta Microsoft associada à sua assinatura do Azure.
1. No campo de pesquisa superior, pesquise **Serviço de Fala**. Selecione-o na lista e, em seguida, selecione **Criar**.
1. Provisione o recurso usando as seguintes configurações:
    - **Assinatura**: *sua assinatura do Azure*.
    - **Grupo de recursos**: *crie ou escolha um grupo de recursos*.
    - **Região**:*escolha uma região disponível*
    - **Nome**: *insira um nome exclusivo*.
    - **Tipo de preço**: selecione **F0** (*gratuito*), ou **S** (*Standard*) se F não estiver disponível.
1. Selecione **Revisar + criar** e, em seguida, selecione **Criar** para provisionar o recurso.
1. Aguarde a conclusão da implantação e acesse o recurso implantado.
1. Veja a página **Chaves e Ponto de extremidade** na seção **Gerenciamento de Recursos**. Você precisará das informações nesta página mais adiante no exercício.

## Preparar-se para desenvolver um aplicativo no Cloud Shell

1. Mantendo a página **Chaves e Ponto de extremidade** aberta, use o botão **[\>_]** à direita da barra de pesquisa no topo da página para criar um novo Cloud Shell no portal do Azure, selecionando um ambiente ***PowerShell***. O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure.

    > **Observação**: se você já criou um Cloud Shell que usa um ambiente *Bash*, alterne-o para o ***PowerShell***.

1. Na barra de ferramentas do Cloud Shell, no menu **Configurações**, selecione **Ir para a versão clássica** (isso é necessário para usar o editor de código).

    **<font color="red">Verifique se você mudou para a versão clássica do Cloud Shell antes de continuar.</font>**

1. No painel do PowerShell, insira os seguintes comandos para clonar o repositório GitHub para este exercício:

    ```
   rm -r mslearn-ai-language -f
   git clone https://github.com/microsoftlearning/mslearn-ai-language
    ```

    > **Dica**: Ao colar comandos no Cloud Shell, a saída poderá ocupar uma grande quantidade do buffer da tela. Você pode limpar a tela digitando o comando `cls` para facilitar o foco em cada tarefa.

1. Depois que o repositório tiver sido clonado, navegue até a pasta que contém os arquivos de código:

    ```
   cd mslearn-ai-language/Labfiles/08-speech-translation/Python/translator
    ```

1. No painel de linha de comando, execute o seguinte comando para exibir os arquivos de código na pasta **translator**:

    ```
   ls -a -l
    ```

    Os arquivos incluem um arquivo de configuração (**.env**) e um arquivo de código (**translator.py**).

1. Crie um ambiente virtual Python e instale o pacote SDK de Fala de IA do Azure e outros pacotes necessários executando o seguinte comando:

    ```
    python -m venv labenv
    ./labenv/bin/Activate.ps1
    pip install -r requirements.txt azure-cognitiveservices-speech==1.42.0
    ```

1. Digite o seguinte comando para editar o arquivo de configuração que foi fornecido:

    ```
   code .env
    ```

    O arquivo é aberto em um editor de código.

1. Atualize os valores de configuração para incluir a **região** e uma **chave** do recurso de Fala de IA do Azure que você criou (disponível na página **Chaves e Ponto de extremidade** do seu recurso de Fala de IA do Azure no portal do Azure).
1. Depois de substituir os espaços reservados, use o comando **CTRL+S** para salvar suas alterações e, em seguida, use o comando **CTRL+Q** para fechar o editor de código, mantendo a linha de comando do Cloud Shell aberta.

## Adicionar código para usar o SDK de Fala de IA do Azure

> **Dica**: ao adicionar código, certifique-se de manter o recuo correto.

1. Digite o seguinte comando para editar o arquivo de código que foi fornecido:

    ```
   code translator.py
    ```

1. Na parte superior do arquivo de código, sob as referências de namespace existentes, localize o comentário **Import namespaces**. Em seguida, sob este comentário, adicione o seguinte código específico de linguagem para importar os namespaces que você precisará para usar o SDK de Fala de IA do Azure:

    ```python
   # Import namespaces
   from azure.core.credentials import AzureKeyCredential
   import azure.cognitiveservices.speech as speech_sdk
    ```

1. Na função **main**, abaixo do comentário **Get config settings**, observe que o código carrega a chave e a região que você definiu no arquivo de configuração.

1. Localize o seguinte código abaixo do comentário **Configure translation** e adicione o código a seguir para configurar sua conexão com o ponto de extremidade de Fala dos Serviços de IA do Azure:

    ```python
   # Configure translation
   translation_config = speech_sdk.translation.SpeechTranslationConfig(speech_key, speech_region)
   translation_config.speech_recognition_language = 'en-US'
   translation_config.add_target_language('fr')
   translation_config.add_target_language('es')
   translation_config.add_target_language('hi')
   print('Ready to translate from',translation_config.speech_recognition_language)
    ```

1. Você usará o **SpeechTranslationConfig** para traduzir fala em texto, mas também usará um **SpeechConfig** para sintetizar traduções em fala. Adicione o seguinte código sob o comentário **Configurar fala**:

    ```python
   # Configure speech
   speech_config = speech_sdk.SpeechConfig(speech_key, speech_region)
   print('Ready to use speech service in:', speech_config.region)
    ```

1. Salve as alterações (*CTRL+S*), mas deixe o editor de código aberto.

## Executar o aplicativo

Até agora, o aplicativo apenas se conecta ao seu projeto do recurso de Fala de IA do Azure, mas é útil executá-lo e confirmar se ele funciona antes de adicionar a funcionalidade de fala.

1. Na linha de comando, insira o comando a seguir para executar o aplicativo de tradução:

    ```
   python translator.py
    ```

    O código deve exibir a região do recurso de serviço de fala que o aplicativo usará, uma mensagem que ele está pronto para traduzir do en-US e solicitar um idioma de destino. Uma execução bem-sucedida indica que o aplicativo se conectou ao serviço de Fala de IA do Azure. Pressione ENTER para encerrar o programa.

## Implementar tradução de fala

Agora que você tem um **SpeechTranslationConfig** para o serviço de Fala de IA do Azure, você pode usar a API de tradução de Fala da IA do Azure para reconhecer e traduzir fala.

1. No arquivo de código, observe que o código usa a função **Translate** para traduzir entrada falada. Em seguida, na função **Traduzir**, sob o comentário **Traduzir fala**, adicione o código a seguir para criar um cliente **TranslationRecognizer** que pode ser usado para reconhecer e traduzir fala de um arquivo.

    ```python
   # Translate speech
   current_dir = os.getcwd()
   audioFile = current_dir + '/station.wav'
   audio_config_in = speech_sdk.AudioConfig(filename=audioFile)
   translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config = audio_config_in)
   print("Getting speech from file...")
   result = translator.recognize_once_async().get()
   print('Translating "{}"'.format(result.text))
   translation = result.translations[targetLanguage]
   print(translation)
    ```

1. Salve suas alterações (*CTRL+S*) e execute novamente o programa:

    ```
   python translator.py
    ```

1. Quando solicitado, insira um código de idioma válido (*fr*, *es*ou *hi*). O programa deve transcrever seu arquivo de entrada e traduzi-lo para o idioma especificado (francês, espanhol ou híndi). Repita esse processo, tentando cada idioma suportado pelo aplicativo.

    > **OBSERVAÇÃO**: A tradução para o híndi pode nem sempre ser exibida corretamente na janela do Console devido a problemas de codificação de caracteres.

1. Quando terminar, pressione ENTER para encerrar o programa.

> **OBSERVAÇÃO**: o código em seu aplicativo converte a entrada para todos os três idiomas em uma única chamada. Somente a tradução para o idioma específico é exibida, mas você pode recuperar qualquer uma das traduções especificando o código do idioma de destino na coleção de **traduções** do resultado.

## Sintetizar a tradução para a fala

Até agora, seu aplicativo traduz a entrada falada em texto; o que pode ser suficiente se você precisar pedir ajuda a alguém durante uma viagem. No entanto, seria melhor que a tradução fosse falada em voz alta em uma voz adequada.

> **Observação**: Devido às limitações de hardware do Cloud Shell, direcionaremos a saída de fala sintetizada para um arquivo.

1. Na função **Translate**, localize o comentário **Synthesize translation** e adicione o seguinte código para usar um cliente **SpeechSynthesizer** para sintetizar a tradução como fala e salvá-la em um arquivo .wav:

    ```python
   # Synthesize translation
   output_file = "output.wav"
   voices = {
            "fr": "fr-FR-HenriNeural",
            "es": "es-ES-ElviraNeural",
            "hi": "hi-IN-MadhurNeural"
   }
   speech_config.speech_synthesis_voice_name = voices.get(targetLanguage)
   audio_config_out = speech_sdk.audio.AudioConfig(filename=output_file)
   speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config, audio_config_out)
   speak = speech_synthesizer.speak_text_async(translation).get()
   if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
   else:
        print("Spoken output saved in " + output_file)
    ```

1. Salve suas alterações (*CTRL+S*) e execute novamente o programa:

    ```
   python translator.py
    ```

1. Revise a saída do aplicativo, que deve indicar que a tradução em áudio foi salva em um arquivo. Quando terminar, pressione **ENTER** para encerrar o programa.
1. Se você tiver um reprodutor multimídia capaz de reproduzir arquivos de áudio .wav, baixe o arquivo que foi gerado inserindo o seguinte comando:

    ```
   download ./output.wav
    ```

    O comando download cria um link pop-up no canto inferior direito do seu navegador, que você pode selecionar para baixar e abrir o arquivo.

> **OBSERVAÇÃO**
> * neste exemplo, você usou um **SpeechTranslationConfig** para traduzir fala em texto e, em seguida, usou um **SpeechConfig** para sintetizar a tradução como fala. Na verdade, você pode usar o **SpeechTranslationConfig** para sintetizar a tradução diretamente, mas isso funciona apenas ao traduzir para um único idioma e gera um fluxo de áudio que normalmente é salvo em um arquivo.*

## Limpar os recursos

Se você terminou de explorar o serviço de Fala de IA do Azure, pode excluir os recursos que criou neste exercício. Este é o procedimento:

1. Feche o painel do Azure Cloud Shell
1. No portal do Azure, navegue até o recurso de Fala de IA do Azure criado neste laboratório.
1. Na página de recursos, selecione **Excluir** e siga as instruções para excluir o recurso.

## E se você tiver um microfone e um alto-falante?

Neste exercício, o ambiente do Azure Cloud Shell que usamos não oferece suporte a hardware de áudio, portanto, você usou arquivos de áudio para a entrada e a saída de fala. Vamos ver como o código pode ser modificado para usar o hardware de áudio, caso ele esteja disponível.

### Usar a tradução de fala com um microfone

1. Se você tiver um microfone, pode usar o seguinte código para capturar entrada de fala para tradução:

    ```python
   # Translate speech
   audio_config_in = speech_sdk.AudioConfig(use_default_microphone=True)
   translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config = audio_config_in)
   print("Speak now...")
   result = translator.recognize_once_async().get()
   print('Translating "{}"'.format(result.text))
   translation = result.translations[targetLanguage]
   print(translation)
    ```

> **Observação**: o microfone padrão do sistema é a entrada de áudio padrão, então você também pode simplesmente omitir o AudioConfig completamente!

### Uso da síntese de voz com um alto-falante

1. Se você tiver um alto-falante, poderá usar o código a seguir para sintetizar a fala.
    
    ```python
   # Synthesize translation
   voices = {
            "fr": "fr-FR-HenriNeural",
            "es": "es-ES-ElviraNeural",
            "hi": "hi-IN-MadhurNeural"
   }
   speech_config.speech_synthesis_voice_name = voices.get(targetLanguage)
   audio_config_out = speech_sdk.audio.AudioOutputConfig(use_default_speaker=True)
   speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config, audio_config_out)
   speak = speech_synthesizer.speak_text_async(translation).get()
   if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

> **Observação**: o alto-falante padrão do sistema é a saída de áudio padrão, então você também pode simplesmente omitir o AudioConfig completamente!

## Mais informações

Para obter mais informações sobre a API de tradução de Fala de IA do Azure, confira a [documentação de tradução de fala](https://learn.microsoft.com/azure/ai-services/speech-service/speech-translation).
