---
lab:
  title: Traduzir o texto
  module: Module 3 - Getting Started with Natural Language Processing
---
{% assign site.title = page.lab.title %}

# Traduzir o texto

O **Tradutor de IA do Azure** é um serviço que permite a você traduzir textos entre idiomas. Neste exercício, você o usará para criar um aplicativo simples que traduz a entrada em qualquer idioma suportado para o idioma destino de sua escolha.

## Provisionar um recurso do *Tradutor de IA do Azure*

Caso ainda não tenha um na sua assinatura, provisione um recurso do **Tradutor de IA do Azure**.

1. Abra o portal do Azure em `https://portal.azure.com` e entre usando a conta Microsoft associada à sua assinatura do Azure.
1. No campo de pesquisa superior, procure os **Serviços de IA do Azure** e pressione **Enter** e selecione **Criar** em **Tradutor** nos resultados.
1. Crie um recurso com as seguintes configurações:
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *escolher ou criar um grupo de recursos*
    - **Região**: *escolha uma região disponível*
    - **Nome**: *insira um nome exclusivo*
    - **Tipo de preço**: selecione **F0** (*gratuito*), ou **S** (*Standard*) se F não estiver disponível.
    - **Aviso de IA responsável**: concordar
1. Selecione **Revisar + criar** e, em seguida, selecione **Criar** para provisionar o recurso.
1. Aguarde a conclusão da implantação e acesse o recurso implantado.
1. Exiba a página **Chaves e Ponto de Extremidade**. Você precisará das informações desta página mais adiante no exercício.

## Prepare-se para desenvolver um aplicativo no Visual Studio Code

Você desenvolverá seu aplicativo de tradução de texto usando o Visual Studio Code. Os arquivos de código para seu aplicativo foram fornecidos em um repositório do GitHub.

> **Dica**: se você já clonou o repositório **mslearn-ai-language**, abra-o no Visual Studio Code. Caso contrário, siga estas etapas para cloná-lo em seu ambiente de desenvolvimento.

1. Inicie o Visual Studio Code.
2. Abra a paleta (SHIFT+CTRL+P) e execute o comando **Git: Clone** para clonar o repositório `https://github.com/MicrosoftLearning/mslearn-ai-language` em uma pasta local (não importa qual pasta).
3. Depois que o repositório for clonado, abra a pasta no Visual Studio Code.

    > **Observação**: Se o Visual Studio Code mostrar uma mensagem pop-up para solicitar que você confie no código que está abrindo, clique na opção **Sim, confio nos autores** no pop-up.

4. Aguarde enquanto os arquivos adicionais são instalados para dar suporte aos projetos de código C# no repositório.

    > **Observação**: se você for solicitado a adicionar os ativos necessários para compilar e depurar, selecione **Agora não**.

## Configurar seu aplicativo

Os aplicativos para C# e Python foram fornecidos. Ambos os aplicativos apresentam a mesma funcionalidade. Primeiro, conclua algumas partes principais do aplicativo para habilitar o uso do recurso Tradutor de IA do Azure.

1. No Visual Studio Code, no painel **Explorer**, navegue até a pasta **Labfiles/06b-translator-sdk** e expanda a pasta **CSharp** ou **Python** dependendo de sua preferência de linguagem e da pasta **translate text** que ela contém. Cada pasta contém os arquivos de código específicos do idioma de um aplicativo no qual você integrará a funcionalidade do Tradutor de IA do Azure.
2. Clique com o botão direito do mouse na pasta **translate-text** que contém seus arquivos de código e abra um terminal integrado. Em seguida, instale o pacote SDK de Tradutor de IA do Azure executando o comando apropriado para sua preferência de linguagem:

    **C#**:

    ```
    dotnet add package Azure.AI.Translation.Text --version 1.0.0-beta.1
    ```

    **Python**:

    ```
    pip install azure-ai-translation-text==1.0.0b1
    ```

3. No painel **Explorer**, na pasta **translate-text**, abra o arquivo de configuração da sua linguagem de preferência

    - **C#**: appsettings.json
    - **Python**: .env
    
4. Atualize os valores de configuração para incluir a **região** e uma **chave** do recurso do Tradutor de IA do Azure que você criou (disponível na página **Chaves e Ponto de Extremidade** para seu recurso do Tradutor de IA do Azure no portal do Azure).

    > **Observação**: certifique-se de adicionar a *região* para o seu recurso, <u>não</u> para o ponto de extremidade!

5. Salve o arquivo de configuração.

## Adicionar código para traduzir texto

Agora você está pronto para usar o Tradutor de IA do Azure para traduzir texto.

1. Observe que a pasta **translate-text** contém um arquivo de código para o aplicativo cliente:

    - **C#**: Program.cs
    - **Python**: translate.py

    Abra o arquivo de código e, na parte superior, sob as referências de namespace existentes, localize o comentário **Importar namespaces**. Neste comentário, adicione o seguinte código específico do idioma para importar os namespaces necessários para usar o SDK de Análise de Texto:

    **C#**: Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.Translation.Text;
    ```

    **Python**: translate.py

    ```python
    # import namespaces
    from azure.ai.translation.text import *
    from azure.ai.translation.text.models import InputTextItem
    ```

1. Na função **Principal**, observe que o código existente lê as definições de configuração.
1. Localize o comentário **Criar cliente usando ponto de extremidade e chave** e adicione o seguinte código:

    **C#**: Programs.cs

    ```csharp
    // Create client using endpoint and key
    AzureKeyCredential credential = new(translatorKey);
    TextTranslationClient client = new(credential, translatorRegion);
    ```

    **Python**: translate.py

    ```python
    # Create client using endpoint and key
    credential = TranslatorCredential(translatorKey, translatorRegion)
    client = TextTranslationClient(credential)
    ```

1. Localize o comentário **Escolher idioma de destino** e adicione o código a seguir, que usa o serviço Tradutor de Texto para retornar a lista de idiomas com suporte para tradução e solicita que o usuário selecione um código de idioma para o idioma de destino.

    **C#**: Programs.cs

    ```csharp
    // Choose target language
    Response<GetLanguagesResult> languagesResponse = await client.GetLanguagesAsync(scope:"translation").ConfigureAwait(false);
    GetLanguagesResult languages = languagesResponse.Value;
    Console.WriteLine($"{languages.Translation.Count} languages available.\n(See https://learn.microsoft.com/azure/ai-services/translator/language-support#translation)");
    Console.WriteLine("Enter a target language code for translation (for example, 'en'):");
    string targetLanguage = "xx";
    bool languageSupported = false;
    while (!languageSupported)
    {
        targetLanguage = Console.ReadLine();
        if (languages.Translation.ContainsKey(targetLanguage))
        {
            languageSupported = true;
        }
        else
        {
            Console.WriteLine($"{targetLanguage} is not a supported language.");
        }

    }
    ```

    **Python**: translate.py

    ```python
    # Choose target language
    languagesResponse = client.get_languages(scope="translation")
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

1. Localize o comentário **Traduzir texto** e adicione o código a seguir que, solicita repetidamente ao usuário pelo texto a ser traduzido, usa o serviço de Tradutor de IA do Azure para traduzí-lo para o idioma de destino (detectando o idioma de origem automaticamente) e exibe os resultados até que o usuário digite *quit*.

    **C#**: Programs.cs

    ```csharp
    // Translate text
    string inputText = "";
    while (inputText.ToLower() != "quit")
    {
        Console.WriteLine("Enter text to translate ('quit' to exit)");
        inputText = Console.ReadLine();
        if (inputText.ToLower() != "quit")
        {
            Response<IReadOnlyList<TranslatedTextItem>> translationResponse = await client.TranslateAsync(targetLanguage, inputText).ConfigureAwait(false);
            IReadOnlyList<TranslatedTextItem> translations = translationResponse.Value;
            TranslatedTextItem translation = translations[0];
            string sourceLanguage = translation?.DetectedLanguage?.Language;
            Console.WriteLine($"'{inputText}' translated from {sourceLanguage} to {translation?.Translations[0].To} as '{translation?.Translations?[0]?.Text}'.");
        }
    } 
    ```

    **Python**: translate.py

    ```python
    # Translate text
    inputText = ""
    while inputText.lower() != "quit":
        inputText = input("Enter text to translate ('quit' to exit):")
        if inputText != "quit":
            input_text_elements = [InputTextItem(text=inputText)]
            translationResponse = client.translate(content=input_text_elements, to=[targetLanguage])
            translation = translationResponse[0] if translationResponse else None
            if translation:
                sourceLanguage = translation.detected_language
                for translated_text in translation.translations:
                    print(f"'{inputText}' was translated from {sourceLanguage.language} to {translated_text.to} as '{translated_text.text}'.")
    ```

1. Salve as alterações no arquivo de código.

## Teste seu aplicativo

Agora, seu aplicativo está pronto para teste.

1. No terminal integrado para a pasta **Traduzir texto**, digite o seguinte comando para executar o programa:

    - **C#**: `dotnet run`
    - **Python**: `python translate.py`

    > **Dica**: você pode usar o ícone **Maximizar o tamanho do painel** (**^**) na barra de ferramentas do terminal para ver mais do texto do console.

1. Quando solicitado, insira um idioma de destino válido na lista exibida.
1. Insira uma frase a ser traduzida (por exemplo `This is a test` ou `C'est un test`) e exiba os resultados, que devem detectar o idioma de origem e traduzir o texto para o idioma de destino.
1. Quando terminar, digite `quit`. Você pode executar o aplicativo novamente e escolher um idioma de destino diferente.

## Limpeza

Quando não precisar mais do projeto, você poderá excluir o recurso de Tradutor de IA do Azure no [portal do Azure](https://portal.azure.com).
