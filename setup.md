# Configurar o MCP do Atlas — instruções para o agente

Você é um agente de IA ajudando uma pessoa a conectar o Atlas da Sancon. Siga os passos na ordem e
pare no primeiro que falhar, dizendo à pessoa o que aconteceu.

**Servidor:** `https://app-sancon-platform-prod-brs.azurewebsites.net/mcp`
**Login:** OAuth com PKCE no Keycloak da Sancon, sem segredo de cliente.

| Cliente | ID do cliente OAuth |
| --- | --- |
| Claude Code, Codex | `mcp-local` |
| claude.ai, Claude Desktop, Cowork | `mcp-claude` |
| ChatGPT | `mcp-chatgpt` |

## 1. Adicionar o servidor

- **Claude Code:**
  `claude mcp add --transport http atlas https://app-sancon-platform-prod-brs.azurewebsites.net/mcp --client-id mcp-local`
- **Codex:**
  `codex mcp add atlas --url https://app-sancon-platform-prod-brs.azurewebsites.net/mcp --oauth-client-id mcp-local`
- **Outro cliente:** peça à pessoa para seguir a tela **Atlas › Conectar IA** da plataforma, que tem
  o passo a passo de claude.ai, Cowork e ChatGPT.

## 2. Autenticar

- **Claude Code:** `/mcp`, escolha `atlas` e **Authenticate**.
- **Codex:** `codex mcp login atlas`.

O navegador abre o login da Sancon. A pessoa entra com a conta dela na plataforma. Quem atende mais
de uma empresa tem uma conta por empresa: para trocar, desconecte e entre com a outra conta.

## 3. Confirmar

Chame a ferramenta de leitura `list_environments`. A resposta lista os Ambientes (ERP ou HCM) da
empresa conectada.

## 4. Informar

Diga à pessoa:

- qual empresa está conectada: é a primeira linha das instruções do servidor ("Empresa conectada: …");
- quais Ambientes apareceram;
- que ela pode pedir um Painel, e que publicar é feito por ela na tela do Atlas.

## Se falhar

| Sintoma | Causa |
| --- | --- |
| Login recusado com "Invalid scopes" ou "client" | ID do cliente errado para este cliente de IA |
| Conectado, mas sem ferramentas | Conta sem o papel Atlas Construtor ou Visualizador, ou empresa sem o Atlas contratado |
| 401 depois do login | Conta em mais de uma empresa; use a conta da empresa certa |
