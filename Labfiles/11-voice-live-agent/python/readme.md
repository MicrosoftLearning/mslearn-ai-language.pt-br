# Requisitos

## Executar no Cloud Shell

* Assinatura do Azure com acesso ao OpenAI
* Se estiver em execução no Azure Cloud Shell, escolha o shell Bash. A CLI do Azure e o Azure Developer CLI estão incluídas no Cloud Shell.

## Executar localmente

* Você pode executar o aplicativo Web localmente depois de executar o script de implantação:
    * [Azure Developer CLI (azd)](https://learn.microsoft.com/en-us/azure/developer/azure-developer-cli/install-azd)
    * [CLI do Azure](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)
    * Assinatura do Azure com acesso ao OpenAI


## Variáveis de ambiente

O  arquivo `.env` é criado pelo script *azdeploy.sh*. O ponto de extremidade do modelo de IA, a chave de API e o nome do modelo são adicionados durante a implantação dos recursos.

## Implantação de recursos do Azure

O `azdeploy.sh` fornecido cria os recursos necessários no Azure:

* Altere as duas variáveis na parte superior do script para corresponder às suas necessidades. Não altere mais nada.
* O script:
    * Implanta o modelo *gpt-4o* usando o AZD.
    * Cria o serviço de Registro de Contêiner do Azure
    * Usa Tarefas do ACR para criar e implantar a imagem do Dockerfile no ACR
    * Cria o Plano do Serviço de Aplicativo
    * Cria o aplicativo Web do Serviço de Aplicativo
    * Configura o aplicativo web para usar a imagem de contêiner no ACR
    * Configura as variáveis de ambiente do aplicativo Web
    * O script fornecerá o ponto de extremidade do Serviço de Aplicativo

O script fornece duas opções de implantação: 1. Implantação completa; e 2. Reimplantar somente a imagem. A opção 2 é usada apenas após a implantação, quando você quiser testar alterações no aplicativo. 

> Observação: Você pode executar o script no PowerShell ou no Bash usando o comando `bash azdeploy.sh`, que também permite executá-lo no Bash sem precisar torná-lo executável.

## Desenvolvimento local

### Provisionar o modelo de IA no Azure

Você pode executar o projeto localmente e provisionar apenas o modelo de IA seguindo estas etapas:

1. **Inicializar o ambiente** (escolha um nome descritivo):

   ```bash
   azd env new gpt-realtime-lab --confirm
   # or: azd env new your-name-gpt-experiment --confirm
   ```
   
   **Importante**: Esse nome fará parte dos nomes dos seus recursos no Azure!  
   O sinalizador `--confirm` define este como seu ambiente padrão sem solicitação.

1. **Defina o grupo de recursos**:

   ```bash
   azd env set AZURE_RESOURCE_GROUP "rg-your-name-gpt"
   ```

1. **Fazer logon e provisionar recursos de IA**:

   ```bash
   az login
   azd provision
   ```

    > **Importante**: NÃO execute `azd deploy` – o aplicativo não está configurado nos modelos do AZD.

Se você apenas provisionou o modelo usando o método `azd provision`, você DEVE criar um arquivo `.env` na raiz do diretório com as seguintes entradas:

```
AZURE_VOICE_LIVE_ENDPOINT=""
AZURE_VOICE_LIVE_API_KEY=""
VOICE_LIVE_MODEL=""
VOICE_LIVE_VOICE="en-US-JennyNeural"
VOICE_LIVE_INSTRUCTIONS="You are a helpful AI assistant with a focus on world history. Respond naturally and conversationally. Keep your responses concise but engaging."
VOICE_LIVE_VERBOSE="" #Suppresses excessive logging to the terminal if running locally
```

Observações:

1. O ponto de extremidade é o ponto de extremidade do modelo e deve incluir apenas `https://<proj-name>.cognitiveservices.azure.com`.
1. A chave de API é a chave do modelo.
1. O modelo é o nome do modelo usado durante a implantação.
1. Você pode recuperar esses valores no portal da Fábrica de IA.

### Executar o projeto localmente

O projeto foi criado e gerenciado usando o **uv**, mas não é necessário para execução. 

Se você tiver o **uv** instalado:

* Execute `uv venv` para criar o ambiente
* Execute `uv sync` para adicionar pacotes
* Alias criado para aplicativo Web: `uv run web` para iniciar o script `flask_app.py`.
* Arquivo requirements.txt criado com `uv pip compile pyproject.toml -o requirements.txt`

Se você não tiver o **uv** instalado:

* Criar o ambiente: `python -m venv .venv`
* Ativar o ambiente: `.\.venv\Scripts\Activate.ps1`
* Instalar dependências: `pip install -r requirements.txt`
* Executar o aplicativo (da raiz do projeto): `python .\src\real_time_voice\flask_app.py`
