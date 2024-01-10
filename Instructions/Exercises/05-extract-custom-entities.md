---
lab:
  title: Extrair entidades personalizadas
  module: Module 3 - Getting Started with Natural Language Processing
---

# Extrair entidades personalizadas

Além de outras funcionalidades de processamento de linguagem natural, o serviço de Linguagem de IA do Azure permite que você defina entidades personalizadas e extraia instâncias delas a partir de textos.

Para testar a extração de entidade personalizada, criaremos um modelo e o treinaremos por meio do Estúdio da Linguagem de IA do Azure, então usaremos um aplicativo de linha de comando para testá-lo.

## Provisionar um recurso da *Linguagem de IA do Azure*

Caso ainda não tenha um na sua assinatura, provisione um recurso de **serviço de Linguagem de IA do Azure**. Além disso, use a classificação de textos personalizada, você precisa habilitar o recurso **classificação de textos personalizada e extração**.

1. Em um navegador, abra o portal do Azure em `https://portal.azure.com` e entre com sua conta Microsoft.
1. Selecione o botão **Criar um recurso**, procure por *Linguagem* e crie um recurso do **Serviço de Linguagem de IA do Azure**. Quando perguntado sobre *recursos adicionais*, selecione **Classificação de textos personalizada e extração**. Crie o recurso com as seguintes configurações:
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *selecionar ou criar um grupo de recursos*
    - **Região**: *escolha uma região disponível*
    - **Nome**: *insira um nome exclusivo*
    - **Tipo de preço**: selecione **F0** (*gratuito*), ou **S** (*Standard*) se F não estiver disponível.
    - **Conta de armazenamento**: nova conta de armazenamento:
      - **Nome da conta de armazenamento**: *insira um nome exclusivo*.
      - **Tipo de conta de armazenamento**: LRS Standard
    - **Aviso de IA Responsável**: selecionado.

1. Selecione **Revisar + criar** e, em seguida, selecione **Criar** para provisionar o recurso.
1. Aguarde a conclusão da implantação e acesse o recurso implantado.
1. Vizualize a página **Chaves e Ponto de Extremidade**. Você precisará das informações nesta página mais adiante no exercício.

## Carregar anúncios de exemplo

Depois de criar o serviço de linguagem e a conta de armazenamento, você precisará carregar anúncios de exemplo para treinar seu modelo mais tarde.

1. Em uma nova guia do navegador, baixe amostras de anúncios classificados de `https://aka.ms/entity-extraction-ads` e extraia os arquivos para uma pasta de sua escolha.

2. No portal do Azure, navegue até a conta de armazenamento que você criou e selecione-a.

3. Na sua conta de armazenamento, selecione **Configuração**, localizada abaixo de **Configurações**, e ative a opção **Permitir acesso anônimo de blob** e selecione **Salvar**.

4. Selecione **Contêineres** no menu esquerdo localizado abaixo de **Armazenamento de dados**. Na tela exibida, selecione **+ Contêiner**. Dê ao contêiner o nome `classifieds` e defina **Nível de acesso anônimo** ao **Contêiner (acesso de leitura anônimo para contêineres e blobs)**.

    > **OBSERVAÇÃO**: ao configurar uma conta de armazenamento para uma solução real, tenha cuidado para atribuir o nível de acesso apropriado. Para saber mais sobre cada nível de acesso, confira a [documentação de armazenamento do Azure](https://learn.microsoft.com/azure/storage/blobs/anonymous-read-access-configure).

5. Depois de criar o contêiner, selecione-o e clique no botão **Carregar** e carregue os anúncios de exemplo baixados.

## Crie um projeto de reconhecimento de entidade nomeada personalizado

Agora você está pronto para criar um projeto de reconhecimento de entidade nomeada personalizada. Esse projeto proporciona um local de trabalho para criar, treinar e implantar o modelo.

> **OBSERVAÇÃO**: você também pode criar, compilar, treinar e implantar seu modelo por meio da API REST.

1. Em uma nova guia do navegador, abra o portal do Azure AI Language Studio em `https://language.cognitive.azure.com/` e entre usando a conta Microsoft associada à sua assinatura do Azure.
1. Se for solicitado a escolher um recurso de idioma, selecione as seguintes configurações:

    - **Azure Directory**: o diretório do Azure contendo a sua assinatura.
    - **Assinatura do Azure**: sua assinatura do Azure.
    - **Tipo de recurso**: linguagem.
    - **Recurso de linguagem**: o recurso de Linguagem de IA do Azure criado anteriormente.

    Se você <u>não</u> foi solicitado a escolher um recurso de idioma, pode ser porque você tem vários recursos de idioma em sua assinatura; nesse caso:

    1. Na barra na parte superior da página, selecione o botão **Configurações (&#9881;)**.
    2. Na página **Configurações**, exiba a guia **Recursos**.
    3. Selecione o recurso de idioma que você acabou de criar e clique em **Alternar recurso**.
    4. Na parte superior da página, clique em **Language Studio** para retornar à página incial do Language Studio

1. Na parte superior do portal, no menu **Criar novo**, selecione *Reconhecimento de entidade nomeada personalizada**.

1. Criar um novo projeto com as seguintes configurações:
    - **Conectar armazenamento**: *esse valor provavelmente já está preenchido. Mude para a sua conta de armazenamento se ainda não tiver feito isso*
    - **Informações Básicas:**
    - **Nome**: `CustomEntityLab`
        - **Idioma primário do texto**: inglês (EUA)
        - **Seu conjunto de dados inclui documentos que não estão no mesmo idioma?** : *Não*
        - **Descrição**: `Custom entities in classified ads`
    - **Contêiner:**
        - **Contêiner de armazenamento de blobs**: classificados
        - **Seus arquivos são rotulados com classes?**: não, preciso rotular meus arquivos como parte deste projeto

## Rotular seus dados.

Agora que seu projeto foi criado, você precisa rotular os dados a fim de treinar o modelo para identificar entidades.

1. Se a página **Rotulagem de dados** ainda não estiver aberta, no painel à esquerda, selecione **Rotulagem de dados**. Você verá uma lista dos arquivos que carregou na conta de armazenamento.
1. No lado direito, no painel **Activity**, selecione **Adicionar entidade** e adicione uma entidade nomeada `ItemForSale`.
1.  Repita a etapa anterior para criar as entidades a seguir:
    - `Price`
    - `Location`
1. Depois de criar suas três entidades, selecione **Ad 1.txt** para que você possa lê-lo.
1. No *ad 1.txt*: 
    1. Realce o texto *face cord of firewood* e selecione a entidade **ItemForSale**.
    1. Realce o texto *Denver, CO* e selecione a entidade **Local**.
    1. Realce o texto *$90* e selecione a entidade **Preço**.
1. No painel **Atividade**, observe que este documento será adicionado ao conjunto de dados para treinar o modelo.
1. Use o botão **Próximo documento** para mover para o próximo documento e continuar atribuindo texto às entidades apropriadas para todo o conjunto de documentos, adicionando-os todos ao conjunto de dados de treinamento.
1. Depois de rotular o último documento (*Ad 9.txt*), salve os rótulos.

## Treinar seu modelo

Depois de rotular os dados, você precisa treinar o modelo.

1. Selecione **Trabalhos de treinamento** no painel à esquerda.
2. Escolha **Iniciar um trabalho de treinamento**
3. Selecione a opção para obter um novo modelo chamado `ExtractAds`
4. Escolha **Dividir automaticamente o conjunto de testes dos dados de treinamento**

    > **DICA**: em seus projetos de extração, use a divisão de teste mais adequada aos seus dados. Para dados mais consistentes e conjuntos de dados maiores, o Serviço de Linguagem de IA do Azure dividirá automaticamente o conjunto de testes por percentual. Com conjuntos de dados menores, é importante treinar com a variedade certa de documentos de entrada possíveis.

5. Clique em **Treinar**

    > **IMPORTANTE**: treinar seu modelo pode levar vários minutos. Você receberá uma notificação quando ele for concluído.

## Avaliar o modelo

Em aplicativos do mundo real, é importante avaliar e melhorar seu modelo para verificar se ele está funcionando conforme o esperado. Duas páginas à esquerda mostram os detalhes do modelo treinado, bem como os testes que falharam.

Selecione **Desempenho do modelo** no menu do lado esquerdo e selecione seu modelo `ExtractAds`. Lá, você pode ver a pontuação do modelo, as métricas de desempenho e quando ele foi treinado. Você poderá ver se algum documento de teste falhou, e essas falhas ajudarão você a entender como melhorar.

## Implantar o seu modelo

Quando você estiver satisfeito com o treinamento do seu modelo, implante-o para começar a extrair entidades por meio da API.

1. No painel esquerdo, selecione **Implantação de um modelo**.
2. Selecione **Adicionar implantação**, insira o nome `AdEntities` e selecione o modelo **ExtractAds**.
3. Clique em **Implantar** para implantar o modelo.

## Prepare-se para desenvolver um aplicativo no Visual Studio Code

Para testar os recursos de extração de entidade personalizada do serviço de Linguagem de IA do Azure, você desenvolverá um aplicativo de console simples no Visual Studio Code.

> **Dica**: se você já clonou o repositório **mslearn-ai-language**, abra-o no Visual Studio Code. Caso contrário, siga estas etapas para cloná-lo em seu ambiente de desenvolvimento.

1. Inicie o Visual Studio Code.
2. Abra a paleta (SHIFT+CTRL+P) e execute um comando **Git: Clone** para clonar o repositório `https://github.com/MicrosoftLearning/mslearn-ai-language` para uma pasta local (não importa a pasta).
3. Depois que o repositório for clonado, abra a pasta no Visual Studio Code.
4. Aguarde enquanto arquivos adicionais são instalados para que haja suporte aos projetos com o código C# no repositório.

    > **Observação**: se você for solicitado a adicionar os ativos necessários para compilar e depurar, selecione **Agora não**.

## Configurar seu aplicativo

Aplicativos para C# e Python foram fornecidos, bem como um arquivo de texto de exemplo que você usará para testar o resumo. Ambos os aplicativos apresentam a mesma funcionalidade. Primeiro, você concluirá algumas partes importantes do aplicativo para habilitar o uso do recurso Linguagem de IA do Azure.

1. No Visual Studio Code, no painel **Explorer**, navegue até a pasta **Labfiles/05-custom-entity-recognition** e expanda a pasta **CSharp** ou **Python** dependendo de sua preferência de linguagem e a pasta de **entidades personalizadas** que ela contém. Cada pasta contém os arquivos específicos de linguagem do aplicativo no qual você integrará a funcionalidade de classificação de textos de Linguagem de IA do Azure.
1. Clique com o botão direito do mouse na pasta **custom-entities** que contém seus arquivos de código e abra um terminal integrado. Em seguida, instale o pacote do SDK de análise de texto da Linguagem de IA do Azure executando o comando apropriado para sua preferência de idioma:

    **C#**:

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.3.0
    ```

    **Python**:

    ```
    pip install azure-ai-textanalytics==5.3.0
    ```

1. No painel **Explorer**, na pasta **custom-entities**, abra o arquivo de configuração da linguagem de sua preferência

    - **C#**: appsettings.json
    - **Python**: .env
    
1. Atualize os valores de configuração para incluir o  **ponto de extremidade** e uma **chave** do recurso de linguagem do Azure que você criou (disponível na página **Chaves e Ponto de Extremidade** para seu recurso de Linguagem de IA do Azure no portal do Azure). O arquivo já deve conter os nomes de projeto e implantação para seu modelo de extração de entidade personalizada.
1. Salve o arquivo de configuração.

## Adicionar código para extrair entidades

Agora você está pronto para usar o serviço de Linguagem de IA do Azure para extrair entidades personalizadas do texto.

1. Expanda a pasta **ads** na pasta **custom-entities** para visualizar os anúncios classificados que seu aplicativo analisará.
1. Na pasta **custom-entities**, abra o arquivo de código para o aplicativo cliente:

    - **C#**: Program.cs
    - **Python**: custom-entities.py

1. Localize o comentário **Importar namespaces**. Em seguida, neste comentário, adicione o seguinte código específico de linguagem para importar os namespaces necessários para usar o SDK da Análise de Texo:

    **C#**: Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.TextAnalytics;
    ```

    **Python**: custom-entities.py

    ```python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.textanalytics import TextAnalyticsClient
    ```

1. Na função **Principal**, observe que o código para carregar o ponto de extremidade e a chave do serviço de Linguagem de IA do Azure e os nomes de projeto e implantação do arquivo de configuração já foram fornecidos. Em seguida, localize o comentário **Criar cliente usando ponto de extremidade e chave** e adicione o seguinte código para criar um cliente para a API de Análise de Texto:

    **C#**: Programs.cs

    ```csharp
    // Create client using endpoint and key
    AzureKeyCredential credentials = new(aiSvcKey);
    Uri endpoint = new(aiSvcEndpoint);
    TextAnalyticsClient aiClient = new(endpoint, credentials);
    ```

    **Python**: custom-entities.py

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(ai_key)
    ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

1. Na função **Principal**, observe que o código existente lê todos os arquivos na pasta **artigos** e cria uma lista contendo seu conteúdo. No caso do código C#, uma lista de objetos **TextDocumentInput** é usada para incluir o nome do arquivo como uma ID e o idioma. Em Python uma lista simples do conteúdo do texto é usada.
1. Localize o comentário **Extrair entidades** e adicione o seguinte código:

    **C#**: Program.cs

    ```csharp
    // Extract entities
    RecognizeCustomEntitiesOperation operation = await aiClient.RecognizeCustomEntitiesAsync(WaitUntil.Completed, batchedDocuments, projectName, deploymentName);

    await foreach (RecognizeCustomEntitiesResultCollection documentsInPage in operation.Value)
    {
        foreach (RecognizeEntitiesResult documentResult in documentsInPage)
        {
            Console.WriteLine($"Result for \"{documentResult.Id}\":");

            if (documentResult.HasError)
            {
                Console.WriteLine($"  Error!");
                Console.WriteLine($"  Document error code: {documentResult.Error.ErrorCode}");
                Console.WriteLine($"  Message: {documentResult.Error.Message}");
                Console.WriteLine();
                continue;
            }

            Console.WriteLine($"  Recognized {documentResult.Entities.Count} entities:");

            foreach (CategorizedEntity entity in documentResult.Entities)
            {
                Console.WriteLine($"  Entity: {entity.Text}");
                Console.WriteLine($"  Category: {entity.Category}");
                Console.WriteLine($"  Offset: {entity.Offset}");
                Console.WriteLine($"  Length: {entity.Length}");
                Console.WriteLine($"  ConfidenceScore: {entity.ConfidenceScore}");
                Console.WriteLine($"  SubCategory: {entity.SubCategory}");
                Console.WriteLine();
            }

            Console.WriteLine();
        }
    }
    ```

    **Python**: custom-entities.py

    ```Python
    # Extract entities
    operation = ai_client.begin_recognize_custom_entities(
        batchedDocuments,
        project_name=project_name,
        deployment_name=deployment_name
    )

    document_results = operation.result()

    for doc, custom_entities_result in zip(files, document_results):
        print(doc)
        if custom_entities_result.kind == "CustomEntityRecognition":
            for entity in custom_entities_result.entities:
                print(
                    "\tEntity '{}' has category '{}' with confidence score of '{}'".format(
                        entity.text, entity.category, entity.confidence_score
                    )
                )
        elif custom_entities_result.is_error is True:
            print("\tError with code '{}' and message '{}'".format(
                custom_entities_result.error.code, custom_entities_result.error.message
                )
            )
    ```

1. Salve as alterações no arquivo de código.

## Teste seu aplicativo

Agora, seu aplicativo está pronto para teste.

1. No terminal integrado para a pasta **classify-text** e digite o seguinte comando para executar o programa:

    - **C#** : `dotnet run`
    - **Python**: `python custom-entities.py`

    > **Dica**: você pode usar o ícone **Maximizar o tamanho do painel** (**^**) na barra de ferramentas do terminal para ver mais do texto do console.

1. Observe a saída. O aplicativo deve listar detalhes das entidades encontradas em cada arquivo de texto.

## Limpar

Quando não precisar mais do projeto, você poderá excluí-lo de sua página de **Projetos** no Language Studio. Você também pode remover o serviço de Linguagem de IA do Azure e a conta de armazenamento associada a ele no [portal do Azure](https://portal.azure.com).
