---
lab:
  title: Reconhecimento e sintetização de fala
  description: Implemente um relógio falante que converta fala em texto e texto em fala.
---

# Reconhecer e sintetizar fala

A **Fala de IA do Azure** é um serviço que fornece funcionalidade relacionada à fala, incluindo:

- Uma API de *reconhecimento de fala* que permite implementar o reconhecimento de fala (conversão de palavras faladas audíveis em texto).
- Uma API de *conversão de texto em fala* que permite implementar a sintetização da fala (convertendo texto em fala audível).

Neste exercício, você usará essas duas APIs para implementar um aplicativo de relógio falante.

Embora este exercício seja baseado em Python, você pode desenvolver aplicativos de fala usando vários SDKs específicos da linguagem, incluindo:

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

## Preparar e configurar o aplicativo de relógio falante

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

1. Após o repositório ser clonado, navegue até a pasta que contém os arquivos de código do aplicativo do relógio falante:  

    ```
   cd mslearn-ai-language/Labfiles/07-speech/Python/speaking-clock
    ```

1. No painel de linha de comando, execute o seguinte comando para exibir os arquivos de código na pasta **speaking-clock**:

    ```
   ls -a -l
    ```

    Os arquivos incluem um arquivo de configuração (**.env**) e um arquivo de código (**speaking-clock.py**). Os arquivos de áudio que seu aplicativo usará estão na subpasta **audio**.

1. Crie um ambiente virtual Python e instale o pacote SDK de Fala de IA do Azure e outros pacotes necessários executando o seguinte comando:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-cognitiveservices-speech==1.42.0
    ```

1. Insira o seguinte comando para editar o arquivo de configuração:

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
   code speaking-clock.py
    ```

1. Na parte superior do arquivo de código, sob as referências de namespace existentes, localize o comentário **Import namespaces**. Em seguida, sob este comentário, adicione o seguinte código específico de linguagem para importar os namespaces que você precisará para usar o SDK de Fala de IA do Azure:

    ```python
   # Import namespaces
   from azure.core.credentials import AzureKeyCredential
   import azure.cognitiveservices.speech as speech_sdk
    ```

1. Na função **main**, abaixo do comentário **Get config settings**, observe que o código carrega a chave e a região que você definiu no arquivo de configuração.

1. Encontre o comentário **Configure speech service** e adicione o código a seguir para usar a chave dos Serviços de IA e a região do projeto para configurar sua conexão com o ponto de extremidade de Fala dos Serviços de IA do Azure:

    ```python
   # Configure speech service
   speech_config = speech_sdk.SpeechConfig(speech_key, speech_region)
   print('Ready to use speech service in:', speech_config.region)
    ```

1. Salve as alterações (*CTRL+S*), mas deixe o editor de código aberto.

## Executar o aplicativo

Até agora, o aplicativo apenas se conecta ao seu projeto do Serviço de Fala de IA do Azure, mas é útil executá-lo e confirmar se ele funciona antes de adicionar a funcionalidade de fala.

1. Na linha de comando, insira o seguinte comando para executar o aplicativo de relógio falante:

    ```
   python speaking-clock.py
    ```

    O código deve exibir a região do recurso de serviço de fala que o aplicativo usará. Uma execução bem-sucedida indica que o aplicativo se conectou ao recurso de Fala de IA do Azure.

## Adicionar código para reconhecer a fala

Agora que você tem um **SpeechConfig** para o serviço de fala no recurso Serviços de IA do Azure do projeto, você pode usar a API de **reconhecimento de fala** para reconhecer a fala e transcrevê-la para texto.

Neste procedimento, a entrada de fala é capturada de um arquivo de áudio, que você pode reproduzir aqui:

<video controls src="https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Instructions/media/Time.mp4" title="Que horas são?" width="150"></video>

1. No arquivo de código, observe que o código usa a função **TranscribeCommand** para aceitar a entrada falada. Em seguida, na função **TranscribeCommand**, localize o comentário **Configure speech recognition** e adicione o código apropriado abaixo para criar um cliente **SpeechRecognizer** que possa ser usado para reconhecer e transcrever fala a partir de um arquivo de áudio:

    ```python
   # Configure speech recognition
   current_dir = os.getcwd()
   audioFile = current_dir + '/time.wav'
   audio_config = speech_sdk.AudioConfig(filename=audioFile)
   speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    ```

1. Na função **TranscribeCommand**, sob o comentário **Processar entrada de fala**, adicione o seguinte código para ouvir a entrada falada, tomando cuidado para não substituir o código no fim da função que retorna o comando:

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

1. Salve suas alterações (*CTRL+S*) e, na linha de comando abaixo do editor de código, execute novamente o programa:
1. Revise a saída, que deve “ouvir” com sucesso a fala no arquivo de áudio e retornar uma resposta adequada (observe que seu Cloud Shell do Azure pode estar sendo executado em um servidor em um fuso horário diferente do seu!)

    > **Dica**: se o SpeechRecognizer encontrar um erro, ele produzirá um resultado de "Cancelado". O código no aplicativo exibirá a mensagem de erro. A causa mais provável é uma região ou valor incorreto no arquivo de configuração.

## Sintetizar fala

Seu aplicativo de relógio de fala aceita entrada falada, mas na verdade não fala! Vamos corrigir isso adicionando código para sintetizar a fala.

Mais uma vez, devido às limitações de hardware do Cloud Shell, direcionaremos a saída de fala sintetizada para um arquivo.

1. No arquivo de código, observe que o código usa a função **TellTime** para informar ao usuário a hora atual.
1. Na função **TellTime**, sob o comentário **Configurar síntese de fala**, adicione o seguinte código para criar um cliente **SpeechSynthesizer** que pode ser usado para gerar saída falada:

    ```python
   # Configure speech synthesis
   output_file = "output.wav"
   speech_config.speech_synthesis_voice_name = "en-GB-RyanNeural"
   audio_config = speech_sdk.audio.AudioConfig(filename=output_file)
   speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config, audio_config,)
    ```

1. Na função **TellTime**, sob o comentário **Sintetizar saída falada**, adicione o seguinte código para gerar saída falada, tomando cuidado para não substituir o código no final da função que imprime a resposta:

    ```python
   # Synthesize spoken output
   speak = speech_synthesizer.speak_text_async(response_text).get()
   if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
   else:
        print("Spoken output saved in " + output_file)
    ```

1. Salve suas alterações (*CTRL+S*) e execute novamente o programa, que deve indicar que a saída de voz foi salva em um arquivo.

1. Se você tiver um reprodutor multimídia capaz de reproduzir arquivos de áudio .wav, baixe o arquivo que foi gerado inserindo o seguinte comando:

    ```
   download ./output.wav
    ```

    O comando download cria um link pop-up no canto inferior direito do seu navegador, que você pode selecionar para baixar e abrir o arquivo.

    O arquivo terá um som como este:

    <video controls src="https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Instructions/media/Output.mp4" title="O tempo é 2:15" width="150"></video>

## Usar a Linguagem de Marcação de Sintetização de Voz

O SSML (Linguagem de Marcação de Sintetização de Fala) permite personalizar a maneira como sua fala é sintetizada usando um formato baseado em XML.

1. Na função **TellTime**, substitua todo o código atual sob o comentário **Sintetizar saída falada** com o seguinte código (deixe o código sob o comentário **Imprimir a resposta**):

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
       print("Spoken output saved in " + output_file)
    ```

1. Salve suas alterações e execute novamente o programa, que deve indicar que a saída de voz foi salva em um arquivo.
1. Baixe e reproduza o arquivo gerado, que deve soar semelhante a este:
    
    <video controls src="https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Instructions/media/Output2.mp4" title="O tempo é 5:30. Hora de finalizar este laboratório." width="150"></video>

## Limpar

Se tiver terminado de explorar a Fala de IA do Azure, deverá excluir os recursos que criou neste exercício para evitar incorrer em custos desnecessários do Azure.

1. Feche o painel do Azure Cloud Shell
1. No portal do Azure, navegue até o recurso de Fala de IA do Azure criado neste laboratório.
1. Na página de recursos, selecione **Excluir** e siga as instruções para excluir o recurso.

## E se você tiver um microfone e um alto-falante?

Neste exercício, o ambiente do Azure Cloud Shell que usamos não oferece suporte a hardware de áudio, portanto, você usou arquivos de áudio para a entrada e a saída de fala. Vamos ver como o código pode ser modificado para usar o hardware de áudio, caso ele esteja disponível.

### Uso do Reconhecimento de fala com um microfone

Se você tiver um microfone, poderá usar o seguinte código para capturar a entrada de fala para reconhecimento de fala:

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

> **Observação**: o microfone padrão do sistema é a entrada de áudio padrão, então você também pode simplesmente omitir o AudioConfig completamente!

### Uso da síntese de voz com um alto-falante

Se você tiver um alto-falante, poderá usar o código a seguir para sintetizar a fala.

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

> **Observação**: o alto-falante padrão do sistema é a saída de áudio padrão, então você também pode simplesmente omitir o AudioConfig completamente!

## Mais informações

Para obter mais informações sobre como usar as APIs de **Reconhecimento de Fala** e **Conversão de texto em fala**, consulte a [documentação de Reconhecimento de Fala](https://learn.microsoft.com/azure/ai-services/speech-service/index-speech-to-text) e [Documentação de conversão de texto em fala](https://learn.microsoft.com/azure/ai-services/speech-service/index-text-to-speech).
