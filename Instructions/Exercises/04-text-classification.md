---
lab:
  title: Classificação personalizada de textos
  module: Module 3 - Getting Started with Natural Language Processing
---

# Classificação personalizada de textos

A Linguagem de IA do Azure fornece vários recursos de PNL, incluindo identificação de frase-chave, resumo de texto e análise de sentimento. O serviço de Linguagem também fornece funcionalidades personalizadas, como respostas às perguntas personalizadas e classificação de textos personalizada.

Para testar a classificação de textos personalizada do serviço de Linguagem de IA do Azure, configuraremos o modelo usando o Language Studio e usaremos um pequeno aplicativo de linha de comando executado no Cloud Shell para testá-lo. O mesmo padrão e funcionalidade usados aqui podem ser seguidos para aplicativos do mundo real.

## Provisionar um recurso da *Linguagem de IA do Azure*

Caso ainda não tenha um na sua assinatura, provisione um recurso do **serviço de Linguagem de IA do Azure**. Além disso, use a classificação de textos personalizada; você precisa habilitar o **recurso de extração e classificação de textos personalizada**.

1. Em um navegador, abra o portal do Azure em `https://portal.azure.com` e entre com sua conta Microsoft.
1. Selecione o campo de pesquisa na parte superior do portal, pesquise `Azure AI services` e crie um recurso do **Serviço de Linguagem**.
1. Selecione a caixa que inclui **Classificação de texto personalizada**. Em seguida, selecione **Continuar para criar o recurso**.
1. Crie um recurso com as seguintes configurações:
    - **Assinatura**: *sua assinatura do Azure*.
    - **Grupo de recursos**: *selecione ou crie um grupo de recursos*.
    - **Região**: *escolha qualquer região disponível*:
    - **Nome**: *insira um nome exclusivo*.
    - **Tipo de preço**: selecione **F0** (*gratuito*), ou **S** (*Standard*) se F não estiver disponível.
    - **Conta de armazenamento**: nova conta de armazenamento
      - **Nome da conta de armazenamento**: *insira um nome exclusivo*.
      - **Tipo de conta de armazenamento**: LRS Standard
    - **Aviso de IA Responsável**: selecionado.

1. Selecione **Revisar + criar** e, em seguida, selecione **Criar** para provisionar o recurso.
1. Aguarde a conclusão da implantação e acesse o recurso implantado.
1. Exiba a página **Chaves e Ponto de Extremidade**. Você precisará das informações nesta página mais adiante no exercício.

## Carregar artigos de exemplo

Após criar o serviço de Linguagem de IA do Azure e a conta de armazenamento, você precisará carregar artigos de exemplo para treinar o modelo mais tarde.

1. Em uma nova guia do navegador, baixe artigos de amostra de `https://aka.ms/classification-articles` e extraia os arquivos para uma pasta de sua escolha.

1. No portal do Azure, navegue até a conta de armazenamento que você criou e selecione-a.

1. Na sua conta de armazenamento, selecione **Configuração**, localizada abaixo de **Configurações**. Na tela configuração, habilite a opção **Permitir acesso anônimo de blob** e selecione **Salvar**.

1. Selecione **Contêineres** no menu esquerdo localizado abaixo de **Armazenamento de dados**. Na tela exibida, selecione **+ Contêiner**. Dê ao contêiner o nome `articles` e defina **Nível de acesso anônimo** como **Contêiner (acesso de leitura anônimo para contêineres e blobs)**.

    > **OBSERVAÇÃO**: ao configurar uma conta de armazenamento para uma solução real, tome cuidado para atribuir o nível de acesso apropriado. Para saber mais sobre cada nível de acesso, confira a [documentação de armazenamento do Azure](https://learn.microsoft.com/azure/storage/blobs/anonymous-read-access-configure).

1. Depois de criar o contêiner, selecione-o e selecione o botão **Carregar**. Selecione **Procurar arquivos** para procurar os artigos de exemplo baixados. Em seguida, selecione **Carregar**.

## Criar um projeto de classificação de textos personalizada

Após a configuração ser concluída, crie um projeto de classificação de textos personalizada. Esse projeto proporciona um local de trabalho para criar, treinar e implantar o modelo.

> **OBSERVAÇÃO**: este laboratório utiliza o **Language Studio**, mas você também pode compilar, treinar e implantar um modelo usando a API REST.

1. Em uma nova guia do navegador, abra o portal do Azure AI Language Studio em `https://language.cognitive.azure.com/` e entre usando a conta Microsoft associada à sua assinatura do Azure.
1. Se for solicitado a escolher um recurso de linguagem, selecione as seguintes configurações:

    - **Azure Directory**: o diretório do Azure contendo a sua assinatura.
    - **Assinatura do Azure**: sua assinatura do Azure.
    - **Tipo de recurso**: linguagem.
    - **Recurso de linguagem**: o recurso de Linguagem de IA do Azure criado anteriormente.

    Se você <u>não</u> foi solicitado a escolher um recurso de linguagem, pode ser porque você tem vários recursos de linguagem em sua assinatura; nesse caso:

    1. Na barra na parte superior da página, clique no botão **Configurações (&#9881;)**.
    2. Na página **Configurações**, exiba a guia **Recursos**.
    3. Selecione o recurso de linguagem que você acabou de criar e clique em **Alternar recurso**.
    4. Na parte superior da página, clique em **Language Studio** para retornar à página incial do Language Studio

1. Na parte superior do portal, no menu **Criar novo**, selecione **Classificação de textos personalizada**.
1. A página **Conectar armazenamento** é exibida. Todos os valores já terão sido preenchidos. Portanto, selecione **Avançar**.
1. Na página **Selecionar tipo de projeto**, selecione **Classificação de rótulo único**. Em seguida, selecione **Avançar**.
1. No painel **Inserir informações básicas**, defina o seguinte:
    - **Nome**: `ClassifyLab`  
    - **Idioma primário do texto**: inglês (EUA)
    - **Descrição**: `Custom text lab`

1. Selecione **Avançar**.
1. Na página **Escolher contêiner**, defina a lista suspensa **Contêiner do repositório de blobs** como seu contêiner de *artigos*.
1. Selecione a opção **Não, eu preciso rotular meus arquivos como parte deste projeto**. Em seguida, selecione **Avançar**.
1. Selecione **Criar projeto**.

> **Dica**: se você receber um erro sobre não ter autorização para executar essa operação, será necessário adicionar uma atribuição de função. Para corrigir isso, adicionamos a função "Colaborador de Dados de Blob de Armazenamento" na conta de armazenamento do usuário que está executando o laboratório. Você encontra mais detalhes na [página da documentação](https://learn.microsoft.com/azure/ai-services/language-service/custom-named-entity-recognition/how-to/create-project?tabs=portal%2Clanguage-studio#enable-identity-management-for-your-resource).

## Rotular seus dados.

Agora que o projeto foi criado, você precisa rotular ou marcar seus dados para treinar o modelo na classificação de textos.

1. À esquerda, selecione **Rotulagem de dados**, se ainda não estiver selecionada. Você verá uma lista dos arquivos que carregou na conta de armazenamento.
1. No lado direito, no painel **Atividade**, selecione **+ Adicionar classe**.  Os artigos neste laboratório se encaixam em quatro classes que você precisará criar: `Classifieds`, `Sports`, `News` e `Entertainment`.

    ![Captura de tela mostrando a página de marcação de dados e o botão para adicionar uma classe.](../media/tag-data-add-class-new.png#lightbox)

1. Depois de criar suas quatro classes, selecione **Artigo 1** para iniciar. Aqui, você pode ler o artigo, definir a classe o arquivo pertence e para qual conjunto de dados (treinamento ou teste) atribui-lo.
1. Atribua a cada artigo à classe e ao conjunto de dados apropriado (treinamento ou teste) usando o painel **Atividade** à direita.  Você pode selecionar um rótulo na lista de laboratórios à direita e definir cada artigo como **treinamento** ou **teste** usando as opções na parte inferior do painel Atividade. Selecione **Próximo documento** para ir ao próximo documento. Para fins do laboratório, definiremos quais serão usados para treinar o modelo e quais serão usados para testá-lo:

    | Artigo  | Classe  | Dataset  |
    |---------|---------|---------|
    | Artigo 1 | Esportes | Treinamento |
    | Artigo 10 | News | Treinamento |
    | Artigo 11 | Entretenimento | Teste |
    | Artigo 12 | News | Teste |
    | Artigo 13 | Esportes | Teste |
    | Artigo 2 | Esportes | Treinamento |
    | Artigo 3 | Classificados | Treinamento |
    | Artigo 4 | Classificados | Treinamento |
    | Artigo 5 | Entretenimento | Treinamento |
    | Artigo 6 | Entretenimento | Treinamento |
    | Artigo 7 | News | Treinamento |
    | Artigo 8 | News | Treinamento |
    | Artigo 9 | Entretenimento | Treinamento |

    > **OBSERVAÇÃO** Os arquivos no Language Studio são listados em ordem alfabética, razão pela qual a lista acima não está em ordem sequencial. Visite as duas páginas dos documentos ao rotular seus artigos.

1. Selecione **Salvar rótulos** para salvar seus rótulos.

## Treinar seu modelo

Depois de rotular os dados, você precisa treinar o modelo.

1. Selecione **Trabalhos de treinamento** no menu esquerdo.
1. Escolha **Iniciar um trabalho de treinamento**.
1. Treine um novo modelo chamado `ClassifyArticles`.
1. Selecione **Usar uma divisão manual de dados de treinamento e de teste**.

    > **DICA** Em seus projetos de classificação, o serviço de Linguagem de IA do Azure dividirá automaticamente o conjunto de teste por porcentagem, o que é útil com um conjunto de dados grande. Com conjuntos de dados menores, é importante treinar com a distribuição de classes correta.

1. Selecione **Treinar**

> **IMPORTANTE** Às vezes, o treinamento do modelo pode levar vários minutos. Você receberá uma notificação quando ele for concluído.

## Avaliar o modelo

Em aplicativos de classificação de texto do mundo real, é importante avaliar e aprimorar o modelo para verificar se ele está funcionando conforme o esperado.

1. Selecione **Desempenho do modelo** e clique em seu modelo **ClassifyArticles**. Lá, você pode ver a pontuação do modelo, as métricas de desempenho e quando ele foi treinado. Se a pontuação do modelo não for 100%, isso significa que um dos documentos usados para teste não foi avaliado para o que foi rotulado. Essas falhas podem ajudá-lo a entender onde melhorar.
1. Selecione a guia **Detalhes do conjunto de testes**. Se houver erros, essa guia permitirá que você veja os artigos indicados para teste e como o modelo os previu, e se isso está em conflito com o rótulo de teste. O padrão da guia é mostrar apenas previsões incorretas. Você pode alternar a opção **Mostrar incompatibilidades somente** para ver todos os artigos indicados para teste e a previsão de cada um.

## Implantar o seu modelo

Quando estiver satisfeito com o treinamento do seu modelo, é hora de implantá-lo, o que permite que você comece a classificar texto usando a API.

1. No painel esquerdo, selecione **Implantar modelo**.
1. Selecione **Adicionar implantação**, insira `articles` no campo **Criar um novo nome de implantação** e selecione **ClassifyArticles** no campo **Modelo**.
1. Selecione **Implantar** para implantar seu modelo.
1. Depois que o modelo for implantado, deixe essa página aberta. Você precisará do nome do projeto e da implantação na próxima etapa.

## Prepare-se para desenvolver um aplicativo no Visual Studio Code

Para testar os recursos de classificação de textos personalizada do serviço de Linguagem de IA do Azure, você desenvolverá um aplicativo de console simples no Visual Studio Code.

> **Dica**: se você já clonou o repositório **mslearn-ai-language**, abra-o no Visual Studio Code. Caso contrário, siga estas etapas para cloná-lo em seu ambiente de desenvolvimento.

1. Inicie o Visual Studio Code.
2. Abra a paleta (SHIFT+CTRL+P) e execute o comando **Git: Clone** para clonar o repositório `https://github.com/MicrosoftLearning/mslearn-ai-language` em uma pasta local (não importa qual pasta).
3. Depois que o repositório for clonado, abra a pasta no Visual Studio Code.

    > **Observação**: Se o Visual Studio Code mostrar uma mensagem pop-up para solicitar que você confie no código que está abrindo, clique na opção **Sim, confio nos autores** no pop-up.

4. Aguarde enquanto os arquivos adicionais são instalados para dar suporte aos projetos de código C# no repositório.

    > **Observação**: se você for solicitado a adicionar os ativos necessários para compilar e depurar, selecione **Agora não**.

## Configurar seu aplicativo

Aplicativos para C# e Python foram fornecidos, bem como um arquivo de texto de exemplo que você usará para testar o resumo. Ambos os aplicativos apresentam a mesma funcionalidade. Primeiro, você concluirá algumas partes importantes do aplicativo para habilitar o uso do recurso Linguagem de IA do Azure.

1. No Visual Studio Code, no painel **Explorer**, navegue até a pasta **Labfiles/04-text-classification** e expanda a pasta **CSharp** ou **Python** dependendo de sua preferência de linguagem e da pasta **classify-text** que ela contém. Cada pasta contém os arquivos específicos de linguagem do aplicativo no qual você integrará a funcionalidade de classificação de textos de Linguagem de IA do Azure.
1. Clique com o botão direito do mouse na pasta **classify-text** que contém seus arquivos de código e abra um terminal integrado. Em seguida, instale o pacote do SDK de análise de texto da Linguagem de IA do Azure executando o comando apropriado para sua preferência de idioma:

    **C#**:

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.3.0
    ```

    **Python**:

    ```
    pip install azure-ai-textanalytics==5.3.0
    ```

1. No painel **Explorer**, na pasta **classify-text**, abra o arquivo de configuração da linguagem de sua preferência

    - **C#**: appsettings.json
    - **Python**: .env
    
1. Atualize os valores de configuração para incluir o  **ponto de extremidade** e uma **chave** do recurso de linguagem do Azure que você criou (disponível na página **Chaves e Ponto de Extremidade** para seu recurso de Linguagem de IA do Azure no portal do Azure). O arquivo já deve conter os nomes do projeto e da implantação do seu modelo de classificação de texto.
1. Salve o arquivo de configuração.

## Adicionar código para classificar documentos

Agora você está pronto para usar o serviço de Linguagem de IA do Azure para classificar documentos.

1. Expanda a pasta **artigos** na pasta **classify-text** para exibir os artigos de texto que seu aplicativo classificará.
1. Na pasta **classify-text**, abra o arquivo de código para o aplicativo cliente:

    - **C#**: Program.cs
    - **Python**: classify-text.py

1. Localize o comentário **Importar namespaces**. Neste comentário, adicione o seguinte código específico da linguagem para importar os namespaces necessários para usar o SDK de Análise de Texto:

    **C#**: Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.TextAnalytics;
    ```

    **Python**: classify-text.py

    ```python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.textanalytics import TextAnalyticsClient
    ```

1. Na função **Principal**, observe que o código para carregar o ponto de extremidade e a chave do serviço de Linguagem de IA do Azure e os nomes de projeto e implantação do arquivo de configuração já foram fornecidos. Em seguida, localize o comentário **Criar cliente usando ponto de extremidade e chave** e adicione o seguinte código para criar um cliente para a API de Análise de Texto:

    **C#**: Programs.cs

    ```csharp
    // Create client using endpoint and key
    AzureKeyCredential credentials = new AzureKeyCredential(aiSvcKey);
    Uri endpoint = new Uri(aiSvcEndpoint);
    TextAnalyticsClient aiClient = new TextAnalyticsClient(endpoint, credentials);
    ```

    **Python**: classify-text.py

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(ai_key)
    ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

1. Na função **Principal**, observe que o código existente lê todos os arquivos na pasta **artigos** e cria uma lista contendo seu conteúdo. Em seguida, localize o comentário **Obter classificações** e adicione o seguinte código:

    **C#**: Program.cs

    ```csharp
    // Get Classifications
    ClassifyDocumentOperation operation = await aiClient.SingleLabelClassifyAsync(WaitUntil.Completed, batchedDocuments, projectName, deploymentName);

    int fileNo = 0;
    await foreach (ClassifyDocumentResultCollection documentsInPage in operation.Value)
    {
        
        foreach (ClassifyDocumentResult documentResult in documentsInPage)
        {
            Console.WriteLine(files[fileNo].Name);
            if (documentResult.HasError)
            {
                Console.WriteLine($"  Error!");
                Console.WriteLine($"  Document error code: {documentResult.Error.ErrorCode}");
                Console.WriteLine($"  Message: {documentResult.Error.Message}");
                continue;
            }

            Console.WriteLine($"  Predicted the following class:");
            Console.WriteLine();

            foreach (ClassificationCategory classification in documentResult.ClassificationCategories)
            {
                Console.WriteLine($"  Category: {classification.Category}");
                Console.WriteLine($"  Confidence score: {classification.ConfidenceScore}");
                Console.WriteLine();
            }
            fileNo++;
        }
    }
    ```
    
    **Python**: classify-text.py

    ```Python
    # Get Classifications
    operation = ai_client.begin_single_label_classify(
        batchedDocuments,
        project_name=project_name,
        deployment_name=deployment_name
    )

    document_results = operation.result()

    for doc, classification_result in zip(files, document_results):
        if classification_result.kind == "CustomDocumentClassification":
            classification = classification_result.classifications[0]
            print("{} was classified as '{}' with confidence score {}.".format(
                doc, classification.category, classification.confidence_score)
            )
        elif classification_result.is_error is True:
            print("{} has an error with code '{}' and message '{}'".format(
                doc, classification_result.error.code, classification_result.error.message)
            )
    ```

1. Salve as alterações no arquivo de código.

## Teste seu aplicativo

Agora, seu aplicativo está pronto para teste.

1. No terminal integrado da pasta **classify-text**, digite o seguinte comando para executar o programa:

    - **C#**: `dotnet run`
    - **Python**: `python classify-text.py`

    > **Dica**: você pode usar o ícone **Maximizar tamanho do painel** (**^**) na barra de ferramentas do terminal para ver mais do texto do console.

1. Observe a saída. O aplicativo deve listar uma classificação e pontuação de confiança para cada arquivo de texto.


## Limpar

Quando não precisar mais do projeto, você poderá excluí-lo de sua página de **Projetos** no Language Studio. Você também pode remover o serviço de Linguagem de IA do Azure e a conta de armazenamento associada a ele no [portal do Azure](https://portal.azure.com).
