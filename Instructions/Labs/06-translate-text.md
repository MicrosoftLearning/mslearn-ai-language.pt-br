---
lab:
  title: Traduzir o texto
  description: Traduza textos fornecidos entre quaisquer idiomas suportados com o Tradutor de IA do Azure.
---

# Traduzir o texto

O **Tradutor de IA do Azure** é um serviço que permite a você traduzir textos entre idiomas. Neste exercício, você o usará para criar um aplicativo simples que traduz a entrada em qualquer idioma suportado para o idioma destino de sua escolha.

Embora este exercício seja baseado em Python, você pode desenvolver aplicativos de tradução de texto usando vários SDKs específicos da linguagem, incluindo:

- [Biblioteca de clientes de Tradução de IA do Azure para Python](https://pypi.org/project/azure-ai-translation-text/)
- [Biblioteca de clientes de Tradução de IA do Azure para .NET](https://www.nuget.org/packages/Azure.AI.Translation.Text)
- [Biblioteca de clientes de Tradução de IA do Azure para JavaScript](https://www.npmjs.com/package/@azure-rest/ai-translation-text)

Este exercício levará aproximadamente **30** minutos.

## Provisionar um recurso do *Tradutor de IA do Azure*

Caso ainda não tenha um na sua assinatura, provisione um recurso do **Tradutor de IA do Azure**.

1. Abra o portal do Azure em `https://portal.azure.com` e entre usando a conta Microsoft associada à sua assinatura do Azure.
1. No campo de pesquisa na parte superior, pesquise por **Tradutor** e, em seguida, selecione **Tradutor** nos resultados.
1. Crie um recurso com as seguintes configurações:
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *escolher ou criar um grupo de recursos*
    - **Região**: *escolha uma região disponível*
    - **Nome**: *insira um nome exclusivo*
    - **Tipo de preço**: selecione **F0** (*gratuito*), ou **S** (*Standard*) se F não estiver disponível.
1. Selecione **Revisar + criar** e, em seguida, selecione **Criar** para provisionar o recurso.
1. Aguarde a conclusão da implantação e acesse o recurso implantado.
1. Exiba a página **Chaves e Ponto de Extremidade**. Você precisará das informações nesta página mais adiante no exercício.

## Preparar-se para desenvolver um aplicativo no Cloud Shell

Para testar os recursos de tradução de texto do Tradutor de IA do Azure, você desenvolverá um aplicativo de console simples no Azure Cloud Shell.

1. No Portal do Azure, use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um Cloud Shell no portal do Azure selecionando um ambiente do ***PowerShell***. O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure.

    > **Observação**: se você já criou um Cloud Shell que usa um ambiente *Bash*, alterne-o para o ***PowerShell***.

1. Na barra de ferramentas do Cloud Shell, no menu **Configurações**, selecione **Ir para a versão clássica** (isso é necessário para usar o editor de código).

    **<font color="red">Verifique se você mudou para a versão clássica do Cloud Shell antes de continuar.</font>**

1. No painel do PowerShell, insira os seguintes comandos para clonar o repositório GitHub para este exercício:

    ```
   rm -r mslearn-ai-language -f
   git clone https://github.com/microsoftlearning/mslearn-ai-language
    ```

    > **Dica**: Ao colar comandos no Cloud Shell, a saída poderá ocupar uma grande quantidade do buffer da tela. Você pode limpar a tela digitando o comando `cls` para facilitar o foco em cada tarefa.

1. Após o repositório ser clonado, navegue até a pasta que contém os arquivos de código do aplicativo:  

    ```
   cd mslearn-ai-language/Labfiles/06-translator-sdk/Python/translate-text
    ```

## Configurar seu aplicativo

1. No painel de linha de comando, execute o seguinte comando para exibir os arquivos de código na pasta **translate-text**:

    ```
   ls -a -l
    ```

    Os arquivos incluem um arquivo de configuração (**.env**) e um arquivo de código (**translate.py**).

1. Crie um ambiente virtual Python e instale o pacote SDK de Tradução de IA do Azure e outros pacotes necessários executando o seguinte comando:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-translation-text==1.0.1
    ```

1. Insira o seguinte comando para editar o arquivo de configuração de aplicativo:

    ```
   code .env
    ```

    O arquivo é aberto em um editor de código.

1. Atualize os valores de configuração para incluir a **região** e uma **chave** do recurso do Tradutor de IA do Azure que você criou (disponível na página **Chaves e Ponto de Extremidade** para seu recurso do Tradutor de IA do Azure no portal do Azure).

    > **Observação**: certifique-se de adicionar a *região* para o seu recurso, <u>não</u> para o ponto de extremidade!

1. Depois de substituir os espaços reservados, use o comando **CTRL+S** ou **botão direito do mouse > Salvar** para salvar as suas alterações e, em seguida, use o comando **CTRL+Q** ou **botão direito do mouse > Sair** para fechar o editor de código, mantendo a linha de comando do Cloud Shell aberta.

## Adicionar código para traduzir texto

1. Insira o seguinte comando para editar o arquivo de código do aplicativo:

    ```
   code translate.py
    ```

1. Revise o código existente. Você adicionará código para trabalhar com o SDK de Tradução de IA do Azure.

    > **Dica**: ao adicionar código ao arquivo, certifique-se de manter o recuo correto.

1. No topo do arquivo de código, abaixo das referências de namespace existentes, localize o comentário **Import namespaces** e adicione o seguinte código para importar os namespaces necessários para usar o SDK de Tradução:

    ```python
   # import namespaces
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.translation.text import *
   from azure.ai.translation.text.models import InputTextItem
    ```

1. Na função **main**, observe que o código existente lê as configurações de configuração.
1. Localize o comentário **Criar cliente usando ponto de extremidade e chave** e adicione o seguinte código:

    ```python
   # Create client using endpoint and key
   credential = AzureKeyCredential(translatorKey)
   client = TextTranslationClient(credential=credential, region=translatorRegion)
    ```

1. Localize o comentário **Choose target language** e adicione o seguinte código, que usa o serviço de Tradução de Texto para retornar a lista de idiomas suportados para tradução e solicita que o usuário selecione um código de idioma para o idioma de destino:

    ```python
   # Choose target language
   languagesResponse = client.get_supported_languages(scope="translation")
   print("{} languages supported.".format(len(languagesResponse.translation)))
   print("(See https://learn.microsoft.com/azure/ai-services/translator/language-support#translation)")
   print("Enter a target language code for translation (for example, 'en'):")
   targetLanguage = "xx"
   supportedLanguage = False
   while supportedLanguage == False:
        targetLanguage = input()
        if  targetLanguage in languagesResponse.translation.keys():
            supportedLanguage = True
        else:
            print("{} is not a supported language.".format(targetLanguage))
    ```

1. Localize o comentário **Translate text** e adicione o seguinte código, que solicita repetidamente ao usuário o texto a ser traduzido, usa o serviço de Tradutor de IA do Azure para traduzi-lo para o idioma de destino (detectando automaticamente o idioma de origem) e exibe os resultados até que o usuário digite *quit*:

    ```python
   # Translate text
   inputText = ""
   while inputText.lower() != "quit":
        inputText = input("Enter text to translate ('quit' to exit):")
        if inputText != "quit":
            input_text_elements = [InputTextItem(text=inputText)]
            translationResponse = client.translate(body=input_text_elements, to_language=[targetLanguage])
            translation = translationResponse[0] if translationResponse else None
            if translation:
                sourceLanguage = translation.detected_language
                for translated_text in translation.translations:
                    print(f"'{inputText}' was translated from {sourceLanguage.language} to {translated_text.to} as '{translated_text.text}'.")
    ```

1. Salve suas alterações (CTRL+S) e, em seguida, insira o seguinte comando para executar o programa (você maximiza o painel do Cloud Shell e redimensiona os painéis para ver mais texto no painel de linha de comando):

    ```
   python translate.py
    ```

1. Quando solicitado, insira um idioma de destino válido na lista exibida.
1. Insira uma frase a ser traduzida (por exemplo `This is a test` ou `C'est un test`) e exiba os resultados, que devem detectar o idioma de origem e traduzir o texto para o idioma de destino.
1. Quando terminar, digite `quit`. Você pode executar o aplicativo novamente e escolher um idioma de destino diferente.

## Limpar os recursos

Se você terminou de explorar o serviço de Tradutor de IA do Azure, pode excluir os recursos que criou neste exercício. Este é o procedimento:

1. Feche o painel do Azure Cloud Shell
1. No portal do Azure, navegue até o recurso de Tradutor de IA do Azure criado neste laboratório.
1. Na página de recursos, selecione **Excluir** e siga as instruções para excluir o recurso.

## Mais informações

Para obter mais informações sobre o uso do **Tradutor de IA do Azure**, consulte a [documentação do Tradutor de IA do Azure ](https://learn.microsoft.com/azure/ai-services/translator/).
