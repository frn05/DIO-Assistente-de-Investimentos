# Assistente de Investimentos com RPA e IA Generativa

Projeto desenvolvido para o desafio da DIO: extração de clientes com Python, processamento via webhook no n8n, cruzamento com uma base CSV e geração de mensagens personalizadas por perfil.

> **Importante:** este projeto é educacional. Os clientes e dados são fictícios. As mensagens são exemplos informativos, não recomendações financeiras individualizadas nem promessa de rentabilidade. Não envie dados reais nem credenciais para o repositório.

## Arquitetura

1. **Página de clientes (`docs/index.html`)** — tabela fictícia de clientes.
2. **RPA (`src/extrair_clientes.py`)** — lê a tabela com `requests` e `BeautifulSoup`, normaliza os dados e envia um JSON para o webhook do n8n.
3. **Base de investimentos (`docs/data.csv`)** — opções demonstrativas por perfil.
4. **Workflow (`n8n/workflow.json`)** — recebe o lote no Webhook, cruza os perfis com opções compatíveis, cria mensagens e devolve o resultado em JSON.
5. **Modo de teste** — o script pode salvar os dados localmente sem chamar o webhook; o envio de e-mail não é ativado por padrão.

## Estrutura

```text
.
├── README.md
├── requirements.txt
├── .env.example
├── docs/
│   ├── index.html
│   └── data.csv
├── src/
│   └── extrair_clientes.py
├── n8n/
│   └── workflow.json
└── tests/
    └── test_recomendacoes.py
```

## Como executar localmente

### 1. Pré-requisitos

- Python 3.10+
- Uma instância do n8n local ou n8n Cloud
- Git (opcional)

### 2. Instalar dependências

```bash
python -m venv .venv
# Windows:
.venv\Scripts\activate
# macOS/Linux:
source .venv/bin/activate

pip install -r requirements.txt
```

### 3. Executar o RPA sem webhook

Por padrão, o script usa a página pública do repositório-base da DIO. Para testar usando a página local, informe a URL correspondente.

```bash
python src/extrair_clientes.py --clientes-url "https://digitalinnovationone.github.io/dio-lab-assistente-investimentos-rpa-n8n/"
```

O script salva `output/clientes_extraidos.json`. Para apontar a URL de webhook do n8n:

```bash
# Windows PowerShell:
$env:N8N_WEBHOOK_URL="http://localhost:5678/webhook/assistente-investimentos"
python src/extrair_clientes.py --enviar

# macOS/Linux:
export N8N_WEBHOOK_URL="http://localhost:5678/webhook/assistente-investimentos"
python src/extrair_clientes.py --enviar
```

Use `--enviar` somente depois de importar e ativar o workflow no n8n. Para uma instância n8n remota, substitua a URL local pela URL de produção do seu webhook.

## Importar o workflow no n8n

1. Abra o n8n.
2. Vá em **Workflows → Import from File**.
3. Selecione `n8n/workflow.json`.
4. Salve o workflow.
5. Ative-o e copie a **Production URL** do nó Webhook.
6. Configure essa URL em `N8N_WEBHOOK_URL` e execute o script com `--enviar`.

O workflow utiliza um CSV hospedado no GitHub. Depois de publicar este projeto, abra `n8n/workflow.json` e substitua `SEU_USUARIO/SEU_REPOSITORIO` no nó **HTTP Request - Buscar CSV** pelo endereço real do repositório.

### Credenciais obrigatórias do n8n

- **Google Gemini API:** crie/seleciona uma credencial Google Gemini API no nó `Google Gemini - Message a model`.
- **SMTP:** configure o nó `Send an Email - Enviar mensagem` com servidor, porta, usuário, senha/app password e remetente autorizado.
- Faça o primeiro teste enviando para seu próprio endereço e com os dados fictícios. O fluxo envia e-mail por cliente quando estiver ativo.

O endpoint espera um POST JSON no formato:

```json
{
  "clientes": [
    {
      "nome": "Ana Silva",
      "email": "ana@email.com",
      "saldo": 12500.0,
      "perfil": "Conservador"
    }
  ]
}
```

A resposta do webhook confirma o processamento ao script chamador. As mensagens geradas pelo Gemini são enviadas por e-mail usando SMTP. O envio real só funciona depois que você configurar as credenciais no n8n.

## Integração com IA generativa

O workflow importável inclui o nó **Google Gemini — Message a model**. Configure a credencial Gemini e SMTP no seu n8n antes de ativar o fluxo. Como as credenciais são específicas da sua conta, elas não podem ser embutidas no JSON. O prompt orienta o modelo a usar apenas os produtos encontrados no CSV e a não prometer rentabilidade.

## Testes

```bash
python -m unittest discover -s tests -v
```

## Limitações e decisões técnicas

- A extração usa seletores CSS estáveis (`#clientes tbody tr` e classes das células).
- O parser de saldo converte valores brasileiros, como `R$ 12.500,00`, para `12500.0`.
- A lógica de cruzamento considera perfil e saldo mínimo, mas não calcula suitability regulatória nem avalia objetivos, prazo ou tolerância real a perdas.
- Os percentuais e categorias do CSV são dados demonstrativos fornecidos para o exercício; não representam taxas atuais.
- O workflow contém envio automático de e-mail; mantenha-o inativo até configurar e testar SMTP com um destinatário controlado.
- Prints e imagens de execução devem ser adicionados manualmente após testar o fluxo no seu ambiente.

## Entrega DIO

Antes de enviar o link na plataforma:
- [ ] Importar e testar o workflow no n8n.
- [ ] Executar o RPA em modo local e, depois, com webhook.
- [ ] Adicionar prints próprios da execução.
- [ ] Confirmar que nenhum token, senha ou dado real foi incluído no repositório.
