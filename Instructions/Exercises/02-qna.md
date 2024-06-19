---
lab:
  title: Criar uma solução de respostas às perguntas
  module: Module 6 - Create question answering solutions with Azure AI Language
---

# Criar uma solução de respostas às perguntas

Um dos cenários de conversação mais comuns é fornecer suporte por meio de uma base de dados de conhecimento de FAQs (perguntas frequentes). Muitas organizações publicam perguntas frequentes como documentos ou páginas da Web, o que funciona bem para um pequeno conjunto de pares de perguntas e respostas, mas documentos grandes podem ser difíceis e demorados de pesquisar.

A **Linguagem de IA do Azure** inclui uma capacidade de *respostas às perguntas* que permite criar uma base de dados de conhecimento de pares de perguntas e respostas que pode ser consultada usando entrada de linguagem natural e é mais comumente usada como um recurso que um bot pode usar para procurar respostas a perguntas enviadas pelos usuários.

## Provisionar um recurso da *Linguagem de IA do Azure*

Caso ainda não tenha um na sua assinatura, provisione um recurso de **serviço de Linguagem de IA do Azure**. Além disso, para criar e hospedar uma base de dados de conhecimento para respostas às perguntas, você precisa habilitar o recurso **Respostas às perguntas**.

1. Abra o portal do Azure em `https://portal.azure.com` e entre usando a conta Microsoft associada à sua assinatura do Azure.
1. No campo de pesquisa na parte superior, insira **Serviços de IA do Azure** e pressione **Inserir**.
1. Selecione **Criar** no recurso **Serviço de linguagem**, nos resultados.
1. **Selecione** o bloco **Respostas às perguntas personalizadas**. Em seguida, selecione **Continuar para criar o recurso**. Você precisará inserir as seguintes configurações:

    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *escolha ou crie um grupo de recursos*.
    - **Região**: *escolha qualquer localização disponível*
    - **Nome**: *insira um nome exclusivo*
    - **Tipo de preço**: selecione **F0** (*gratuito*), ou **S** (*Standard*) se F não estiver disponível.
    - **Região de pesquisa do Azure**: *escolha uma localização na mesma região global que seu recurso de linguagem*
    - **Tipo de preço de pesquisa do Azure**: F (Gratuito) (*se esse não estiver disponível, selecione B (Básico)*)
    - **Aviso de IA Responsável**: *Concordar*

1. Selecione **Criar + revisar** e, em seguinda, selecione **Criar**.

    > **OBSERVAÇÃO** As respostas às perguntas personalizadas usa a Pesquisa do Azure para indexar e consultar a base de dados de conhecimento de perguntas e respostas.

1. Aguarde a conclusão da implantação e acesse o recurso implantado.
1. Exiba a página **Chaves e Ponto de Extremidade**. Você precisará das informações nesta página mais adiante no exercício.

## Criar um projeto de respostas às perguntas

Para criar uma base de dados de conhecimento para as respostas às perguntas em seu recurso de Linguagem de IA do Azure, você pode usar o portal do Language Studio para criar um projeto de respostas às perguntas. Nesse caso, você criará uma base de dados de conhecimento contendo perguntas e respostas sobre o [Microsoft Learn](https://docs.microsoft.com/learn).

1. Em uma nova guia do navegador, vá para o portal do Language Studio em [https://language.cognitive.azure.com/](https://language.cognitive.azure.com/) e entre usando a conta Microsoft associada à sua assinatura do Azure.
1. Se você for solicitado a escolher um recurso de linguagem, selecione as seguintes configurações:
    - **Azure Directory**: o diretório do Azure contendo a sua assinatura.
    - **Assinatura do Azure**: sua assinatura do Azure.
    - **Tipo de recurso**: linguagem
    - **Nome do recurso**: o recurso de Linguagem de IA do Azure criado anteriormente.

    Se você <u>não</u> foi solicitado a escolher um recurso de linguagem, pode ser porque você tem vários recursos de linguagem em sua assinatura; nesse caso:

    1. Na barra na parte superior da página, clique no botão **Configurações (&#9881;)**.
    2. Na página **Configurações**, exiba a guia **Recursos**.
    3. Selecione o recurso de idioma que você acabou de criar e clique em **Alternar recurso**.
    4. Na parte superior da página, clique em **Language Studio** para retornar à home page do Language Studio.

1. Na parte superior do portal, no menu **Criar novo**, selecione **Respostas às perguntas personalizadas**.
1. No assistente ***Criar um projeto**, na página **Escolher configuração de linguagem**, selecione a opção **Definir a linguagem para todos os projetos neste recurso** e selecione **Inglês** como linguagem. Em seguida, selecione **Avançar**.
1. Na página **Inserir informações básicas**, insira os seguintes detalhes:
    - **Nome**`LearnFAQ`
    - **Descrição**: `FAQ for Microsoft Learn`
    - **Resposta padrão quando nenhuma resposta é retornada**: `Sorry, I don't understand the question`
1. Selecione **Avançar**.
1. Na página **Revisar e concluir**, selecione **Criar projeto**.

## Adicionar fontes à base de dados de conhecimento

Você pode criar uma base de dados de conhecimento do zero, mas é comum começar importando perguntas e respostas de uma página ou documento de perguntas frequentes existente. Nesse caso, você importará dados de uma página da Web de perguntas frequentes existente para o Microsoft Learn e também importará algumas perguntas e respostas predefinidas de "bate-papo" para dar suporte a trocas conversacionais comuns.

1. Na página **Gerenciar fontes** para seu projeto de respostas às perguntas, na lista **&#9547; Adicionar fonte**, selecione **URLs**. Em seguida, na caixa de diálogo **Adicionar URLs**, selecione **&#9547; Adicionar URL** e defina o seguinte nome e URL antes de selecionar **Adicionar tudo** para adicioná-la à base de dados de conhecimento:
    - **Nome**: `Learn FAQ Page`
    - **URL**: `https://docs.microsoft.com/en-us/learn/support/faq`
1. Na página **Gerenciar fontes** para seu projeto de respostas às perguntas, na lista **&#9547; Adicionar fonte**, selecione **Bate-papo**. Na caixa de diálogo **Adicionar bate-papo**, selecione **Amigável** e selecione **Adicionar bate-papo**.

## Editar a base de dados de conhecimento

Sua base de dados de conhecimento foi preenchida com pares de perguntas e respostas das perguntas frequentes do Microsoft Learn, complementadas com um conjunto de pares de perguntas e respostas de *bate-papo* conversacionais. Você pode estender a base de dados de conhecimento adicionando pares de perguntas e respostas adicionais.

1. Em seu projeto **LearnFAQ** no Language Studio, selecione a página **Editar base de dados de conhecimento** para ver os pares de perguntas e respostas existentes (se algumas dicas forem exibidas, leia-as e escolha **Entendi** para ignorá-las ou selecione **Pular todas**)
1. Na base de dados de conhecimento, na guia **Pares de perguntas e respostas**, selecione **&#65291;** e crie um novo par de perguntas e respostas com as seguintes configurações:
    - **Origem**:  `https://docs.microsoft.com/en-us/learn/support/faq`
    - **Pergunta**: `What are Microsoft credentials?`
    - **Resposta**: `Microsoft credentials enable you to validate and prove your skills with Microsoft technologies.`
1. Selecione **Concluído**.
1. Na página da pergunta **O que são credenciais da Microsoft?** que é criada, expanda as **Perguntas alternativas**. Em seguida, adicione a pergunta alternativa `How can I demonstrate my Microsoft technology skills?`.

    Em alguns casos, faz sentido permitir que o usuário acompanhe uma resposta criando uma conversa em *vários turnos* que permite ao usuário refinar iterativamente a pergunta para chegar à resposta de que precisa.

1. Na resposta inserida para a pergunta de certificação, expanda **Prompts de acompanhamento** e adicione o seguinte prompt de acompanhamento:
    - **SMS exibido no prompt para o usuário**: `Learn more about credentials`.
    - Selecione a guia **Criar link para novo par** e insira este texto: `You can learn more about credentials on the [Microsoft credentials page](https://docs.microsoft.com/learn/credentials/).`
    - Selecione **Mostrar apenas no fluxo contextual**. Essa opção garante que a resposta só seja retornada no contexto de uma pergunta de acompanhamento da pergunta de certificação original.
1. Selecione **Adicionar prompt**.

## Treinar e testar a base de conhecimento

Agora que você tem uma base de dados de conhecimento, pode testá-la no Language Studio.

1. Salve as alterações na sua base de dados de conhecimento selecionando o botão **Salvar** na guia **Pares de peguntas e respostas** à esquerda.
1. Depois que as alterações forem salvas, selecione o botão **Testar** para abrir o painel de teste.
1. No painel de teste, na parte superior, anule a seleção **Incluir resposta curta** (se ainda não estiver desmarcada). Em seguida, na parte inferior, insira a mensagem `Hello`. Uma resposta adequada deve ser retornada.
1. No painel de teste, na parte inferior, insira a mensagem `What is Microsoft Learn?`. Uma resposta apropriada das perguntas frequentes deve retornar.
1. Insira a mensagem `Thanks!` Uma resposta de bate-papo apropriada deve ser retornada.
1. Insira a mensagem `Tell me about Microsoft credentials`. A resposta que você criou deve ser retornada junto com um link de prompt de acompanhamento.
1. Selecione o link de acompanhamento **Saiba mais sobre credenciais**. A resposta de acompanhamento com um link para a página de certificação deve ser retornada.
1. Quando terminar de testar a base de dados de conhecimento, feche o painel de teste.

## Implantar a base de dados de conhecimento

A base de dados de conhecimento fornece um serviço de back-end que os aplicativos cliente podem usar para responder a perguntas. Agora você está pronto para publicar sua base de dados de conhecimento e acessar sua interface REST a partir de um cliente.

1. No projeto **LearnFAQ** no Language Studio, selecione a página **Implantar base de dados de conhecimento**.
1. Na parte superior da página, selecione **Implantar**. Em seguida, selecione **Implantar** para confirmar que deseja implantar a base de dados de conhecimento.
1. Quando a implantação estiver concluída, selecione **Obter URL de previsão** para exibir o ponto de extremidade REST da sua base de dados de conhecimento e observe que a solicitação de amostra inclui parâmetros para:
    - **projectName**: o nome do seu projeto (que deve ser *LearnFAQ*)
    - **deploymentName**: o nome da sua implantação (que deve ser *produção*)
1. Feche a caixa de diálogo URL de previsão.

## Prepare-se para desenvolver um aplicativo no Visual Studio Code

Você desenvolverá seu aplicativo de respostas às perguntas usando o Visual Studio Code. Os arquivos de código para seu aplicativo foram fornecidos em um repositório do GitHub.

> **Dica**: se você já clonou o repositório **mslearn-ai-language**, abra-o no Visual Studio Code. Caso contrário, siga estas etapas para cloná-lo em seu ambiente de desenvolvimento.

1. Inicie o Visual Studio Code.
2. Abra a paleta (SHIFT+CTRL+P) e execute o comando **Git: Clone** para clonar o repositório `https://github.com/MicrosoftLearning/mslearn-ai-language` em uma pasta local (não importa qual pasta).
3. Depois que o repositório for clonado, abra a pasta no Visual Studio Code.

    > **Observação**: Se o Visual Studio Code mostrar uma mensagem pop-up para solicitar que você confie no código que está abrindo, clique na opção **Sim, confio nos autores** no pop-up.

4. Aguarde enquanto os arquivos adicionais são instalados para dar suporte aos projetos de código C# no repositório.

    > **Observação**: se você for solicitado a adicionar os ativos necessários para compilar e depurar, selecione **Agora não**.

## Configurar seu aplicativo

Aplicativos para C# e Python foram fornecidos, bem como um arquivo de texto de exemplo que você usará para testar o resumo. Ambos os aplicativos apresentam a mesma funcionalidade. Primeiro, você concluirá algumas partes importantes do aplicativo para habilitar o uso do recurso Linguagem de IA do Azure.

1. No Visual Studio Code, no painel **Explorer**, navegue até a pasta **Labfiles/02-qna** e expanda a pasta **CSharp** ou **Python** dependendo da sua preferência de linguagem e da pasta **qna-app** que ela contém. Cada pasta contém os arquivos específicos de linguagem de um aplicativo no qual você integrará à funcionalidade de respostas às perguntas de Linguagem de IA do Azure.
2. Clique com o botão direito do mouse na pasta **qna-app** que contém seus arquivos de código e abra um terminal integrado. Em seguida, instale o pacote SDK de respostas às perguntas de Linguagem de IA do Azure executando o comando apropriado para sua preferência de linguagem:

    **C#**:

    ```
    dotnet add package Azure.AI.Language.QuestionAnswering
    ```

    **Python**:

    ```
    pip install azure-ai-language-questionanswering
    ```

3. No painel **Explorer**, na pasta **qna-app**, abra o arquivo de configuração da linguagem de sua preferência

    - **C#**: appsettings.json
    - **Python**: .env
    
4. Atualize os valores de configuração para incluir o  **ponto de extremidade** e uma **chave** do recurso de linguagem do Azure que você criou (disponível na página **Chaves e Ponto de Extremidade** para seu recurso de Linguagem de IA do Azure no portal do Azure). O nome do projeto e o nome da implantação da base de dados de conhecimento implantada também devem estar nesse arquivo.
5. Salve o arquivo de configuração.

## Adicionar código ao aplicativo

Agora você está pronto para adicionar o código necessário para importar as bibliotecas SDK necessárias, estabelecer uma conexão autenticada com seu projeto implantado e enviar perguntas.

1. Observe que a pasta **qna-app** contém um arquivo de código para o aplicativo cliente:

    - **C#**: Program.cs
    - **Python**: qna-app.py

    Abra o arquivo de código e, na parte superior, sob as referências de namespace existentes, localize o comentário **Importar namespaces**. Neste comentário, adicione o seguinte código específico do idioma para importar os namespaces necessários para usar o SDK de Análise de Texto:

    **C#**: Programas.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.Language.QuestionAnswering;
    ```

    **Python**: qna-app.py

    ```python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.language.questionanswering import QuestionAnsweringClient
    ```

1. Na função **Principal**, observe que o código para carregar o ponto de extremidade de serviço e a chave de Linguagem de IA do Azure do arquivo de configuração já foi fornecido. Em seguida, localize o comentário **Criar cliente usando ponto de extremidade e chave** e adicione o seguinte código para criar um cliente para a API de Análise de Texto:

    **C#**: Programas.cs

    ```C#
    // Create client using endpoint and key
    AzureKeyCredential credentials = new AzureKeyCredential(aiSvcKey);
    Uri endpoint = new Uri(aiSvcEndpoint);
    QuestionAnsweringClient aiClient = new QuestionAnsweringClient(endpoint, credentials);
    ```

    **Python**: qna-app.py

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(ai_key)
    ai_client = QuestionAnsweringClient(endpoint=ai_endpoint, credential=credential)
    ```

1. Na função **Principal**, localize o comentário **Enviar uma pergunta e exibir a resposta** e adicione o seguinte código para ler repetidamente perguntas da linha de comando, enviá-las ao serviço e exibir detalhes das respostas:

    **C#**: Programas.cs

    ```C#
    // Submit a question and display the answer
    string user_question = "";
    while (user_question.ToLower() != "quit")
        {
            Console.Write("Question: ");
            user_question = Console.ReadLine();
            QuestionAnsweringProject project = new QuestionAnsweringProject(projectName, deploymentName);
            Response<AnswersResult> response = aiClient.GetAnswers(user_question, project);
            foreach (KnowledgeBaseAnswer answer in response.Value.Answers)
            {
                Console.WriteLine(answer.Answer);
                Console.WriteLine($"Confidence: {answer.Confidence:P2}");
                Console.WriteLine($"Source: {answer.Source}");
                Console.WriteLine();
            }
        }
    ```

    **Python**: qna-app.py

    ```Python
    # Submit a question and display the answer
    user_question = ''
    while user_question.lower() != 'quit':
        user_question = input('\nQuestion:\n')
        response = ai_client.get_answers(question=user_question,
                                        project_name=ai_project_name,
                                        deployment_name=ai_deployment_name)
        for candidate in response.answers:
            print(candidate.answer)
            print("Confidence: {}".format(candidate.confidence))
            print("Source: {}".format(candidate.source))
    ```

1. Salve suas alterações e retorne ao terminal integrado para a pasta **qna-app** e digite o seguinte comando para executar o programa:

    - **C#**: `dotnet run`
    - **Python**: `python qna-app.py`

    > **Dica**: Você pode usar o ícone **Maximizar o tamanho do painel** (**^**) na barra de ferramentas do terminal para ver mais do texto do console.

1. Quando solicitado, insira uma pergunta a ser enviada ao seu projeto de respostas às perguntas; por exemplo `What is a learning path?`.
1. Analise a resposta retornada.
1. Faça mais perguntas. Quando terminar, digite `quit`.

## Limpar os recursos

Dica: se você tiver concluído a exploração do serviço de Linguagem de IA do Azure, exclua os recursos criados no exercício. Este é o procedimento:

1. Abra o portal do Azure em `https://portal.azure.com` e entre usando a conta Microsoft associada à sua assinatura do Azure.
2. Navegue até o recurso de Linguagem de IA do Azure que você criou neste laboratório.
3. Na página do recurso, selecione **Excluir** e siga as instruções para excluir o recurso.

## Mais informações

Para obter mais informações sobre respostas às perguntas na Linguagem de IA do Azure, consulte a [documentação de Linguagem de IA do Azure](https://learn.microsoft.com/azure/ai-services/language-service/question-answering/overview).
