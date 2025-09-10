---
lab:
  title: Classificação personalizada de textos
  description: Aplique classificações personalizadas à entrada de texto usando a Linguagem de IA do Azure.
---

# Classificação personalizada de textos

A Linguagem de IA do Azure fornece vários recursos de PNL, incluindo identificação de frase-chave, resumo de texto e análise de sentimento. O serviço de Linguagem também fornece funcionalidades personalizadas, como respostas às perguntas personalizadas e classificação de textos personalizada.

Para testar a classificação de texto personalizada do serviço de Linguagem de IA do Azure, você configurará o modelo usando o Estúdio de Linguagem e, em seguida, usará um aplicativo Python para testá-lo.

Embora este exercício seja baseado em Python, você pode desenvolver aplicativos de classificação de textos usando vários SDKs específicos da linguagem, incluindo:

- [Biblioteca de clientes de Análise de Texto de IA do Azure para Python](https://pypi.org/project/azure-ai-textanalytics/)
- [Biblioteca de clientes de Análise de Texto de IA do Azure para .NET](https://www.nuget.org/packages/Azure.AI.TextAnalytics)
- [Biblioteca de clientes de Análise de Texto de IA do Azure para JavaScript](https://www.npmjs.com/package/@azure/ai-text-analytics)

Este exercício levará aproximadamente **35** minutos.

## Provisionar um recurso da *Linguagem de IA do Azure*

Caso ainda não tenha um na sua assinatura, provisione um recurso do **serviço de Linguagem de IA do Azure**. Além disso, use a classificação de textos personalizada; você precisa habilitar o **recurso de extração e classificação de textos personalizada**.

1. Abra o portal do Azure em `https://portal.azure.com` e entre usando a conta Microsoft associada à sua assinatura do Azure.
1. Selecione **Criar um recurso**.
1. No campo de pesquisa, pesquise **Serviço de linguagem**. Nos resultados, selecione **Criar** em **Serviço de Linguagem**.
1. Selecione a caixa que inclui **Classificação de texto personalizada**. Em seguida, selecione **Continuar para criar o recurso**.
1. Crie um recurso com as seguintes configurações:
    - **Assinatura**: *sua assinatura do Azure*.
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
    - **Nome**: *insira um nome exclusivo*.
    - **Tipo de preço**: selecione **F0** (*gratuito*), ou **S** (*Standard*) se F não estiver disponível.
    - **Conta de armazenamento**: nova conta de armazenamento
      - **Nome da conta de armazenamento**: *insira um nome exclusivo*.
      - **Tipo de conta de armazenamento**: LRS Standard
    - **Aviso de IA Responsável**: selecionado.

1. Selecione **Revisar + criar** e, em seguida, selecione **Criar** para provisionar o recurso.
1. Aguarde a conclusão da implantação e acesse o grupo de recursos.
1. Localize a conta de armazenamento que você criou, selecione-a e verifique se o _Tipo de conta_ é **StorageV2**. Se for v1, atualize o tipo de conta de armazenamento nessa página de recursos.

## Configurar o acesso baseado em função para seu usuário

> **OBSERVAÇÃO**: Se você ignorar esta etapa, receberá um erro 403 ao tentar se conectar ao seu projeto personalizado. É importante que o usuário atual tenha essa função para acessar dados de blob da conta de armazenamento, mesmo que você seja o proprietário da conta de armazenamento.

1. Acesse a página da conta de armazenamento no portal do Azure.
2. Selecione **Controle de acesso (IAM)** no menu de navegação à esquerda.
3. Clique em **Adicionar ** para adicionar atribuições de função e escolha a função **Proprietário de dados do blob armazenamento** na conta de armazenamento.
4. Em **Atribuir acesso a**, selecione **Usuário, grupo ou entidade de serviço**.
5. Selecione **Selecionar membros**.
6. Selecione seu Usuário. É possível pesquisar nomes de usuário no campo **Selecionar**.

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

## Preparar-se para desenvolver um aplicativo no Cloud Shell

Para testar os recursos de classificação de texto personalizados do serviço de Linguagem de IA do Azure, você vai desenvolver um aplicativo de console simples no Azure Cloud Shell.

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

1. Após o repositório ser clonado, navegue até a pasta que contém os arquivos de código do aplicativo:  

    ```
   cd mslearn-ai-language/Labfiles/04-text-classification/Python/classify-text
    ```

## Configurar seu aplicativo

1. No painel de linha de comando, execute o seguinte comando para exibir os arquivos de código na pasta **classify-text**:

    ```
   ls -a -l
    ```

    Os arquivos incluem um arquivo de configuração (**.env**) e um arquivo de código (**classify-text.py**). O texto que seu aplicativo analisará está na subpasta **articles**.

1. Crie um ambiente virtual Python e instale o pacote SDK de Análise de Texto de Linguagem de IA do Azure e outros pacotes necessários executando o seguinte comando:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-textanalytics==5.3.0
    ```

1. Insira o seguinte comando para editar o arquivo de configuração de aplicativo:

    ```
   code .env
    ```

    O arquivo é aberto em um editor de código.

1. Atualize os valores de configuração para incluir o **ponto de extremidade** e uma **chave** do recurso Linguagem de IA do Azure que você criou (disponível na página **Chaves e Ponto de extremidade** do seu recurso Linguagem de IA do Azure no portal do Azure). O arquivo já deve conter os nomes do projeto e da implantação do seu modelo de classificação de texto.
1. Depois de substituir os espaços reservados, use o comando **CTRL+S** ou **botão direito do mouse > Salvar** para salvar as suas alterações e, em seguida, use o comando **CTRL+Q** ou **botão direito do mouse > Sair** para fechar o editor de código, mantendo a linha de comando do Cloud Shell aberta.

## Adicionar código para classificar documentos

1. Insira o seguinte comando para editar o arquivo de código do aplicativo:

    ```
    code classify-text.py
    ```

1. Revise o código existente. Você adicionará código para trabalhar com o SDK de Análise de Texto da Linguagem de IA.

    > **Dica**: ao adicionar código ao arquivo, certifique-se de manter o recuo correto.

1. No topo do arquivo de código, abaixo das referências de namespace existentes, localize o comentário **Import namespaces** e adicione o seguinte código para importar os namespaces necessários para usar o SDK de Análise de Texto:

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

1. Observe que o código existente lê todos os arquivos na pasta **articles** e cria uma lista que contém seu conteúdo. Em seguida, localize o comentário **Obter classificações** e adicione o seguinte código:

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

1. Salve suas alterações (CTRL+S) e, em seguida, insira o seguinte comando para executar o programa (você maximiza o painel do Cloud Shell e redimensiona os painéis para ver mais texto no painel de linha de comando):

    ```
   python classify-text.py
    ```

1. Observe a saída. O aplicativo deve listar uma classificação e pontuação de confiança para cada arquivo de texto.

## Limpar

Quando não precisar mais do projeto, você poderá excluí-lo de sua página de **Projetos** no Estúdio de Linguagem. Você também pode remover o serviço de Linguagem de IA do Azure e a conta de armazenamento associada a ele no [portal do Azure](https://portal.azure.com).
