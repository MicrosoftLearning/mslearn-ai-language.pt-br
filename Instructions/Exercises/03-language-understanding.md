---
lab:
  title: Criar um modelo de reconhecimento de linguagem com o serviço de Linguagem de IA do Azure
  module: Module 5 - Create language understanding solutions
---

# Criar um modelo de reconhecimento de linguagem com serviço de linguagem

> **OBSERVAÇÃO** O recurso de compreensão da linguagem coloquial do serviço de linguagem de IA do Azure está atualmente em visualização e sujeito a alterações. Em alguns casos, o treinamento do modelo pode falhar – se isso acontecer, tente novamente.  

O serviço de Linguagem de IA do Azure permite que você defina um modelo de *compreensão da linguagem coloquial* que os aplicativos podem usar para interpretar a entrada de linguagem natural dos usuários, prever a *intenção* dos usuários (o que eles desejam alcançar) e identificar *entidades* às quais a intenção deve ser aplicada.

Por exemplo, pode-se esperar que um modelo de linguagem coloquial para um aplicativo de relógio processe entradas como:

*Que horas são em Londres?*

Essa variante de entrada é um exemplo de um *enunciado* (algo que um usuário pode dizer ou digitar), para o qual a *intenção* desejada é obter o tempo em um local específico (uma *entidade*), no caso, Londres.

> **OBSERVAÇÃO** A tarefa do modelo de linguagem coloquial é prever a intenção do usuário e identificar as entidades às quais a intenção se aplica. <u>Não</u> é a função de um modelo de linguagem coloquial realmente executar as ações necessárias para satisfazer a intenção. Por exemplo, um aplicativo de relógio pode usar um modelo de linguagem coloquial para discernir que o usuário deseja saber a hora em Londres; o próprio aplicativo cliente, porém, deve implementar a lógica para determinar a hora correta e apresentá-la ao usuário.

## Provisionar um recurso da *Linguagem de IA do Azure*

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
1. Exiba a página **Chaves e Ponto de Extremidade**. Você precisará das informações nesta página mais adiante no exercício.

## Criar um projeto de compreensão da linguagem coloquial

Agora que você criou um recurso de criação, pode usá-lo para criar um projeto de compreensão da linguagem coloquial.

1. Em uma nova guia do navegador, abra o portal do Azure AI Language Studio em `https://language.cognitive.azure.com/` e entre usando a conta Microsoft associada à sua assinatura do Azure.

1. Se for solicitado a escolher um recurso de linguagem, selecione as seguintes configurações:

    - **Azure Directory**: o diretório do Azure contendo a sua assinatura.
    - **Assinatura do Azure**: sua assinatura do Azure.
    - **Tipo de recurso**: linguagem.
    - **Recurso de linguagem**: o recurso de Linguagem de IA do Azure criado anteriormente.

    Se você <u>não</u> foi solicitado a escolher um recurso de linguagem, pode ser porque você tem vários recursos de linguagem em sua assinatura; nesse caso:

    1. Na barra na parte superior da página, selecione o botão **Configurações(&#9881;)**.
    2. Na página **Configurações**, exiba a guia **Recursos**.
    3. Selecione o recurso de linguagem que você acabou de criar e clique em **Alternar recurso**.
    4. Na parte superior da página, clique em **Language Studio** para retornar à página incial do Language Studio

1. Na parte superior do portal do Language Studio, no menu **Criar**, selecione **Compreensão da linguagem coloquial**.

1. Na caixa de diálogo **Criar um projeto**, na página **Inserir informações básicas**, insira os seguintes detalhes e selecione **Avançar**:
    - **Nome**: `Clock`
    - **Idioma primário dos enunciados**: inglês
    - **Habilitar vários idiomas no projeto?**: *Não Selecionado*
    - **Descrição**: `Natural language clock`

1. Na página **Análise e conclusão**, selecione **Criar**.

### Criar intenções

A primeira coisa que faremos no novo projeto é definir algumas intenções. O modelo irá finalmente prever quais dessas intenções um usuário está solicitando ao enviar um enunciado de linguagem natural.

> **Dica**: ao trabalhar em seu projeto, se algumas dicas forem exibidas, leia-as e selecione **Entendi** para descartá-las ou selecione **Ignorar tudo**.

1. Na página **Definição de esquema**, na guia **Intenções**, selecione **&#65291; Adicionar** para adicionar uma nova intenção nomeada `GetTime`.
1. Verifique se a intenção **GetTime** está listada (junto com a intenção padrão **Nenhum**). Em seguida, adicione as seguintes intenções adicionais:
    - `GetDay`
    - `GetDate`

### Rotular cada intenção com enunciados de amostra

Para ajudar o modelo a prever qual intenção um usuário está solicitando, você deve rotular cada intenção com algumas declarações de amostra.

1. No painel à esquerda, selecione a página **Rotulagem de Dados**.

> **Dica**: você pode expandir o painel com o ícone **>>** para ver os nomes das páginas e ocultá-lo novamente com o ícone **<<**.

1. Selecione a nova intenção **GetTime** e insira o enunciado `what is the time?`. Isso adiciona o enunciado como entrada de amostra para a intenção.
1. Adicione as seguintes declarações adicionais para a intenção **GetTime**:
    - `what's the time?`
    - `what time is it?`
    - `tell me the time`

    > **OBSERVAÇÃO** Para adicionar um novo enunciado, escreva o enunciado na caixa de texto ao lado da intenção e pressione ENTER. 

1. Selecione a intenção **GetDay** e adicione as seguintes declarações como entrada de exemplo para essa intenção:
    - `what day is it?`
    - `what's the day?`
    - `what is the day today?`
    - `what day of the week is it?`

1. Selecione a intenção **GetDate** e adicione os seguintes enunciados para ela:
    - `what date is it?`
    - `what's the date?`
    - `what is the date today?`
    - `what's today's date?`

1. Depois de adicionar enunciados para cada uma das suas intenções, selecione **Salvar alterações**.

### Treinar e testar o modelo

Agora que você adicionou algumas intenções, vamos treinar o modelo de linguagem e ver se ele pode predizê-las corretamente a partir da entrada de usuário.

1. No painel à esquerda, selecione **Trabalhos de treinamento**. Então escolha **+ Iniciar um trabalho de treinamento**.

1. Na caixa de diálogo **Iniciar um trabalho de treinamento**, selecione a opção para treinar um novo modelo e dê o nome `Clock`. Selecione o modo de **treinamento padrão** e as opções padrão de **divisão de dados**.

1. Para iniciar o processo de treinamento do seu modelo, selecione **Treinar**.

1. Quando o treinamento for concluído (o que pode levar vários minutos), o **status** do trabalho será alterado para **Treinamento bem-sucedido**.

1. Selecione a página **Desempenho do modelo** e, em seguida, selecione o modelo de **relógio**. Revise as métricas de avaliação global e por intenção (*precisão*, *recall* e *medida f1*) e a *matriz de confusão* geradas pela avaliação que foi realizada durante o treinamento (observe que, devido ao pequeno número de enunciados de amostra, nem todas as intenções podem ser incluídas nos resultados).

    > **OBSERVAÇÃO** Para saber mais sobre as métricas de avaliação, consulte a [documentação](https://learn.microsoft.com/azure/ai-services/language-service/conversational-language-understanding/concepts/evaluation-metrics)

1. Vá para a página **Implantação de um modelo** e selecione **Adicionar implantação**.

1. Na caixa de diálogo **Adicionar implantação**, selecione **Criar um novo nome de implantação** e digite `production`.

1. Selecione o modelo **relógio** no campo **Modelo** e, em seguida, selecione **Implantar**. A implantação pode levar algum tempo.

1. Quando o modelo tiver sido implantado, selecione a página **Teste de implantações** e, em seguida, selecione a implantação de **produção** no campo **Nome da implantação**.

1. No campo de texto vazio, insira o seguinte texto e selecione **Executar o teste**:

    `what's the time now?`

    Analise o resultado retornado, indicando que ele inclui a intenção prevista (que deve ser **GetTime**) e a pontuação de confiança que indica a probabilidade calculada pelo modelo para a intenção e a intenção prevista. A guia JSON mostra um comparativo de confiança para cada intenção potencial (aquela com a pontuação de confiança mais alta é a intenção prevista)

1. Desmarque a caixa de texto e execute outro teste com o seguinte texto:

    `tell me the time`

    Novamente, revise a intenção prevista e a pontuação de confiança.

1. Tente o seguinte texto:

    `what's the day today?`

    Espero que o modelo preveja a intenção **GetDay**.

## Adicionar entidades

Até agora você definiu alguns enunciados simples que mapeiam as intenções. A maioria dos aplicativos reais inclui enunciados mais complexos dos quais entidades de dados específicas devem ser extraídas para obter mais contexto para a intenção.

### Adicione uma entidade aprendida

O tipo mais comum de entidade é uma entidade *aprendida*, na qual o modelo aprende a identificar valores de entidade com base em exemplos.

1. No Language Studio, retorne à página **Definição de esquema** e, na guia **Entidades**, selecione **&#65291; Adicionar** para adicionar uma nova entidade.

1. Na caixa de diálogo **Adicionar uma entidade**, digite o nome da entidade `Location` e verifique se a guia **Aprendido** está selecionada. Selecione **Adicionar entidade**.

1. Depois que a entidade **Local** tiver sido criada, retorne à página **Rotulagem de dados**.
1. Selecione a intenção **GetTime** e insira o seguinte novo exemplo de enunciado:

    `what time is it in London?`

1. Quando o enunciado tiver sido adicionado, selecione a palavra **Londres** e, na lista suspensa exibida, selecione **Local** para indicar que "Londres" é um exemplo de local.

1. Adicione outro exemplo de enunciado para a intenção **GetTime**:

    `Tell me the time in Paris?`

1. Quando o enunciado tiver sido adicionado, selecione a palavra **Paris** e mapeie-a para a entidade **Local**.

1. Adicione outro exemplo de enunciado para a intenção **GetTime**:

    `what's the time in New York?`

1. Quando o enunciado tiver sido adicionado, selecione as palavras **Nova York** e mapeie-as para a entidade **Local**.

1. Selecione **Salvar alterações** para salvar os novos enunciados.

### Adicionar uma entidade de *lista*

Em alguns casos, os valores válidos para uma entidade podem ser restritos a uma lista de termos e sinônimos específicos; o que pode ajudar o aplicativo a identificar instâncias da entidade em enunciados.

1. No Language Studio, retorne à página **Definição de esquema** e, na guia **Entidades**, selecione **&#65291; Adicionar** para adicionar uma nova entidade.

1. Na caixa de diálogo **Adicionar uma entidade**, digite o nome da entidade `Weekday` e selecione a guia entidade de **lista**. Em seguida, selecione **Adicionar entidade**.

1. Na página da entidade para **dia da semana**, na seção **Aprendido**, verifique se a opção **Não necessário** está selecionada. Em seguida, na seção **Lista**, selecione **&#65291; Adicionar nova lista**. Insira os valores e sinônimos a seguir e selecione **Salvar**:

    | Listar chave | sinônimos|
    |-------------------|---------|
    | `Sunday` | `Sun` |

    > **OBSERVAÇÃO** Para inserir os campos da nova lista, insira o valor `Sunday` no campo de texto e clique no campo onde "Digite o valor e pressione enter..." é exibido, digite os sinônimos e pressione ENTER.

1. Repita a etapa anterior para adicionar a seguinte lista de componentes:

    | Valor | sinônimos|
    |-------------------|---------|
    | `Monday` | `Mon` |
    | `Tuesday` | `Tue, Tues` |
    | `Wednesday` | `Wed, Weds` |
    | `Thursday` | `Thur, Thurs` |
    | `Friday` | `Fri` |
    | `Saturday` | `Sat` |

1. Depois de adicionar e salvar os valores da lista, retorne à página de **Rotulagem de dados**.
1. Selecione a intenção **GetDate** e insira o seguinte novo exemplo de enunciado:

    `what date was it on Saturday?`

1. Quando o enunciado tiver sido adicionado, selecione a palavra ***Sábado*** e, na lista suspensa exibida, selecione **Dia da semana**.

1. Adicione outro exemplo de enunciado para a intenção **GetDate**:

    `what date will it be on Friday?`

1. Quando o enunciado tiver sido adicionado, mapeie **sexta-feira** para a entidade **Dia da semana**.

1. Adicione outro exemplo de enunciado para a intenção **GetDate**:

    `what will the date be on Thurs?`

1. Quando o enunciado tiver sido adicionado, mapeie **Quinta** para a entidade **Dia da Semana**.

1. selecione **Salvar alterações** para salvar os novos enunciados.

### Adicionar uma entidade *predefinida*

O serviço de Linguagem de IA do Azure fornece um conjunto de entidades *prédefinidas* que são comumente usadas em aplicativos de conversação.

1. No Language Studio, retorne à página **Definição de esquema** e, na guia **Entidades**, selecione **&#65291; Adicionar** para adicionar uma nova entidade.

1. Na caixa de diálogo **Adicionar uma entidade**, digite o nome da entidade `Date` e selecione a guia de entidade **prédefinida**. Em seguida, selecione **Adicionar entidade**.

1. Na página da entidade **Data**, na seção **Aprendido**, verifique se **Não obrigatório** está selecionada. Em seguida, na seção **predefinido**, selecione **&#65291; Adicionar novo predefinido**.

1. Na lista **Selecionar predefinido**, selecione **DateTime** e, em seguida, selecione **Salvar**.
1. Depois de adicionar a entidade predefinida, retorne à página **Rotulagem de dados**
1. Selecione a intenção **GetDay** e insira o seguinte novo exemplo de enunciado:

    `what day was 01/01/1901?`

1. Quando o enunciado tiver sido adicionado, selecione ***01/01/1901*** e, na lista suspensa exibida, selecione **Data**.

1. Adicione outro exemplo de enunciado para a intenção **GetDay**:

    `what day will it be on Dec 31st 2099?`

1. Quando o enunciado tiver sido adicionado, mapeie **31 de dezembro de 2099** para a entidade **Data**.

1. Selecione **Salvar alterações** para salvar os novos enunciados.

### Treinar o modelo novamente

Agora que você modificou o esquema, precisará treinar novamente e testar novamente o modelo.

1. Na página **Trabalhos de treinamento**, selecione **Iniciar um trabalho de treinamento**.

1. Na caixa de diálogo **Iniciar um trabalho de treinamento**, selecione **substituir um modelo existente** e especifique o modelo **Relógio**. Selecione **Treinar** para treinar o modelo. Se solicitado, confirme que deseja substituir o modelo existente.

1. Quando o treinamento for concluído, o **Status** do trabalho será atualizado para **Treinamento bem-sucedido**.

1. Selecione a página **desempenho do modelo** e, em seguida, selecione o modelo **relógio**. Revise as métricas de avaliação (*precisão*, *recall* e *medida F1*) e a *matriz de confusão* geradas pela avaliação realizada durante o treinamento (observe que, devido ao pequeno número de amostras de enunciados, nem todas as intenções podem ser incluídas nos resultados).

1. Na página **Implantação de um modelo**, selecione **Adicionar implantação**.

1. Na caixa de diálogo **Adicionar implantação**, selecione **Substituir um nome de implantação existente** e selecione **produção**.

1. Selecione o modelo **Relógio** no campo **Modelo** e, em seguida, selecione **Implantar** para implantá-lo. Isso pode levar algum tempo.

1. Quando o modelo for implantado, na página **Teste de implantações**, selecione a implantação de **produção** no campo **Nome da implantação** e teste-a com o seguinte texto:

    `what's the time in Edinburgh?`

1. Revise o resultado retornado, que deve prever a intenção **GetTime** e uma entidade de **Localização** com o valor de texto "Edinburgh".

1. Tente testar os seguintes enunciados:

    `what time is it in Tokyo?`

    `what date is it on Friday?`

    `what's the date on Weds?`

    `what day was 01/01/2020?`

    `what day will Mar 7th 2030 be?`

## Usar o modelo de um aplicativo cliente

Em um projeto real, você refinaria iterativamente intenções e entidades, treinaria novamente e testaria novamente até ficar satisfeito com o desempenho preditivo. Em seguida, quando você o testar e estiver satisfeito com seu desempenho preditivo, poderá usá-lo em um aplicativo cliente chamando sua interface REST ou um SDK específico de tempo de execução.

### Prepare-se para desenvolver um aplicativo no Visual Studio Code

Você desenvolverá seu aplicativo de reconhecimento de linguagem usando o Visual Studio Code. Os arquivos de código para seu aplicativo foram fornecidos em um repositório do GitHub.

> **Dica**: se você já clonou o repositório **mslearn-ai-language**, abra-o no Visual Studio Code. Caso contrário, siga estas etapas para cloná-lo em seu ambiente de desenvolvimento.

1. Inicie o Visual Studio Code.
2. Abra a paleta (SHIFT+CTRL+P) e execute o comando **Git: Clone** para clonar o repositório `https://github.com/MicrosoftLearning/mslearn-ai-language` em uma pasta local (não importa qual pasta).
3. Depois que o repositório for clonado, abra a pasta no Visual Studio Code.

    > **Observação**: Se o Visual Studio Code mostrar uma mensagem pop-up para solicitar que você confie no código que está abrindo, clique na opção **Sim, confio nos autores** no pop-up.

4. Aguarde enquanto os arquivos adicionais são instalados para dar suporte aos projetos de código C# no repositório.

    > **Observação**: se você for solicitado a adicionar os ativos necessários para compilar e depurar, selecione **Agora não**.

### Configurar seu aplicativo

Aplicativos para C# e Python foram fornecidos, bem como um arquivo de texto de exemplo que você usará para testar o resumo. Ambos os aplicativos apresentam a mesma funcionalidade. Primeiro, você concluirá algumas partes importantes do aplicativo para habilitar o uso do recurso Linguagem de IA do Azure.

1. No Visual Studio Code, no painel **Explorer**, navegue até a pasta **Labfiles/03-language** e expanda a pasta **CSharp** ou **Python** dependendo da sua linguagem de preferência e da pasta **clock-client** que ela contém. Cada pasta contém os arquivos específicos de linguagem de um aplicativo no qual você integrará à funcionalidade de respostas às perguntas de Linguagem de IA do Azure.
2. Clique com o botão direito do mouse na pasta **clock-client** que contém os arquivos de código e abra um terminal integrado. Em seguida, instale o pacote SDK de compreensão da linguagem coloquial da Linguagem de IA do Azure executando o comando apropriado para sua preferência de linguagem:

    **C#**:

    ```
    dotnet add package Azure.AI.Language.Conversations --version 1.1.0
    ```

    **Python**:

    ```
    pip install azure-ai-language-conversations
    ```

3. No painel **Explorer**, na pasta **clock-client**, abra o arquivo de configuração da linguagem de sua preferência

    - **C#**: appsettings.json
    - **Python**: .env
    
4. Atualize os valores de configuração para incluir o  **ponto de extremidade** e uma **chave** do recurso de linguagem do Azure que você criou (disponível na página **Chaves e Ponto de Extremidade** para seu recurso de Linguagem de IA do Azure no portal do Azure).
5. Salve o arquivo de configuração.

### Adicionar código ao aplicativo

Agora você está pronto para adicionar o código necessário para importar as bibliotecas SDK necessárias, estabelecer uma conexão autenticada com seu projeto implantado e enviar perguntas.

1. Observe que a pasta **clock-client** contém um arquivo de código para o aplicativo cliente:

    - **C#**: Program.cs
    - **Python**: clock-client.py

    Abra o arquivo de código e, na parte superior, sob as referências de namespace existentes, localize o comentário **Importar namespaces**. Neste comentário, adicione o seguinte código específico do idioma para importar os namespaces necessários para usar o SDK de Análise de Texto:

    **C#**: Programs.cs

    ```c#
    // import namespaces
    using Azure;
    using Azure.AI.Language.Conversations;
    ```

    **Python**: clock-client.py

    ```python
    # Import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.language.conversations import ConversationAnalysisClient
    ```

1. Na função **Principal**, observe que o código para carregar o ponto de extremidade de previsão e a chave do arquivo de configuração já foi fornecido. Em seguida, localize o comentário **Criar um cliente para o modelo de serviço de linguagem** e adicione o seguinte código para criar um cliente de previsão para seu aplicativo de serviço de linguagem:

    **C#**: Programs.cs

    ```c#
    // Create a client for the Language service model
    Uri endpoint = new Uri(predictionEndpoint);
    AzureKeyCredential credential = new AzureKeyCredential(predictionKey);

    ConversationAnalysisClient client = new ConversationAnalysisClient(endpoint, credential);
    ```

    **Python**: clock-client.py

    ```python
    # Create a client for the Language service model
    client = ConversationAnalysisClient(
        ls_prediction_endpoint, AzureKeyCredential(ls_prediction_key))
    ```

1. Observe que o código na função **Principal** solicita a entrada de usuário até que o usuário digite "quit". Dentro desse loop, localize o comentário **Chamar o modelo de serviço de idioma para obter intenção e entidades** e adicione o seguinte código:

    **C#**: Programs.cs

    ```c#
    // Call the Language service model to get intent and entities
    var projectName = "Clock";
    var deploymentName = "production";
    var data = new
    {
        analysisInput = new
        {
            conversationItem = new
            {
                text = userText,
                id = "1",
                participantId = "1",
            }
        },
        parameters = new
        {
            projectName,
            deploymentName,
            // Use Utf16CodeUnit for strings in .NET.
            stringIndexType = "Utf16CodeUnit",
        },
        kind = "Conversation",
    };
    // Send request
    Response response = await client.AnalyzeConversationAsync(RequestContent.Create(data));
    dynamic conversationalTaskResult = response.Content.ToDynamicFromJson(JsonPropertyNames.CamelCase);
    dynamic conversationPrediction = conversationalTaskResult.Result.Prediction;   
    var options = new JsonSerializerOptions { WriteIndented = true };
    Console.WriteLine(JsonSerializer.Serialize(conversationalTaskResult, options));
    Console.WriteLine("--------------------\n");
    Console.WriteLine(userText);
    var topIntent = "";
    if (conversationPrediction.Intents[0].ConfidenceScore > 0.5)
    {
        topIntent = conversationPrediction.TopIntent;
    }
    ```

    **Python**: clock-client.py

    ```python
    # Call the Language service model to get intent and entities
    cls_project = 'Clock'
    deployment_slot = 'production'

    with client:
        query = userText
        result = client.analyze_conversation(
            task={
                "kind": "Conversation",
                "analysisInput": {
                    "conversationItem": {
                        "participantId": "1",
                        "id": "1",
                        "modality": "text",
                        "language": "en",
                        "text": query
                    },
                    "isLoggingEnabled": False
                },
                "parameters": {
                    "projectName": cls_project,
                    "deploymentName": deployment_slot,
                    "verbose": True
                }
            }
        )

    top_intent = result["result"]["prediction"]["topIntent"]
    entities = result["result"]["prediction"]["entities"]

    print("view top intent:")
    print("\ttop intent: {}".format(result["result"]["prediction"]["topIntent"]))
    print("\tcategory: {}".format(result["result"]["prediction"]["intents"][0]["category"]))
    print("\tconfidence score: {}\n".format(result["result"]["prediction"]["intents"][0]["confidenceScore"]))

    print("view entities:")
    for entity in entities:
        print("\tcategory: {}".format(entity["category"]))
        print("\ttext: {}".format(entity["text"]))
        print("\tconfidence score: {}".format(entity["confidenceScore"]))

    print("query: {}".format(result["result"]["query"]))
    ```

    A chamada para o modelo de serviço de idioma retorna uma previsão/resultado, que inclui a intenção superior (mais provável), bem como entidades que foram detectadas no enunciado de entrada. Seu aplicativo cliente agora deve usar essa previsão para determinar e executar a ação apropriada.

1. Localize o comentário **Aplique a ação apropriada** e adicione o código a seguir, que verifica as intenções suportadas pelo aplicativo (**GetTime**, **GetDate** e **GetDay**) e determina se alguma entidade relevante foi detectada, antes de chamar uma função existente para produzir uma resposta apropriada.

    **C#**: Programs.cs

    ```c#
    // Apply the appropriate action
    switch (topIntent)
    {
        case "GetTime":
            var location = "local";           
            // Check for a location entity
            foreach (dynamic entity in conversationPrediction.Entities)
            {
                if (entity.Category == "Location")
                {
                    //Console.WriteLine($"Location Confidence: {entity.ConfidenceScore}");
                    location = entity.Text;
                }
            }
            // Get the time for the specified location
            string timeResponse = GetTime(location);
            Console.WriteLine(timeResponse);
            break;
        case "GetDay":
            var date = DateTime.Today.ToShortDateString();            
            // Check for a Date entity
            foreach (dynamic entity in conversationPrediction.Entities)
            {
                if (entity.Category == "Date")
                {
                    //Console.WriteLine($"Location Confidence: {entity.ConfidenceScore}");
                    date = entity.Text;
                }
            }            
            // Get the day for the specified date
            string dayResponse = GetDay(date);
            Console.WriteLine(dayResponse);
            break;
        case "GetDate":
            var day = DateTime.Today.DayOfWeek.ToString();
            // Check for entities            
            // Check for a Weekday entity
            foreach (dynamic entity in conversationPrediction.Entities)
            {
                if (entity.Category == "Weekday")
                {
                    //Console.WriteLine($"Location Confidence: {entity.ConfidenceScore}");
                    day = entity.Text;
                }
            }          
            // Get the date for the specified day
            string dateResponse = GetDate(day);
            Console.WriteLine(dateResponse);
            break;
        default:
            // Some other intent (for example, "None") was predicted
            Console.WriteLine("Try asking me for the time, the day, or the date.");
            break;
    }
    ```

    **Python**: clock-client.py

    ```python
    # Apply the appropriate action
    if top_intent == 'GetTime':
        location = 'local'
        # Check for entities
        if len(entities) > 0:
            # Check for a location entity
            for entity in entities:
                if 'Location' == entity["category"]:
                    # ML entities are strings, get the first one
                    location = entity["text"]
        # Get the time for the specified location
        print(GetTime(location))

    elif top_intent == 'GetDay':
        date_string = date.today().strftime("%m/%d/%Y")
        # Check for entities
        if len(entities) > 0:
            # Check for a Date entity
            for entity in entities:
                if 'Date' == entity["category"]:
                    # Regex entities are strings, get the first one
                    date_string = entity["text"]
        # Get the day for the specified date
        print(GetDay(date_string))

    elif top_intent == 'GetDate':
        day = 'today'
        # Check for entities
        if len(entities) > 0:
            # Check for a Weekday entity
            for entity in entities:
                if 'Weekday' == entity["category"]:
                # List entities are lists
                    day = entity["text"]
        # Get the date for the specified day
        print(GetDate(day))

    else:
        # Some other intent (for example, "None") was predicted
        print('Try asking me for the time, the day, or the date.')
    ```

1. Salve suas alterações e retorne ao terminal integrado para a pasta **clock-client** e digite o seguinte comando para executar o programa:

    - **C#**: `dotnet run`
    - **Python**: `python clock-client.py`

    > **Dica**: você pode usar o ícone **Maximizar o tamanho do painel** (**^**) na barra de ferramentas do terminal para ver mais do texto do console.

1. Quando solicitado, insira os enunciados para testar o aplicativo. Por exemplo, tente:

    *Olá*

    *Que horas são?*

    *Que horas são em Londres?*

    *Qual é a data?*

    *Qual é a data de domingo?*

    *Que dia é esse?*

    *Que dia é 01/01/2025?*

    > **Observação**: a lógica no aplicativo é deliberadamente simples e tem uma série de limitações. Por exemplo, ao obter o tempo, apenas um conjunto restrito de cidades é suportado e o horário de verão é ignorado. O objetivo é ver um exemplo de um típico padrão para usar o serviço de linguagem no qual seu aplicativo deve:
    >   1. Conectar-se a um ponto de extremidade de previsão.
    >   2. Enviar um enunciado para obter uma previsão.
    >   3. Implementar lógica para responder adequadamente à intenção e entidades previstas.

1. Quando terminar o teste, digite *quit.*.

## Limpar os recursos

Dica: se você tiver concluído a exploração do serviço de Linguagem de IA do Azure, exclua os recursos criados no exercício. Este é o procedimento:

1. Abra o portal do Azure em `https://portal.azure.com` e entre usando a conta Microsoft associada à sua assinatura do Azure.
2. Navegue até o recurso de Linguagem de IA do Azure que você criou neste laboratório.
3. Na página do recurso, selecione **Excluir** e siga as instruções para excluir o recurso.

## Mais informações

Para saber mais sobre a compreensão da linguagem coloquial na Linguagem de IA do Azure, confira a [documentação da Linguagem de IA do Azure](https://learn.microsoft.com/azure/ai-services/language-service/conversational-language-understanding/overview).
