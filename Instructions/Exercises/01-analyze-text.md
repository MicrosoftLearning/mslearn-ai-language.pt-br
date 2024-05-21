---
lab:
  title: Analisar o texto
  module: Module 3 - Develop natural language processing solutions
---

# Analisar o texto

A **Linguagem do Azure** permite análise de texto, incluindo detecção de idioma, análise de sentimento, extração de frases-chave e reconhecimento de entidade.

Por exemplo, suponha que uma agência de viagens queira processar avaliações de hotéis que foram enviadas ao site da empresa. Usando a Linguagem de IA do Azure, eles podem determinar o idioma em que cada avaliação está escrita, o sentimento (positivo, neutro ou negativo) das avaliações, frases-chave que podem indicar os principais tópicos discutidos na avaliação e entidades nomeadas, como lugares, pontos de referência ou pessoas mencionadas.

## Provisionar um recurso de *Linguagem de IA do Azure*

Caso ainda não tenha um na sua assinatura, provisione um recurso do **serviço de Linguagem de IA do Azure** na sua assinatura do Azure.

1. Abra o portal do Azure em `https://portal.azure.com` e entre usando a conta Microsoft associada à sua assinatura do Azure.
1. Usando o campo de pesquisa na parte superior, pesquise **serviços de IA do Azure**. Nos resultados, selecione **Criar** em **Serviço de Linguagem**.
1. Selecione **Continuar para criar o recurso**.
1. Provisione o recurso usando as seguintes configurações:
    - **Assinatura**: *sua assinatura do Azure*.
    - **Grupo de recursos**: *crie ou escolha um grupo de recursos*.
    - **Região**:*escolha uma região disponível*
    - **Nome**: *insira um nome exclusivo*.
    - **Tipo de preço**: selecione **F0** (*gratuito*) ou **S** (*padrão*) se F não estiver disponível.
    - **Aviso de IA responsável**: concordar
1. Selecione **Revisar + criar** e, em seguida, selecione **Criar** para provisionar o recurso.
1. Aguarde a conclusão da implantação e acesse o recurso implantado.
1. Exiba a página **Chaves e Ponto de Extremidade**. Você precisará das informações desta página mais adiante no exercício.

## Preparar-se para desenvolver um aplicativo no Visual Studio Code

Você desenvolverá seu aplicativo de análise de texto usando o Visual Studio Code. Os arquivos de código do seu aplicativo foram fornecidos em um repositório do GitHub.

> **Dica**: se você já clonou o repositório **mslearn-ai-language**, abra-o no Visual Studio Code. Caso contrário, siga estas etapas para cloná-lo em seu ambiente de desenvolvimento.

1. Inicie o Visual Studio Code.
2. Abra a paleta (SHIFT+CTRL+P) e execute o comando **Git: Clone** para clonar o repositório `https://github.com/MicrosoftLearning/mslearn-ai-language` em uma pasta local (não importa qual pasta).
3. Depois que o repositório for clonado, abra a pasta no Visual Studio Code.
4. Aguarde enquanto os arquivos adicionais são instalados para dar suporte aos projetos de código C# no repositório.

    > **Observação**: se você for solicitado a adicionar os ativos necessários para compilar e depurar, selecione **Agora não**.

## Configurar seu aplicativo

Aplicativos para C# e Python foram fornecidos, bem como um arquivo de texto de exemplo que você usará para testar o resumo. Ambos os aplicativos apresentam a mesma funcionalidade. Primeiro, você concluirá algumas partes importantes do aplicativo para habilitar o uso do recurso de Linguagem de IA do Azure.

1. No Visual Studio Code, no painel **Explorer**, navegue até a pasta **Labfiles/01-analyze-text** e expanda a pasta **CSharp** ou **Python**, dependendo da sua preferência de idioma, e a pasta **text-analysis** contida dentro dela. Cada pasta contém os arquivos específicos de idioma de um aplicativo ao qual você integrará a funcionalidade de análise de texto da Linguagem de IA do Azure.
2. Clique com o botão direito na pasta **text-analysis** que contém seus arquivos de código e abra um terminal integrado. Instale o pacote do SDK de Análise de Texto da Linguagem de IA do Azure executando o comando apropriado para sua preferência de idioma. Para o exercício em Python, instale também o pacote `dotenv`:

    **C#**:

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.3.0
    ```

    **Python**:

    ```
    pip install azure-ai-textanalytics==5.3.0
    pip install python-dotenv
    ```

3. No painel **Explorer**, na pasta **text-analysis**, abra o arquivo de configuração do seu idioma preferido

    - **C#**: appsettings.json
    - **Python**: .env
    
4. Atualize os valores de configuração para incluir o **ponto de extremidade** e uma **chave** do recurso de Linguagem do Azure que você criou (disponível na página **Chaves e Ponto de Extremidade** do seu recurso de Linguagem de IA do Azure no portal do Azure)
5. Salve o arquivo de configuração.

6. Observe que a pasta **text-analysis** contém um arquivo de código para o aplicativo cliente:

    - **C#**: Program.cs
    - **Python**: text-analysis.py

    Abra o arquivo de código e, na parte superior, sob as referências de namespace existentes, localize o comentário **Importar namespaces**. Neste comentário, adicione o seguinte código específico do idioma para importar os namespaces necessários para usar o SDK de Análise de Texto:

    **C#**: Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.TextAnalytics;
    ```

    **Python**: text-analysis.py

    ```python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.textanalytics import TextAnalyticsClient
    ```

7. Na função **Principal**, observe que o código para carregar o ponto de extremidade e a chave do serviço de Linguagem de IA do Azure do arquivo de configuração já foi fornecido. Em seguida, localize o comentário **Criar cliente usando ponto de extremidade e chave** e adicione o seguinte código para criar um cliente para a API de Análise de Texto:

    **C#**: Programs.cs

    ```C#
    // Create client using endpoint and key
    AzureKeyCredential credentials = new AzureKeyCredential(aiSvcKey);
    Uri endpoint = new Uri(aiSvcEndpoint);
    TextAnalyticsClient aiClient = new TextAnalyticsClient(endpoint, credentials);
    ```

    **Python**: text-analysis.py

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(ai_key)
    ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

8. Salve suas alterações e retorne ao terminal integrado da pasta **text-analysis** e digite o seguinte comando para executar o programa:

    - **C#**: `dotnet run`
    - **Python**: `python text-analysis.py`

    > **Dica**: você pode usar o ícone **Maximizar tamanho do painel** (**^**) na barra de ferramentas do terminal para ver mais do texto do console.

9. Observe a saída, pois o código deve ser executado sem erros, exibindo o conteúdo de cada arquivo de texto de avaliação na pasta **comentários** O aplicativo cria com êxito um cliente para a API de Análise de Texto, mas não o utiliza. Vamos corrigir isso no próximo procedimento.

## Adicionar código para detectar linguagem

Agora que você criou um cliente para a API, vamos usá-lo para detectar o idioma no qual cada avaliação foi escrita.

1. Na função **Principal** do seu programa, localize o comentário **Obter idioma**. Em seguida, sob este comentário, adicione o código necessário para detectar o idioma em cada documento de avaliação:

    **C#**: Programs.cs

    ```csharp
    // Get language
    DetectedLanguage detectedLanguage = aiClient.DetectLanguage(text);
    Console.WriteLine($"\nLanguage: {detectedLanguage.Name}");
    ```

    **Python**: text-analysis.py

    ```python
    # Get language
    detectedLanguage = ai_client.detect_language(documents=[text])[0]
    print('\nLanguage: {}'.format(detectedLanguage.primary_language.name))
    ```

     > **Observação**: *neste exemplo, cada avaliação é analisada individualmente, o que resulta em uma chamada separada ao serviço para cada arquivo. Uma abordagem alternativa é criar uma coleção de documentos e transmiti-los para o serviço em uma única chamada. Em ambas as abordagens, a resposta do serviço consiste em uma coleção de documentos; é por isso que no código Python acima, o índice do primeiro (e único) documento na resposta ([0]) é especificado.*

1. Salve suas alterações. Retorne ao terminal integrado para a pasta **text-analysis** e execute novamente o programa.
1. Observe a saída, que desta vez identifica a linguagem para cada avaliação.

## Adicionar código para avaliar o sentimento

A *análise de sentimento* é uma técnica comumente usada para classificar o texto como *positivo* ou *negativo* (ou possivelmente *neutro* ou *misto*). É comumente usada para analisar postagens de mídia social, avaliações de produtos e outros itens em que o sentimento do texto pode fornecer insights úteis.

1. Na função **Principal** do seu programa, localize o comentário **Obter sentimento**. Em seguida, sob este comentário, adicione o código necessário para detectar o sentimento de cada documento de avaliação:

    **C#**: Program.cs

    ```csharp
    // Get sentiment
    DocumentSentiment sentimentAnalysis = aiClient.AnalyzeSentiment(text);
    Console.WriteLine($"\nSentiment: {sentimentAnalysis.Sentiment}");
    ```

    **Python**: text-analysis.py

    ```python
    # Get sentiment
    sentimentAnalysis = ai_client.analyze_sentiment(documents=[text])[0]
    print("\nSentiment: {}".format(sentimentAnalysis.sentiment))
    ```

1. Salve suas alterações. Retorne ao terminal integrado para a pasta **text-analysis** e execute novamente o programa.
1. Observe a saída, que agora identifica o sentimento das avaliações.

## Adicionar código para identificar frases-chave

Pode ser útil identificar frases-chave no corpo do texto para ajudar a determinar os principais tópicos discutidos.

1. Na função **Principal** do seu programa, localize o comentário **Obter frases-chave**. Em seguida, sob este comentário, adicione o código necessário para detectar as frases-chave em cada documento de avaliação:

    **C#**: Program.cs

    ```csharp
    // Get key phrases
    KeyPhraseCollection phrases = aiClient.ExtractKeyPhrases(text);
    if (phrases.Count > 0)
    {
        Console.WriteLine("\nKey Phrases:");
        foreach(string phrase in phrases)
        {
            Console.WriteLine($"\t{phrase}");
        }
    }
    ```

    **Python**: text-analysis.py

    ```python
    # Get key phrases
    phrases = ai_client.extract_key_phrases(documents=[text])[0].key_phrases
    if len(phrases) > 0:
        print("\nKey Phrases:")
        for phrase in phrases:
            print('\t{}'.format(phrase))
    ```

1. Salve suas alterações. Retorne ao terminal integrado para a pasta **text-analysis** e execute novamente o programa.
1. Observe a saída, cada documento contém frases-chave que fornecem alguns insights sobre o objeto da avaliação.

## Adicionar código para extrair entidades

Muitas vezes, documentos ou outros corpos de texto mencionam pessoas, lugares, períodos de tempo ou outras entidades. A API de Análise de Texto pode detectar várias categorias (e subcategorias) de entidades em seu texto.

1. Na função **Principal** do seu programa, localize o comentário **Obter entidades**. Em seguida, sob este comentário, adicione o código necessário para identificar as entidades mencionadas em cada documento de avaliação:

    **C#**: Program.cs

    ```csharp
    // Get entities
    CategorizedEntityCollection entities = aiClient.RecognizeEntities(text);
    if (entities.Count > 0)
    {
        Console.WriteLine("\nEntities:");
        foreach(CategorizedEntity entity in entities)
        {
            Console.WriteLine($"\t{entity.Text} ({entity.Category})");
        }
    }
    ```

    **Python**: text-analysis.py

    ```python
    # Get entities
    entities = ai_client.recognize_entities(documents=[text])[0].entities
    if len(entities) > 0:
        print("\nEntities")
        for entity in entities:
            print('\t{} ({})'.format(entity.text, entity.category))
    ```

1. Salve suas alterações. Retorne ao terminal integrado para a pasta **text-analysis** e execute novamente o programa.
1. Observe a saída, que identifica as entidades que foram detectadas no texto.

## Adicionar código para extrair entidades vinculadas

Além das entidades categorizadas, a API de Análise de Texto pode detectar entidades para as quais há links conhecidos para fontes de dados, como a Wikipédia.

1. Na função **Principal** do seu programa, localize o comentário **Obter entidades vinculadas**. Em seguida, sob este comentário, adicione o código necessário para identificar as entidades vinculadas mencionadas em cada documento de avaliação:

    **C#**: Program.cs

    ```csharp
    // Get linked entities
    LinkedEntityCollection linkedEntities = aiClient.RecognizeLinkedEntities(text);
    if (linkedEntities.Count > 0)
    {
        Console.WriteLine("\nLinks:");
        foreach(LinkedEntity linkedEntity in linkedEntities)
        {
            Console.WriteLine($"\t{linkedEntity.Name} ({linkedEntity.Url})");
        }
    }
    ```

    **Python**: text-analysis.py

    ```python
    # Get linked entities
    entities = ai_client.recognize_linked_entities(documents=[text])[0].entities
    if len(entities) > 0:
        print("\nLinks")
        for linked_entity in entities:
            print('\t{} ({})'.format(linked_entity.name, linked_entity.url))
    ```

1. Salve suas alterações. Retorne ao terminal integrado para a pasta **text-analysis** e execute novamente o programa.
1. Observe a saída, que identifica as entidades vinculadas que foram detectadas.

## Limpar os recursos

Dica: se você tiver concluído a exploração do serviço de Linguagem de IA do Azure, exclua os recursos criados no exercício. Este é o procedimento:

1. Abra o portal do Azure em `https://portal.azure.com` e entre usando a conta Microsoft associada à sua assinatura do Azure.

2. Navegue até o recurso de Linguagem de IA do Azure que você criou neste laboratório.

3. Na página do recurso, selecione **Excluir** e siga as instruções para excluir o recurso.

## Mais informações

Para obter mais informações sobre como usar a **Linguagem de IA do Azure**, consulte a [documentação](https://learn.microsoft.com/azure/ai-services/language-service/).
