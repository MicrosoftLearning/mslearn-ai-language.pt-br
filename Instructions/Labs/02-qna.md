---
lab:
  title: Criar uma solução de respostas às perguntas
  description: Use a Linguagem de IA do Azure para criar uma solução personalizada de respostas às perguntas.
---

# Criar uma solução de respostas às perguntas

Um dos cenários de conversação mais comuns é fornecer suporte por meio de uma base de dados de conhecimento de FAQs (perguntas frequentes). Muitas organizações publicam perguntas frequentes como documentos ou páginas da Web, o que funciona bem para um pequeno conjunto de pares de perguntas e respostas, mas documentos grandes podem ser difíceis e demorados de pesquisar.

A **Linguagem de IA do Azure** inclui uma capacidade de *respostas às perguntas* que permite criar uma base de dados de conhecimento de pares de perguntas e respostas que pode ser consultada usando entrada de linguagem natural e é mais comumente usada como um recurso que um bot pode usar para procurar respostas a perguntas enviadas pelos usuários. Neste exercício, você usará o SDK da Linguagem de IA do Azure para Python para análise de texto para implementar um aplicativo simples de respostas às perguntas.

Embora este exercício seja baseado em Python, você pode desenvolver aplicativos de respostas às perguntas usando vários SDKs específicos da linguagem, incluindo:

- [Biblioteca de clientes de Respostas às Perguntas do Serviço de Linguagem de IA do Azure para Python](https://pypi.org/project/azure-ai-language-questionanswering/)
- [Biblioteca de clientes de Respostas às Perguntas do Serviço de Linguagem de IA do Azure para .NET](https://www.nuget.org/packages/Azure.AI.Language.QuestionAnswering)

Este exercício levará aproximadamente **20** minutos.

## Provisionar um recurso da *Linguagem de IA do Azure*

Caso ainda não tenha um na sua assinatura, provisione um recurso de **serviço de Linguagem de IA do Azure**. Além disso, para criar e hospedar uma base de dados de conhecimento para respostas às perguntas, você precisa habilitar o recurso **Respostas às perguntas**.

1. Abra o portal do Azure em `https://portal.azure.com` e entre usando a conta Microsoft associada à sua assinatura do Azure.
1. Selecione **Criar um recurso**.
1. No campo de pesquisa, pesquise **Serviço de linguagem**. Nos resultados, selecione **Criar** em **Serviço de Linguagem**.
1. Clique no bloco **Respostas às perguntas personalizadas**. Em seguida, selecione **Continuar para criar o recurso**. Você precisará inserir as seguintes configurações:

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
1. Veja a página **Chaves e Ponto de extremidade** na seção **Gerenciamento de Recursos**. Você precisará das informações nesta página mais adiante no exercício.

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
1. No assistente ***Criar um projeto**, na página **Escolher configuração de linguagem**, selecione a opção **Escolher a linguagem para todos os projetos** e selecione **Inglês** como idioma. Em seguida, selecione **Avançar**.
1. Na página **Inserir informações básicas**, insira os seguintes detalhes:
    - **Nome**`LearnFAQ`
    - **Descrição**: `FAQ for Microsoft Learn`
    - **Resposta padrão quando nenhuma resposta é retornada**: `Sorry, I don't understand the question`
1. Selecione **Avançar**.
1. Na página **Revisar e concluir**, selecione **Criar projeto**.

## Adicionar fontes à base de dados de conhecimento

Você pode criar uma base de dados de conhecimento do zero, mas é comum começar importando perguntas e respostas de uma página ou documento de perguntas frequentes existente. Nesse caso, você importará dados de uma página da Web de perguntas frequentes para o Microsoft Learn e também importará algumas perguntas e respostas predefinidas de "bate-papo" para dar suporte a trocas conversacionais comuns.

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

1. No projeto **LearnFAQ** no Language Studio, selecione a página **Implantar base de dados de conhecimento** no menu de navegação à esquerda.
1. Na parte superior da página, selecione **Implantar**. Em seguida, selecione **Implantar** para confirmar que deseja implantar a base de dados de conhecimento.
1. Quando a implantação estiver concluída, selecione **Obter URL de previsão** para exibir o ponto de extremidade REST da sua base de dados de conhecimento e observe que a solicitação de amostra inclui parâmetros para:
    - **projectName**: o nome do seu projeto (que deve ser *LearnFAQ*)
    - **deploymentName**: o nome da sua implantação (que deve ser *produção*)
1. Feche a caixa de diálogo URL de previsão.

## Preparar-se para desenvolver um aplicativo no Cloud Shell

Você desenvolverá seu aplicativo de respostas às perguntas usando o Cloud Shell no portal do Azure. Os arquivos de código do seu aplicativo foram fornecidos em um repositório do GitHub.

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
    cd mslearn-ai-language/Labfiles/02-qna/Python/qna-app
    ```

## Configurar seu aplicativo

1. No painel de linha de comando, execute o seguinte comando para exibir os arquivos de código na pasta **qna-app**:

    ```
   ls -a -l
    ```

    Os arquivos incluem um arquivo de configuração (**.env**) e um arquivo de código (**qna-app.py**).

1. Crie um ambiente virtual Python e instale o pacote SDK de Respostas às Perguntas da Linguagem de IA do Azure e outros pacotes necessários executando o seguinte comando:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-language-questionanswering
    ```

1. Insira o seguinte comando para editar o arquivo de configuração:

    ```
    code .env
    ```

    O arquivo é aberto em um editor de código.

1. No arquivo de código, atualize os valores de configuração para refletir o **ponto de extremidade** e a **chave** de autenticação do recurso de Linguagem do Azure que você criou (disponível na página **Chaves e Ponto de extremidade** do seu recurso de Linguagem de IA do Azure no portal do Azure). O nome do projeto e o nome da implantação da base de dados de conhecimento implantada também devem estar nesse arquivo.
1. Depois de substituir os espaços reservados, use o comando **CTRL+S** ou **botão direito do mouse > Salvar** para salvar as suas alterações e, em seguida, use o comando **CTRL+Q** ou **botão direito do mouse > Sair** para fechar o editor de código, mantendo a linha de comando do Cloud Shell aberta.

## Adicionar código para usar sua base de dados de conhecimento

1. Insira o seguinte comando para editar o arquivo de código do aplicativo:

    ```
    code qna-app.py
    ```

1. Revise o código existente. Você adicionará código para trabalhar com sua base de dados de conhecimento.

    > **Dica**: ao adicionar código ao arquivo, certifique-se de manter o recuo correto.

1. No arquivo de código, localize o comentário **Import namespaces**. Em seguida, neste comentário, adicione o seguinte código específico de linguagem para importar os namespaces que você precisará para usar o SDK de respostas às perguntas:

    ```python
   # import namespaces
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.language.questionanswering import QuestionAnsweringClient
    ```

1. Na função **main**, observe que o código para carregar o ponto de extremidade e a chave do serviço de Linguagem de IA do Azure a partir do arquivo de configuração já foi fornecido. Em seguida, localize o comentário **Create client using endpoint and key** e adicione o seguinte código para criar um cliente de respostas às perguntas:

    ```Python
   # Create client using endpoint and key
   credential = AzureKeyCredential(ai_key)
   ai_client = QuestionAnsweringClient(endpoint=ai_endpoint, credential=credential)
    ```

1. No arquivo de código, localize o comentário **Submit a question and display the answer** e adicione o seguinte código para ler repetidamente perguntas da linha de comando, enviá-las ao serviço e exibir os detalhes das respostas:

    ```Python
   # Submit a question and display the answer
   user_question = ''
   while True:
        user_question = input('\nQuestion:\n')
        if user_question.lower() == "quit":                
            break
        response = ai_client.get_answers(question=user_question,
                                        project_name=ai_project_name,
                                        deployment_name=ai_deployment_name)
        for candidate in response.answers:
            print(candidate.answer)
            print("Confidence: {}".format(candidate.confidence))
            print("Source: {}".format(candidate.source))
    ```

1. Salve suas alterações (CTRL+S) e, em seguida, insira o seguinte comando para executar o programa (você maximiza o painel do Cloud Shell e redimensiona os painéis para ver mais texto no painel de linha de comando):

    ```
   python qna-app.py
    ```

1. Quando solicitado, insira uma pergunta a ser enviada ao seu projeto de respostas às perguntas; por exemplo `What is a learning path?`.
1. Analise a resposta retornada.
1. Faça mais perguntas. Quando terminar, digite `quit`.

## Limpar os recursos

Dica: se você tiver concluído a exploração do serviço de Linguagem de IA do Azure, exclua os recursos criados no exercício. Este é o procedimento:

1. Feche o painel do Azure Cloud Shell
1. No portal do Azure, navegue até o recurso de Linguagem de IA do Azure criado neste laboratório.
1. Na página de recursos, selecione **Excluir** e siga as instruções para excluir o recurso.

## Mais informações

Para obter mais informações sobre respostas às perguntas na Linguagem de IA do Azure, consulte a [documentação de Linguagem de IA do Azure](https://learn.microsoft.com/azure/ai-services/language-service/question-answering/overview).
