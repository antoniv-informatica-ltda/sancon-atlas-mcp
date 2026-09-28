# Atlas — MCP da Sancon

Conecte o seu agente de IA ao **Atlas**, da plataforma Sancon: o dicionário de dados do ERP e do HCM
Senior da sua empresa e a construção de Painéis sobre o dado vivo. O agente monta o Rascunho do
Painel; quem publica é uma pessoa, na tela do Atlas.

- **Servidor MCP:** `https://app-sancon-platform-prod-brs.azurewebsites.net/mcp`
- **Login:** a sua conta da plataforma Sancon (OAuth com PKCE). Sem chave de API.
- **Quem pode usar:** conta com papel Atlas Visualizador (dicionário) ou Atlas Construtor (Rascunho e
  dado vivo), em empresa com o Atlas contratado.

O passo a passo de cada cliente, com botão e comando prontos, está na plataforma em
**Atlas › Conectar IA**. Para o agente se configurar sozinho, peça a ele para ler o
[`setup.md`](./setup.md).

## Claude Code

```
/plugin marketplace add antoniv-informatica-ltda/sancon-atlas-mcp
/plugin install atlas@sancon
```

O plugin instala o servidor (com o ID do cliente `mcp-local`) e a skill `construir-painel`. Depois,
`/mcp` → `atlas` → **Authenticate**.

## Codex

```
codex mcp add atlas --url https://app-sancon-platform-prod-brs.azurewebsites.net/mcp --oauth-client-id mcp-local
codex mcp login atlas
codex plugin marketplace add antoniv-informatica-ltda/sancon-atlas-mcp
```

O plugin do Codex traz a skill; o ID do cliente vai no `codex mcp add`, porque o formato de plugin
do Codex não tem campo de OAuth.

## claude.ai, Claude Desktop, celular e Cowork

Conector personalizado com a URL do servidor e, em **Use seu próprio cliente OAuth**, o ID
`mcp-claude`, sem segredo. Em Team e Enterprise, só o Owner adiciona; cada pessoa entra com a
própria conta.

## ChatGPT

Modo desenvolvedor → **Plugins** → **+**, com a URL do servidor e autenticação **OAuth**. Em
**Advanced OAuth settings**: método **User-Defined OAuth Client**, ID `mcp-chatgpt`, segredo vazio
e **Token endpoint auth method = `none`**. Em Business e Enterprise, quem cria é o admin do workspace.

## O que tem aqui

| Caminho | Para quê |
| --- | --- |
| `plugins/atlas/skills/construir-painel/` | A skill: o fluxo de montar um Painel com as ferramentas do Atlas |
| `plugins/atlas/.claude-plugin/`, `plugins/atlas/.mcp.json` | Plugin do Claude Code e do Cowork |
| `plugins/atlas/plugin.json`, `plugins/atlas/mcp.json` | Plugin do Codex (Agent Plugins 1.0.0) |
| `.claude-plugin/marketplace.json`, `.agents/plugins/marketplace.json` | Este repositório como marketplace |
| `setup.md` | Instruções para o agente configurar a conexão |
