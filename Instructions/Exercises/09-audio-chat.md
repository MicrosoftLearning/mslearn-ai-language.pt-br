---
lab:
  title: Desenvolver um aplicativo de chat habilitado para áudio
  description: Saiba como usar a Fábrica de IA do Azure para criar um aplicativo de IA generativa que aceite entradas de áudio.
---

# Desenvolver um aplicativo de chat habilitado para áudio

Neste exercício, você usará o modelo de IA generativa *Phi-4-multimodal-instruct* para gerar respostas para prompts que incluem arquivos de áudio. Você desenvolverá um aplicativo que fornece assistência de IA com produtos frescos em uma mercearia usando a Fábrica de IA do Azure e o serviço de inferência de modelo da IA do Azure.

Este exercício levará aproximadamente **30** minutos.

## Criar um projeto do Azure AI Foundry

Vamos começar criando um projeto da Fábrica de IA do Azure.

1. Em um navegador da Web, abra o [Portal da Fábrica de IA do Azure](https://ai.azure.com) em `https://ai.azure.com` e entre usando suas credenciais do Azure. Feche todas as dicas ou painéis de início rápido abertos na primeira vez que você entrar e, se necessário, use o logotipo da **Fábrica de IA do Azure** no canto superior esquerdo para navegar até a home page, que é semelhante à imagem a seguir:

    ![Captura de tela do portal do Azure AI Foundry.](../media/ai-foundry-home.png)

1. Na home page, selecione **+Criar projeto**.
1. No assistente **Criar um projeto**, insira um nome de projeto adequado e, se um hub existente for sugerido, escolha a opção de criar um novo. Em seguida, examine os recursos do Azure que serão criados automaticamente para dar suporte ao hub e ao projeto.
1. Selecione **Personalizar** e especifique as seguintes configurações para o hub:
    - **Nome do hub**: *um nome válido para o seu hub*
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *criar ou selecionar um grupo de recursos*
    - **Região**: selecione uma das seguintes regiões\*:
        - Leste dos EUA
        - Leste dos EUA 2
        - Centro-Norte dos EUA
        - Centro-Sul dos Estados Unidos
        - Suécia Central
        - Oeste dos EUA
        - Oeste dos EUA 3
    - **Conectar os Serviços de IA do Azure ou o OpenAI do Azure**: *Criar um novo recurso de Serviços de IA*
    - **Conectar-se à Pesquisa de IA do Azure**: Ignorar a conexão

    > \* No momento em que este artigo foi escrito, o modelo *Phi-4-multimodal-instruct* da Microsoft que usaremos neste exercício está disponível nessas regiões. Você pode verificar a disponibilidade regional mais recente para modelos específicos na [documentação da Fábrica de IA do Azure](https://learn.microsoft.com/azure/ai-foundry/how-to/deploy-models-serverless-availability#region-availability). No caso de um limite de cota regional ser atingido mais adiante no exercício, há a possibilidade de você precisar criar outro recurso em uma região diferente.

1. Clique em **Avançar** e revise a configuração. Em seguida, selecione **Criar** e aguarde a conclusão do processo.
1. Quando o projeto for criado, feche todas as dicas exibidas e examine a página do projeto no Portal da Fábrica de IA do Azure, que deve ser semelhante à imagem a seguir:

    ![Captura de tela dos detalhes de um projeto IA do Azure no Portal da Fábrica de IA do Azure.](../media/ai-foundry-project.png)

## Implantar um modelo multimodal

Agora você já pode implantar um modelo multimodal que aceite entrada baseada em áudio. Você pode escolher entre vários modelos, incluindo o modelo *gpt-4o* do OpenAI. Neste exercício, usaremos um modelo *Phi-4-multimodal-instruct*.

1. Na barra de ferramentas no canto superior direito da página do projeto da Fábrica de IA do Azure, use o ícone **Visualizar recursos** (**&#9215;**) para garantir que o recurso **Implantar modelos no serviço de inferência de modelo da IA do Azure** esteja habilitado. Esse recurso garante que a implantação do modelo esteja disponível para o serviço de inferência de IA do Azure, que você usará no código do aplicativo.
1. No painel à esquerda do seu projeto, na seção **Meus ativos**, selecione a página **Modelos + pontos de extremidade**.
1. Na página **Modelos + pontos extremidades**, na guia **Implantações de modelo**, no menu **+ Implantar modelo**, selecione **Implantar modelo base**.
1. Procure o modelo **Phi-4-multimodal-instruct** na lista e, em seguida, selecione-o e confirme-o.
1. Concorde com o contrato de licença, se solicitado, e implante o modelo com as seguintes configurações selecionando **Personalizar** nos detalhes da implantação:
    - **Nome da implantação**: *Um nome válido para sua implantação de modelo*
    - **Tipo de implantação**: padrão global
    - **Detalhes da implantação**: *use as configurações padrão*
1. Aguarde que o estado de provisionamento de implantação seja **Concluído**.

## Criar um aplicativo cliente

Agora que você implantou o modelo, pode usar a implantação em um aplicativo cliente.

> **Dica**: você pode optar por desenvolver sua solução usando Python ou Microsoft C#. Siga as instruções na seção apropriada para o idioma escolhido.

### Preparar a configuração de aplicativo

1. No Portal da Fábrica de IA do Azure, visualize a página **Visão geral** do seu projeto.
1. Na área **Detalhes do projeto**, observe a **Cadeia de conexão do projeto**. Você usará essa cadeia de conexão para se conectar ao seu projeto em um aplicativo cliente.
1. Abra uma nova guia do navegador (mantendo o portal da Fábrica de IA do Azure aberto na guia existente). Em seguida, na nova guia, navegue até o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com`; efetue login com suas credenciais do Azure, se solicitado.

    Feche todas as notificações de boas-vindas para ver a home page do portal do Azure.

1. Use o botão **[\>_]** à direita da barra de pesquisa na parte superior da página para criar um Cloud Shell no portal do Azure selecionando um ambiente do ***PowerShell*** sem armazenamento em sua assinatura.

    O Cloud Shell fornece uma interface de linha de comando em um painel na parte inferior do portal do Azure. Você pode redimensionar ou maximizar esse painel para facilitar o trabalho.

    > **Observação**: se você já criou um Cloud Shell que usa um ambiente *Bash*, alterne-o para o ***PowerShell***.

1. Na barra de ferramentas do Cloud Shell, no menu **Configurações**, selecione **Ir para a versão clássica** (isso é necessário para usar o editor de código).

    **<font color="red">Verifique se você mudou para a versão clássica do Cloud Shell antes de continuar.</font>**

1. No painel do Cloud Shell, insira os seguintes comandos para clonar o repositório GitHub que contém os arquivos de código para este exercício (digite o comando ou copie-o para a área de transferência e clique com o botão direito do mouse na linha de comando e cole como texto sem formatação):


    ```
    rm -r mslearn-ai-audio -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-language mslearn-ai-audio
    ```

    > **Dica**: conforme você colar comandos no cloudshell, a saída pode ocupar uma grande quantidade do espaço da tela. Você pode limpar a tela digitando o comando `cls` para facilitar o foco em cada tarefa.

1. Após o repositório ser clonado, navegue até a pasta que contém os arquivos de código do aplicativo:  

    **Python**

    ```
    cd mslearn-ai-audio/Labfiles/09-audio-chat/python
    ```

    **C#**

    ```
    cd mslearn-ai-audio/Labfiles/09-audio-chat/c-sharp
    ```

1. No painel de linha de comando do Cloud Shell, digite o seguinte comando para instalar as bibliotecas que você usará:

    **Python**

    ```
    python -m venv labenv
    ./labenv/bin/Activate.ps1
    pip install python-dotenv azure-identity azure-ai-projects azure-ai-inference
    ```

    **C#**

    ```
    dotnet add package Azure.Identity
    dotnet add package Azure.AI.Inference --version 1.0.0-beta.3
    dotnet add package Azure.AI.Projects --version 1.0.0-beta.3
    ```

1. Digite o seguinte comando para editar o arquivo de configuração que foi fornecido:

    **Python**

    ```
    code .env
    ```

    **C#**

    ```
    code appsettings.json
    ```

    O arquivo abrirá em um editor de código.

1. No arquivo de código, substitua o espaço reservado **your_project_connection_string** pela cadeia de conexão do seu projeto (copiada da página **Visão Geral** do projeto no portal da Fábrica de IA do Azure) e o espaço reservado **your_model_deployment** pelo nome que você atribuiu à implantação do modelo Phi-4-multimodal-instruct.

1. Após substituir os espaços reservados, no editor de código, use o comando **CTRL+S** ou **Clique com o botão direito > Salvar** para salvar as alterações e, em seguida, use o comando **CTRL+Q** ou **Clique com o botão direito > Sair** para fechar o editor de código, mantendo a linha de comando do Cloud Shell aberta.

### Escrever código para se conectar ao projeto e obter um cliente de chat para o modelo

> **Dica**: ao adicionar código, certifique-se de manter o recuo correto.

1. Digite o seguinte comando para editar o arquivo de código que foi fornecido:

    **Python**

    ```
    code audio-chat.py
    ```

    **C#**

    ```
    code Program.cs
    ```

1. No arquivo de código, observe as instruções existentes que foram adicionadas na parte superior do arquivo para importar os namespaces do SDK necessários. Em seguida, localize o comentário **Add references**, adicione o seguinte código para referenciar os namespaces nas bibliotecas que você instalou anteriormente:

    **Python**

    ```python
    # Add references
    from dotenv import load_dotenv
    from azure.identity import DefaultAzureCredential
    from azure.ai.projects import AIProjectClient
    from azure.ai.inference.models import (
        SystemMessage,
        UserMessage,
        TextContentItem,
        AudioContentItem,
        InputAudio,
        AudioContentFormat,
    )
    ```

    **C#**

    ```csharp
    // Add references
    using Azure.Identity;
    using Azure.AI.Projects;
    using Azure.AI.Inference;
    ```

1. Na função **main**, no comentário **Get configuration settings**, observe que o código carrega a cadeia de conexão do projeto e os valores do nome de implantação do modelo que você definiu no arquivo de configuração.

1. Localize o comentário **Initialize the project client** e adicione o seguinte código para se conectar ao seu projeto da Fábrica de IA do Azure usando as credenciais do Azure com as quais você entrou:

    **Python**

    ```python
    # Initialize the project client
    project_client = AIProjectClient.from_connection_string(
        conn_str=project_connection,
        credential=DefaultAzureCredential())
    ```

    **C#**

    ```csharp
    // Initialize the project client
    var projectClient = new AIProjectClient(project_connection,
                        new DefaultAzureCredential());
    ```

1. No comentário **Get a chat client**, adicione o seguinte código para criar um objeto cliente para conversar com o modelo:

    **Python**

    ```python
    # Get a chat client
    chat_client = project_client.inference.get_chat_completions_client(model=model_deployment)
    ```

    **C#**

    ```csharp
    // Get a chat client
    ChatCompletionsClient chat = projectClient.GetChatCompletionsClient();
    ```
    

### Escrever código para enviar um prompt de áudio baseado em URL

1. No editor de código do arquivo **audio-chat.py**, na seção de loop, no comentário **Get a response to audio input**, adicione o seguinte código para enviar um prompt que inclua o seguinte áudio:

    <video controls src="../media/manzanas.mp4" title="Um pedido de maçãs" width="150"></video>

    **Python**

    ```python
    # Get a response to audio input
    file_path = "https://github.com/microsoftlearning/mslearn-ai-language/raw/refs/heads/main/labfiles/09-audio-chat/data/manzanas.mp3"
    response = chat_client.complete(
        messages=[
            SystemMessage(system_message),
            UserMessage(
                [
                    TextContentItem(text=prompt),
                    {
                        "type": "audio_url",
                        "audio_url": {"url": file_path}
                    }
                ]
            )
        ]
    )
    print(response.choices[0].message.content)
    ```

    **C#**

    ```csharp
    // Get a response to audio input
    string audioUrl = "https://github.com/microsoftlearning/mslearn-ai-language/raw/refs/heads/main/labfiles/09-audio-chat/data/manzanas.mp3";
    var requestOptions = new ChatCompletionsOptions()
    {
        Messages =
        {
            new ChatRequestSystemMessage(system_message),
            new ChatRequestUserMessage(
                new ChatMessageTextContentItem(prompt),
                new ChatMessageAudioContentItem(new Uri(audioUrl))),
        },
        Model = model_deployment
    };
    var response = chat.Complete(requestOptions);
    Console.WriteLine(response.Value.Content);
    ```

1. Use o comando **CTRL+S** para salvar suas alterações no arquivo de código. Você também pode fechar o editor de código (**CTRL+Q**), se desejar.

1. No painel da linha de comando do Cloud Shell abaixo do editor de código, insira o seguinte comando para executar o aplicativo:

    **Python**

    ```
    python audio-chat.py
    ```

    **C#**

    ```
    dotnet run
    ```

1. Quando solicitado, insira o prompt `What is this customer saying in English?`

1. Revise a resposta.

### Usar um prompt diferente

1. No editor de código do código do aplicativo, localize o código adicionado anteriormente no comentário **Get a response to audio input**. Em seguida, modifique o código da seguinte maneira para selecionar um arquivo de áudio diferente:

    <video controls src="../media/caomei.mp4" title="Um pedido de morangos" width="150"></video>

    **Python**

    ```python
    # Get a response to audio input
    file_path = "https://github.com/microsoftlearning/mslearn-ai-language/raw/refs/heads/main/labfiles/09-audio-chat/data/caomei.mp3"
    response = chat_client.complete(
        messages=[
            SystemMessage(system_message),
            UserMessage(
                [
                    TextContentItem(text=prompt),
                    {
                        "type": "audio_url",
                        "audio_url": {"url": file_path}
                    }
                ]
            )
        ]
    )
    print(response.choices[0].message.content)
    ```

    **C#**

    ```csharp
    // Get a response to audio input
    string audioUrl = "https://github.com/microsoftlearning/mslearn-ai-language/raw/refs/heads/main/labfiles/09-audio-chat/data/caomei.mp3";
    var requestOptions = new ChatCompletionsOptions()
    {
        Messages =
        {
            new ChatRequestSystemMessage(system_message),
            new ChatRequestUserMessage(
                new ChatMessageTextContentItem(prompt),
                new ChatMessageAudioContentItem(new Uri(audioUrl))),
        },
        Model = model_deployment
    };
    var response = chat.Complete(requestOptions);
    Console.WriteLine(response.Value.Content);
    ```

1. Use o comando **CTRL+S** para salvar suas alterações no arquivo de código. Você também pode fechar o editor de código (**CTRL+Q**), se desejar.

1. No painel da linha de comando do Cloud Shell abaixo do editor de código, insira o seguinte comando para executar o aplicativo:

    **Python**

    ```
    python audio-chat.py
    ```

    **C#**

    ```
    dotnet run
    ```

1. Quando solicitado, insira o seguinte prompt:

    ```
    A customer left this voice message, can you summarize it?
    ```

1. Revise a resposta. Em seguida, insira `quit` para sair do programa.

    > **Observação**: neste aplicativo simples, não implementamos lógica para reter o histórico de conversas; portanto, o modelo tratará cada prompt como uma nova solicitação sem contexto do prompt anterior.

1. Você pode continuar executando o aplicativo, escolhendo diferentes tipos de prompt e testando diferentes prompts. Quando terminar, digite `quit` para sair do programa.

    Se você tiver tempo, pode modificar o código para usar um prompt do sistema diferente e seus próprios arquivos de áudio acessíveis pela Internet.

    > **Observação**: neste aplicativo simples, não implementamos lógica para reter o histórico de conversas; portanto, o modelo tratará cada prompt como uma nova solicitação sem contexto do prompt anterior.

## Resumo

Neste exercício, você usou a Fábrica de IA do Azure e o SDK de Inferência de IA do Azure para criar um aplicativo cliente que usa um modelo multimodal para gerar respostas para áudio.

## Limpar

Se tiver terminado de explorar o portal da Fábrica de IA do Azure, deverá excluir os recursos que criou neste exercício para evitar incorrer em custos desnecessários do Azure.

1. Retorne à guia do navegador que contém o portal do Azure (ou reabra o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com` em uma nova guia do navegador) e visualize o conteúdo do grupo de recursos onde você implantou os recursos usados neste exercício.
1. Na barra de ferramentas, selecione **Excluir grupo de recursos**.
1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.
