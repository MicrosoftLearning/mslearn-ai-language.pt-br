---
lab:
  title: Analisar o texto
  description: 'Use a Linguagem de IA do Azure para analisar textos, incluindo detecção de idioma, análise de sentimento, extração de palavras-chave e reconhecimento de entidades.'
---

# Analisar o texto

A **Linguagem de IA do Azure** oferece suporte à análise de texto, incluindo detecção de idioma, análise de sentimento, extração de palavras-chave e reconhecimento de entidades.

Por exemplo, suponha que uma agência de viagens queira processar avaliações de hotéis que foram enviadas ao site da empresa. Usando a Linguagem de IA do Azure, eles podem determinar o idioma em que cada avaliação está escrita, o sentimento (positivo, neutro ou negativo) das avaliações, frases-chave que podem indicar os principais tópicos discutidos na avaliação e entidades nomeadas, como lugares, pontos de referência ou pessoas mencionadas. Neste exercício, você usará o SDK de Linguagem de IA do Azure para Python para análise de texto e implementará um aplicativo simples de avaliação de hotéis baseado neste exemplo.

Embora este exercício seja baseado em Python, você pode desenvolver aplicativos de análise de texto usando vários SDKs específicos da linguagem, incluindo:

- [Biblioteca de clientes de Análise de Texto de IA do Azure para Python](https://pypi.org/project/azure-ai-textanalytics/)
- [Biblioteca de clientes de Análise de Texto de IA do Azure para .NET](https://www.nuget.org/packages/Azure.AI.TextAnalytics)
- [Biblioteca de clientes de Análise de Texto de IA do Azure para JavaScript](https://www.npmjs.com/package/@azure/ai-text-analytics)

Este exercício levará aproximadamente **30** minutos.

## Provisionar um recurso de *Linguagem de IA do Azure*

Caso ainda não tenha um na sua assinatura, provisione um recurso do **serviço de Linguagem de IA do Azure** na sua assinatura do Azure.

1. Abra o portal do Azure em `https://portal.azure.com` e entre usando a conta Microsoft associada à sua assinatura do Azure.
1. Selecione **Criar um recurso**.
1. No campo de pesquisa, pesquise **Serviço de linguagem**. Nos resultados, selecione **Criar** em **Serviço de Linguagem**.
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
1. Veja a página **Chaves e Ponto de extremidade** na seção **Gerenciamento de Recursos**. Você precisará das informações nesta página mais adiante no exercício.

## Clone o repositório para este curso

Você desenvolverá seu código usando o Cloud Shell no Portal do Azure. Os arquivos de código do seu aplicativo foram fornecidos em um repositório do GitHub.

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
    cd mslearn-ai-language/Labfiles/01-analyze-text/Python/text-analysis
    ```

## Configurar seu aplicativo

1. No painel de linha de comando, execute o seguinte comando para exibir os arquivos de código na pasta **text-analysis**:

    ```
   ls -a -l
    ```

    Os arquivos incluem um arquivo de configuração (**.env**) e um arquivo de código (**text-analysis.py**). O texto que seu aplicativo analisará está na subpasta **reviews**.

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

1. Atualize os valores de configuração para incluir o **ponto de extremidade** e uma **chave** do recurso de Linguagem do Azure que você criou (disponível na página **Chaves e Ponto de Extremidade** do seu recurso de Linguagem de IA do Azure no portal do Azure)
1. Depois de substituir os espaços reservados, use o comando **CTRL+S** ou **botão direito do mouse > Salvar** para salvar as suas alterações e, em seguida, use o comando **CTRL+Q** ou **botão direito do mouse > Sair** para fechar o editor de código, mantendo a linha de comando do Cloud Shell aberta.

## Adicionar código para se conectar ao recurso de Linguagem de IA do Azure

1. Insira o seguinte comando para editar o arquivo de código do aplicativo:

    ```
    code text-analysis.py
    ```

1. Revise o código existente. Você adicionará código para trabalhar com o SDK de Análise de Texto da Linguagem de IA.

    > **Dica**: ao adicionar código ao arquivo, certifique-se de manter o recuo correto.

1. No topo do arquivo de código, abaixo das referências de namespace existentes, localize o comentário **Import namespaces** e adicione o seguinte código para importar os namespaces necessários para usar o SDK de Análise de Texto:

    ```python
   # import namespaces
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.textanalytics import TextAnalyticsClient
    ```

1. Na função **main**, observe que o código para carregar o ponto de extremidade e a chave do serviço de Linguagem de IA do Azure a partir do arquivo de configuração já foi fornecido. Em seguida, localize o comentário **Criar cliente usando ponto de extremidade e chave** e adicione o seguinte código para criar um cliente para a API de Análise de Texto:

    ```Python
   # Create client using endpoint and key
   credential = AzureKeyCredential(ai_key)
   ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

1. Salve suas alterações (CTRL+S) e, em seguida, insira o seguinte comando para executar o programa (você maximiza o painel do Cloud Shell e redimensiona os painéis para ver mais texto no painel de linha de comando):

    ```
   python text-analysis.py
    ```

1. Observe a saída, pois o código deve ser executado sem erros, exibindo o conteúdo de cada arquivo de texto de avaliação na pasta **comentários** O aplicativo cria com êxito um cliente para a API de Análise de Texto, mas não o utiliza. Corrigiremos isso na próxima seção.

## Adicionar código para detectar linguagem

Agora que você criou um cliente para a API, vamos usá-lo para detectar o idioma no qual cada avaliação foi escrita.

1. No editor de código, localize o comentário **Get language**. Em seguida, adicione o código necessário para detectar o idioma em cada documento de revisão:

    ```python
   # Get language
   detectedLanguage = ai_client.detect_language(documents=[text])[0]
   print('\nLanguage: {}'.format(detectedLanguage.primary_language.name))
    ```

     > **Observação**: *neste exemplo, cada avaliação é analisada individualmente, o que resulta em uma chamada separada ao serviço para cada arquivo. Uma abordagem alternativa é criar uma coleção de documentos e transmiti-los para o serviço em uma única chamada. Em ambas as abordagens, a resposta do serviço consiste em uma coleção de documentos; é por isso que no código Python acima, o índice do primeiro (e único) documento na resposta ([0]) é especificado.*

1. Salve suas alterações. Então, execute novamente o programa.
1. Observe a saída, que desta vez identifica a linguagem para cada avaliação.

## Adicionar código para avaliar o sentimento

A *análise de sentimento* é uma técnica comumente usada para classificar o texto como *positivo* ou *negativo* (ou possivelmente *neutro* ou *misto*). É comumente usada para analisar postagens de mídia social, avaliações de produtos e outros itens em que o sentimento do texto pode fornecer insights úteis.

1. No editor de código, localize o comentário **Get sentiment**. Em seguida, adicione o código necessário para detectar o sentimento em cada documento de revisão:

    ```python
   # Get sentiment
   sentimentAnalysis = ai_client.analyze_sentiment(documents=[text])[0]
   print("\nSentiment: {}".format(sentimentAnalysis.sentiment))
    ```

1. Salve suas alterações. Em seguida, feche o editor de código e execute novamente o programa.
1. Observe a saída, que agora identifica o sentimento das avaliações.

## Adicionar código para identificar frases-chave

Pode ser útil identificar frases-chave no corpo do texto para ajudar a determinar os principais tópicos discutidos.

1. No editor de código, localize o comentário **Get key phrases**. Em seguida, adicione o código necessário para detectar frases-chave em cada documento de revisão:

    ```python
   # Get key phrases
   phrases = ai_client.extract_key_phrases(documents=[text])[0].key_phrases
   if len(phrases) > 0:
        print("\nKey Phrases:")
        for phrase in phrases:
            print('\t{}'.format(phrase))
    ```

1. Salve suas alterações e execute o programa novamente.
1. Observe a saída, cada documento contém frases-chave que fornecem alguns insights sobre o objeto da avaliação.

## Adicionar código para extrair entidades

Muitas vezes, documentos ou outros corpos de texto mencionam pessoas, lugares, períodos de tempo ou outras entidades. A API de Análise de Texto pode detectar várias categorias (e subcategorias) de entidades em seu texto.

1. No editor de código, localize o comentário **Get entities**. Em seguida, adicione o código necessário para identificar entidades mencionadas em cada revisão:

    ```python
   # Get entities
   entities = ai_client.recognize_entities(documents=[text])[0].entities
   if len(entities) > 0:
        print("\nEntities")
        for entity in entities:
            print('\t{} ({})'.format(entity.text, entity.category))
    ```

1. Salve suas alterações e execute o programa novamente.
1. Observe a saída, que identifica as entidades que foram detectadas no texto.

## Adicionar código para extrair entidades vinculadas

Além das entidades categorizadas, a API de Análise de Texto pode detectar entidades para as quais há links conhecidos para fontes de dados, como a Wikipédia.

1. No editor de código, localize o comentário **Get linked entities**. Em seguida, adicione o código necessário para identificar entidades vinculadas mencionadas em cada revisão:

    ```python
   # Get linked entities
   entities = ai_client.recognize_linked_entities(documents=[text])[0].entities
   if len(entities) > 0:
        print("\nLinks")
        for linked_entity in entities:
            print('\t{} ({})'.format(linked_entity.name, linked_entity.url))
    ```

1. Salve suas alterações e execute o programa novamente.
1. Observe a saída, que identifica as entidades vinculadas que foram detectadas.

## Limpar os recursos

Dica: se você tiver concluído a exploração do serviço de Linguagem de IA do Azure, exclua os recursos criados no exercício. Este é o procedimento:

1. Feche o painel do Azure Cloud Shell
1. No portal do Azure, navegue até o recurso de Linguagem de IA do Azure criado neste laboratório.
1. Na página de recursos, selecione **Excluir** e siga as instruções para excluir o recurso.

## Mais informações

Para obter mais informações sobre como usar a **Linguagem de IA do Azure**, consulte a [documentação](https://learn.microsoft.com/azure/ai-services/language-service/).
