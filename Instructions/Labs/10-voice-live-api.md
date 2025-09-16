---
lab:
  title: Explorar a API do Voice Live
  description: Saiba como usar e personalizar a API do Voice Live disponível no Playground da Fábrica de IA do Azure.
---

# Explorar a API do Voice Live

Neste exercício, você irá criar um agente na Fábrica de IA do Azure e explorar a API do Voice Live no Playground de Fala. 

Este exercício levará aproximadamente **30** minutos para ser concluído.

> <span style="color:red">**Observação**:</span> Algumas das tecnologias usadas neste exercício estão atualmente em versão prévia ou em desenvolvimento ativo. Você pode experimentar algum comportamento inesperado, avisos ou erros.

> <span style="color:red">**Observação**:</span> Este exercício foi projetado para ser concluído em um ambiente de navegador com acesso direto ao microfone do computador. Embora os conceitos possam ser explorados no Azure Cloud Shell, os recursos interativos de voz exigem acesso ao hardware de áudio local.

## Criar um projeto da Fábrica de IA do Azure

Vamos começar criando um projeto da Fábrica de IA do Azure.

1. Em um navegador da Web, abra o [Portal da Fábrica de IA do Azure](https://ai.azure.com) em `https://ai.azure.com` e entre usando suas credenciais do Azure. Feche todas as dicas ou painéis de início rápido abertos na primeira vez que você entrar e, se necessário, use o logotipo da **Fábrica de IA do Azure** no canto superior esquerdo para navegar até a home page, que é semelhante à imagem a seguir (feche o painel **Ajuda** se estiver aberto):

    ![Captura de tela da home page da Fábrica de IA do Azure com "Criar um projeto" selecionado.](../media/ai-foundry-new-home-page.png)

1. Na home page, clique em **Criar um agente**.

1. No assistente **Criar um agente**, insira um nome válido para o projeto. 

1. Clique em **Opções avançadas** e especifique as seguintes configurações:
    - **Recurso da Fábrica de IA do Azure**: *manter o nome padrão*
    - **Assinatura**: *sua assinatura do Azure*
    - **Grupo de recursos**: *criar ou selecionar um grupo de recursos*
    - **Região**: Selecione aleatoriamente uma região nas seguintes opções:\*
        - Leste dos EUA 2
        - Suécia Central

    > \* No momento da gravação, a API do Voice Live só tem suporte nas regiões listadas anteriormente. Selecionar uma localização aleatoriamente ajuda a garantir que uma única região não fique sobrecarregada com tráfego e proporciona uma experiência mais fluida. Caso os limites de serviço sejam atingidos, talvez seja necessário criar outro projeto em uma região diferente.

1. Clique em **Criar** e revise a configuração. Aguarde a conclusão do processo de configuração.

    >**Observação**: se você receber um erro de permissões, clique no botão **Corrigir** para adicionar as permissões apropriadas para continuar.

1. Quando o projeto for criado, o playground de Agentes abrirá por padrão no portal da Fábrica de IA do Azure, que será semelhante à imagem a seguir:

    ![Captura de tela dos detalhes de um projeto IA do Azure no Portal da Fábrica de IA do Azure.](../media/ai-foundry-project-2.png)

## Iniciar uma amostra do Voice Live

 Nesta seção do exercício, você irá interagir com um dos agentes. 

1. Selecione **Playgrounds** no painel de navegação.

1. Localize o grupo **Playground de Fala** e selecione o botão **Experimentar o playground de Fala**.

1. O Playground de Fala oferece muitas opções predefinidas. Use a barra de rolagem horizontal para navegar até o final da lista e selecionar o bloco do **Voice Live**. 

    ![Captura de tela do bloco do Voice Live.](../media/voice-live-tile.png)

1. Selecione a amostra do agente de **Chat casual** no painel **Experimentar com amostras**.

1. Verifique se o microfone e os alto-falantes estão funcionando e selecione o botão **Iniciar** na parte inferior da página. 

    Ao interagir com o agente, observe que você pode interrompê-lo, e ele fará uma pausa para ouvir. Tente falar com diferentes durações de pausa entre palavras e frases. Observe com que rapidez o agente reconhece as pausas e preenche a conversa. Quando terminar, selecione o botão **Encerrar**.

1. Inicie os outros agentes de amostra para explorar como eles se comportam.

    À medida que você explora os diferentes agentes, observe as alterações na  seção **Instrução de resposta** no painel **Configuração**.

## Configurar o agente 

Nesta seção, você irá alterar a voz do agente e adicionar um avatar ao agente de **Chat casual**. O painel **Configuração** é dividido em três seções: **GenAI**, **Fala** e **Avatar**.

>**Observação:** Se você alterar ou interagir com qualquer uma das opções de configuração, será preciso selecionar o botão **Aplicar** na parte inferior do painel **Configuração** para habilitar o agente.

Selecione o agente de **Chat casual**. Em seguida, altere a voz do agente e adicione um avatar, com as seguintes instruções:

1. Selecione **> Fala** para expandir a seção e acessar as opções.

1. Selecione o menu suspenso na opção **Voz** e escolha uma voz diferente.

1. Selecione **Aplicar** para salvar suas alterações e, em seguida **Iniciar** para iniciar o agente e ouvir a alteração.

    Repita as etapas anteriores para tentar algumas vozes diferentes. Prossiga para a próxima etapa quando terminar a seleção de voz.

1. Selecione **> Avatar** para expandir a seção e acessar as opções.

1. Selecione o botão de alternância para habilitar o recurso e escolha um dos avatares. 

1. Selecione **Aplicar** para salvar suas alterações e, em seguida **Iniciar** para iniciar o agente. 

    Observe a animação e a sincronização do avatar com o áudio.

1. Expanda a seção **> GenAI** e desative a alternância **Participação proativa**. Em seguida, selecione **Aplicar** para salvar suas alterações e, então **Iniciar** para iniciar o agente.

    Com a **Participação proativa** desativada, o agente não inicia a conversa. Pergunte ao agente : "Você pode me dizer o que você faz?" para iniciar a conversa.

>**Dica:** Você pode selecionar **Redefinir para padrão** e, em seguida, **Aplicar** para retornar o agente ao seu comportamento padrão.

Quando terminar, prossiga para a próxima seção.

## Criar um agente de voz

Nesta seção, você criará seu próprio agente de voz do zero.

1. Selecione **Iniciar do zero** na seção **Experimentar com o seu próprio** do painel. 

1. Expanda a seção **> GenAI** do painel **Configuração**.

1. Selecione o menu suspenso **Modelo de IA generativa** e escolha o modelo **GPT-4o Mini Realtime**.

1. Adicione o texto a seguir na seção **Instrução de resposta**.

    ```
    You are a voice agent named "Ava" who acts as a friendly car rental agent. 
    ```

1. Defina o controle deslizante **Temperatura da resposta** para o valor **0,8**. 

1. Ative a alternância **Participação proativa**.

1. Selecione **Aplicar** para salvar suas alterações e, em seguida **Iniciar** para iniciar o agente.

    O agente se apresentará e perguntará como ele pode ajudá-lo hoje. Pergunte ao agente : "Você tem algum sedan disponível para aluguel na quinta-feira?" Observe quanto tempo o agente leva para responder. Faça outras perguntas ao agente para ver como ele responde. Quando terminar, prossiga para a próxima etapa.

1. Expanda a seção **> Fala** do painel **Configuração**.

1. Defina o botão de alternância **Fim da fala (EOU)** para a posição **ativa**.

1. Defina o botão de alternância **Aprimoramento de áudio** para a posição **ativa**.

1. Selecione **Aplicar** para salvar suas alterações e, em seguida **Iniciar** para iniciar o agente.

    Depois que o agente se apresentar, pergunte:"Você tem algum avião para alugar?" Observe que o agente responde mais rapidamente do que antes depois de terminar sua pergunta. A configuração **Fim da fala (EOU)** faz com que o agente detecte pausas e o final da sua fala com base no contexto e na semântica. Isso permite que ele tenha uma conversa mais natural.

Quando terminar, prossiga para a próxima seção.

## Limpar os recursos

Agora que você concluiu o exercício, exclua o projeto criado para evitar o uso desnecessário de recursos.

1. Selecione **Centro de gerenciamento** no menu de navegação da Fábrica de IA.
1. Selecione **Excluir projeto** no painel de informações à direita e confirme a exclusão.

