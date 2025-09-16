---
lab:
  title: Extrair entidades personalizadas
  description: Treine um modelo para extrair entidades personalizadas da entrada de texto usando a Linguagem de IA do Azure.
---

# Extrair entidades personalizadas

Além de outras funcionalidades de processamento de linguagem natural, o serviço de Linguagem de IA do Azure permite que você defina entidades personalizadas e extraia instâncias destas no texto.

Para testar a extração de entidade personalizada, criaremos um modelo e o treinaremos por meio do Estúdio da Linguagem de IA do Azure, então usaremos um aplicativo Python para testá-lo.

Embora este exercício seja baseado em Python, você pode desenvolver aplicativos de classificação de textos usando vários SDKs específicos da linguagem, incluindo:

- [Biblioteca de clientes de Análise de Texto de IA do Azure para Python](https://pypi.org/project/azure-ai-textanalytics/)
- [Biblioteca de clientes de Análise de Texto de IA do Azure para .NET](https://www.nuget.org/packages/Azure.AI.TextAnalytics)
- [Biblioteca de clientes de Análise de Texto de IA do Azure para JavaScript](https://www.npmjs.com/package/@azure/ai-text-analytics)

Este exercício levará aproximadamente **35** minutos.

## Provisionar um recurso da *Linguagem de IA do Azure*

Caso ainda não tenha um na sua assinatura, provisione um recurso do **serviço de Linguagem de IA do Azure**. Além disso, use a classificação de textos personalizada; você precisa habilitar o **recurso de extração e classificação de textos personalizada**.

1. Em um navegador, abra o Portal do Azure em `https://portal.azure.com` e entre com sua conta Microsoft.
1. Selecione o botão **Criar um recurso**, procure por *Idioma* e crie um recurso do **Serviço de linguagem**. Quando estiver na página para *Selecionar recursos adicionais*, selecione o recurso personalizado que contém a **Extração de reconhecimento de entidade nomeada personalizada**. Crie o recurso com as seguintes configurações:
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *selecione ou crie um grupo de recursos*.
    - **Região**: *escolha uma das seguintes regiões*\*
        - Leste da Austrália
        - Índia Central
        - Leste dos EUA
        - Leste dos EUA 2
        - Norte da Europa
        - Centro-Sul dos Estados Unidos
        - Norte da Suíça
        - Sul do Reino Unido
        - Oeste da Europa
        - Oeste dos EUA 2
        - Oeste dos EUA 3
    - **Nome**: *insira um nome exclusivo*
    - **Tipo de preço**: selecione **F0** (*gratuito*) ou **S** (*padrão*) se F não estiver disponível.
    - **Conta de armazenamento**: nova conta de armazenamento:
      - **Nome da conta de armazenamento**: *insira um nome exclusivo*.
      - **Tipo de conta de armazenamento**: LRS Standard
    - **Aviso de IA Responsável**: selecionado.

1. Selecione **Revisar + criar** e, em seguida, selecione **Criar** para provisionar o recurso.
1. Aguarde a conclusão da implantação e acesse o recurso implantado.
1. Exiba a página **Chaves e Ponto de Extremidade**. Você precisará das informações nesta página mais adiante no exercício.

## Configurar o acesso baseado em função para seu usuário

> **OBSERVAÇÃO**: se você ignorar esta etapa, receberá um erro 403 ao tentar se conectar ao seu projeto personalizado. É importante que o usuário atual tenha essa função para acessar dados de blob da conta de armazenamento, mesmo que você seja o proprietário da conta de armazenamento.

1. Acesse a página da conta de armazenamento no portal do Azure.
2. Selecione **Controle de acesso (IAM)** no menu de navegação à esquerda.
3. Clique em **Adicionar** para Adicionar Atribuições de Função e escolha a função **Colaborador de dados do blob armazenamento** na conta de armazenamento.
4. Em **Atribuir acesso a**, selecione **Usuário, grupo ou entidade de serviço**.
5. Selecione **Selecionar membros**.
6. Selecione seu Usuário. É possível pesquisar nomes de usuário no campo **Selecionar**.

## Carregar anúncios de exemplo

Depois de criar o serviço de Linguagem de IA do Azure e a conta de armazenamento, você precisará carregar anúncios de exemplo para treinar seu modelo mais tarde.

1. Em uma nova guia do navegador, baixe anúncios classificados de amostra de `https://aka.ms/entity-extraction-ads` e extraia os arquivos para uma pasta de sua escolha.

2. No portal do Azure, navegue até a conta de armazenamento que você criou e selecione-a.

3. Na sua conta de armazenamento, selecione **Configuração**, localizada abaixo de **Configurações**, e ative a opção **Permitir acesso anônimo de blobs** e selecione **Salvar**.

4. Selecione **Contêineres** no menu esquerdo localizado abaixo de **Armazenamento de dados**. Na tela exibida, selecione **+ Contêiner**. Dê ao contêiner o nome `classifieds` e defina **Nível de acesso anônimo** como **Contêiner (acesso de leitura anônimo para contêineres e blobs)**.

    > **OBSERVAÇÃO**: ao configurar uma conta de armazenamento para uma solução real, tome cuidado para atribuir o nível de acesso apropriado. Para saber mais sobre cada nível de acesso, confira os [documentação do Armazenamento do Azure](https://learn.microsoft.com/azure/storage/blobs/anonymous-read-access-configure).

5. Depois de criar o contêiner, selecione-o e clique no botão **Carregar** para carregar os anúncios de exemplo baixados.

## Crie um projeto de reconhecimento de entidade nomeada personalizada

Agora você está pronto para criar um projeto personalizado de reconhecimento de entidade nomeada. Esse projeto proporciona um local de trabalho para criar, treinar e implantar o modelo.

> **OBSERVAÇÃO**: você também pode criar, compilar, treinar e implantar seu modelo por meio da API REST.

1. Em uma nova guia do navegador, abra o portal do Azure AI Language Studio em `https://language.cognitive.azure.com/` e entre usando a conta Microsoft associada à sua assinatura do Azure.
1. Se for solicitado a escolher um recurso de linguagem, selecione as seguintes configurações:

    - **Azure Directory**: o diretório do Azure contendo a sua assinatura.
    - **Assinatura do Azure**: sua assinatura do Azure.
    - **Tipo de recurso**: linguagem.
    - **Recurso de linguagem**: o recurso de Linguagem de IA do Azure criado anteriormente.

    Se você <u>não</u> foi solicitado a escolher um recurso de linguagem, pode ser porque você tem vários recursos de linguagem em sua assinatura; nesse caso:

    1. Na barra na parte superior da página, selecione o botão **Configurações(&#9881;)**.
    2. Na página **Configurações**, exiba a guia **Recursos**.
    3. Selecione o recurso de idioma que você acabou de criar e clique em **Alternar recurso**.
    4. Na parte superior da página, clique em **Language Studio** para retornar à home page do Language Studio.

1. Na parte superior do portal, no menu **Criar novo**, selecione **Reconhecimento personalizado de entidade nomeada**.

1. Crie um novo projeto com as seguintes configurações:
    - **Conexão de armazenamento**: *esse valor provavelmente já está preenchido. Se ainda não estiver, altere-o para sua conta de armazenamento*
    - **Informações Básicas:**
    - **Nome**: `CustomEntityLab`
        - **Idioma primário do texto**: inglês (EUA)
        - **Seu conjunto de dados inclui documentos que não estão no mesmo idioma?** : *Não*
        - **Descrição**: `Custom entities in classified ads`
    - **Contêiner:**
        - **Contêiner de armazenamento de blobs**: classificados
        - **Seus arquivos são rotulados com classes?**: não, preciso rotular meus arquivos como parte deste projeto

> **Dica**: se você receber um erro sobre não ter autorização para executar essa operação, será necessário adicionar uma atribuição de função. Para corrigir isso, adicionamos a função "Colaborador de Dados de Blob de Armazenamento" na conta de armazenamento do usuário que está executando o laboratório. Você encontra mais detalhes na [página da documentação](https://learn.microsoft.com/azure/ai-services/language-service/custom-named-entity-recognition/how-to/create-project?tabs=portal%2Clanguage-studio#enable-identity-management-for-your-resource).

## Rotular seus dados.

Agora que seu projeto foi criado, você precisa rotular os dados a fim de treinar o modelo para identificar entidades.

1. Se a página **Rotulagem de dados** ainda não estiver aberta, no painel à esquerda, selecione **Rotulagem de dados**. Você verá uma lista dos arquivos que carregou na conta de armazenamento.
1. No lado direito, no painel **Atividade**, selecione **Adicionar entidade** e adicione uma nova entidade chamada `ItemForSale`.
1.  Repita a etapa anterior para criar as seguintes entidades:
    - `Price`
    - `Location`
1. Depois de criar suas três entidades, selecione **Ad 1.txt** para que você possa lê-lo.
1. Em *Ad 1.txt*: 
    1. Realce o cabo da face do texto *face cord of firewood* e selecione a entidade **ItemForSale**.
    1. Realce o texto *Denver, CO* e selecione a entidade **Location**.
    1. Realce o texto *$90* e selecione a entidade **Price**.
1. No painel **Atividade**, observe que este documento será adicionado ao conjunto de dados para treinar o modelo.
1. Use o botão **Próximo documento** para acessar o documento seguinte e continue atribuindo texto às entidades apropriadas em todo o conjunto de documentos, adicionando-os ao conjunto de dados de treinamento.
1. Depois de rotular o último documento (*Ad 9.txt*), salve os rótulos.

## Treinar seu modelo

Depois de rotular os dados, você precisa treinar o modelo.

1. Selecione **Trabalhos de treinamento** à esquerda no painel.
2. Selecione **Iniciar um trabalho de treinamento**
3. Treinar um novo modelo chamado `ExtractAds`
4. Escolha **Dividir automaticamente o conjunto de testes dos dados de treinamento**

    > **DICA**: em seus projetos de extração, use a divisão de teste mais adequada aos seus dados. Para dados mais consistentes e conjuntos de dados maiores, o Serviço de Linguagem de IA do Azure dividirá automaticamente o conjunto de testes por percentual. Com conjuntos de dados menores, é importante treinar com a variedade certa de documentos de entrada possíveis.

5. Clique em **Treinar**

    > **IMPORTANTE**: às vezes, o treinamento do modelo pode levar vários minutos. Você receberá uma notificação quando ele for concluído.

## Avaliar o modelo

Em aplicativos do mundo real, é importante avaliar e melhorar seu modelo para verificar se ele está funcionando conforme o esperado. Duas páginas à esquerda mostram os detalhes do modelo treinado, bem como os testes que falharam.

Selecione **Desempenho do modelo** no menu do lado esquerdo e selecione seu modelo `ExtractAds`. Lá, você pode ver a pontuação do modelo, as métricas de desempenho e quando ele foi treinado. Você poderá ver se algum documento de teste falhou, e essas falhas ajudarão você a entender como melhorar.

## Implantar o seu modelo

Quando você estiver satisfeito com o treinamento do seu modelo, implante-o para começar a extrair entidades por meio da API.

1. No painel esquerdo, selecione **Implantando um modelo**.
2. Selecione **Adicionar implantação**, insira o nome `AdEntities` e selecione o modelo **ExtractAds**.
3. Clique em **Implantar** para implantar o modelo.

## Preparar-se para desenvolver um aplicativo no Cloud Shell

Para testar os recursos de extração de entidade personalizados do serviço de Linguagem de IA do Azure, você vai desenvolver um aplicativo de console simples no Azure Cloud Shell.

1. No Portal do Azure, use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um Cloud Shell no portal do Azure selecionando um ambiente do ***PowerShell***. O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure.

    > **Observação**: se você já criou um Cloud Shell que usa um ambiente *Bash*, alterne-o para o ***PowerShell***.

1. Na barra de ferramentas do Cloud Shell, no menu **Configurações**, selecione **Ir para a versão clássica** (isso é necessário para usar o editor de código).

    **<font color="red">Verifique se você mudou para a versão clássica do Cloud Shell antes de continuar.</font>**

1. No painel do PowerShell, insira os seguintes comandos para clonar o repositório GitHub para este exercício:

    ```
   rm -r mslearn-ai-language -f
   git clone https://github.com/microsoftlearning/mslearn-ai-language
    ```

    > **Dica**: conforme você colar comandos no cloudshell, a saída pode ocupar uma grande quantidade do espaço da tela. Você pode limpar a tela digitando o comando `cls` para facilitar o foco em cada tarefa.
    ```

1. After the repo has been cloned, navigate to the folder containing the application code files:  

    ```
    cd mslearn-ai-language/Labfiles/05-custom-entity-recognition/Python/custom-entities
    ```

## Configure your application

1. In the command line pane, run the following command to view the code files in the **custom-entities** folder:

    ```
   ls -a -l
    ```

    The files include a configuration file (**.env**) and a code file (**custom-entities.py**). The text your application will analyze is in the **ads** subfolder.

1. Create a Python virtual environment and install the Azure AI Language Text Analytics SDK package and other required packages by running the following command:

    ```
   python -m venv labenv ./labenv/bin/Activate.ps1 pip install -r requirements.txt azure-ai-textanalytics==5.3.0
    ```

1. Enter the following command to edit the application configuration file:

    ```
   code .env
    ```

    The file is opened in a code editor.

1. Update the configuration values to include the  **endpoint** and a **key** from the Azure Language resource you created (available on the **Keys and Endpoint** page for your Azure AI Language resource in the Azure portal).The file should already contain the project and deployment names for your custom entity extraction model.
1. After you've replaced the placeholders, within the code editor, use the **CTRL+S** command or **Right-click > Save** to save your changes and then use the **CTRL+Q** command or **Right-click > Quit** to close the code editor while keeping the cloud shell command line open.

## Add code to extract entities

1. Enter the following command to edit the application code file:

    ```
    code custom-entities.py
    ```

1. Review the existing code. You will add code to work with the AI Language Text Analytics SDK.

    > **Tip**: As you add code to the code file, be sure to maintain the correct indentation.

1. At the top of the code file, under the existing namespace references, find the comment **Import namespaces** and add the following code to import the namespaces you will need to use the Text Analytics SDK:

    ```python
   # import namespaces
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.textanalytics import TextAnalyticsClient
    ```

1. Na função **main**, observe que o código para carregar o ponto de extremidade e a chave do serviço de Linguagem de IA do Azure e os nomes do projeto e da implantação a partir do arquivo de configuração já foi fornecido. Em seguida, localize o comentário **Create client using endpoint and key** e adicione o seguinte código para criar um cliente de análise de texto:

    ```Python
   # Create client using endpoint and key
   credential = AzureKeyCredential(ai_key)
   ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

1. Observe que o código existente lê todos os arquivos na pasta **ads** e cria uma lista que contém seu conteúdo. Então, localize o comentário **Extract entities** e adicione o seguinte código:

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

1. Salve suas alterações (CTRL+S) e, em seguida, insira o seguinte comando para executar o programa (você maximiza o painel do Cloud Shell e redimensiona os painéis para ver mais texto no painel de linha de comando):

    ```
   python custom-entities.py
    ```

1. Observe a saída. O aplicativo deve listar detalhes das entidades encontradas em cada arquivo de texto.

## Limpeza

Quando não precisar mais do projeto, você poderá excluí-lo de sua página de **Projetos** no Language Studio. Você também pode remover o serviço de Linguagem de IA do Azure e a conta de armazenamento associada a ele no [portal do Azure](https://portal.azure.com).
