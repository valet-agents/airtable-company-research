# Airtable Company Research

When a new account lands in your Airtable tracker, the agent researches the tech stack, recent funding, and key contacts — then posts a sourced dossier to your team in Slack. @mention it any time for a follow-up.

## Prerequisites
- An [Airtable](https://airtable.com) workspace with an Accounts (or Companies) table where you can authorize the Airtable MCP via OAuth
- A Slack workspace where you can install the agent's bot and invite it to one or more channels

<table>
  <tr>
    <td><strong>CHANNELS</strong></td>
    <td><code>slack</code> · <code>heartbeat</code> — daily</td>
  </tr>
  <tr>
    <td><strong>CONNECTORS</strong></td>
    <td><code>airtable-mcp</code> <code>parallel-search-mcp</code></td>
  </tr>
  <tr>
    <td colspan="2" align="center">
      <br />
      <a href="https://valet.dev/deploy?from=github.com/valet-agents/airtable-company-research">
        <img src="https://raw.githubusercontent.com/valet-agents/airtable-company-research/main/.github/deploy-button.svg" alt="Deploy Agent →" height="40" />
      </a>
      <br /><br />
    </td>
  </tr>
</table>
