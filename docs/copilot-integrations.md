# Copilot integrations

This setup uses official Copilot CLI and MCP surfaces. GitHub Copilot and
Microsoft 365 Copilot aren't Cursor or n8n model providers, and no unsupported
VS Code Copilot extension is installed in Cursor.

## Current status

### GitHub Copilot CLI

- Installed version: `1.0.110`
- Signed-in account: `serdarsarar-creator`
- License check: successful model response
- Read-only repository test: successful
- Exact-path write test on a temporary branch: successful and cleaned up

Start it from Cursor's integrated terminal:

```bash
cd ~/Projects/self-hosted-ai-starter-kit
copilot
```

Do not use `--allow-all`, `--allow-all-tools`, or `--yolo`. For unattended
prompts, allow only the exact read, write, URL, or shell pattern needed for
that one run.

### Microsoft Work IQ MCP

- Endpoint: `https://workiq.svc.cloud.microsoft/mcp`
- Cursor configuration: `~/.cursor/mcp.json`
- Network and OAuth metadata discovery: successful
- Authentication result: blocked with `invalid_client`
- The server remains configured but should stay disabled until the tenant
  configuration below is complete.

The attempted public client ID wasn't accepted for the organization tenant
and Cursor's loopback redirect. A tenant administrator must:

1. Provision or confirm the Work IQ service principal with application ID
   `fdcc1f02-fc51-4226-8753-f668596af7f7`.
2. Enable Work IQ and its usage-based billing/Copilot Credits policy.
3. Create a single-tenant Entra public-client app for Cursor Work IQ MCP.
4. Add desktop redirect URI `http://localhost:8787/callback`.
5. Enable public-client flows; don't create or distribute a client secret.
6. Add delegated permission `WorkIQAgent.Ask`
   (`0b1715fd-f4bf-4c63-b16d-5be31f9847c2`) and grant admin consent.
7. Assign the intended user the required Microsoft 365 Copilot license,
   billing policy, and Work IQ access.
8. Replace `auth.CLIENT_ID` in `~/.cursor/mcp.json` with the approved app's
   client ID, then run:

```bash
cursor agent mcp login workiq
cursor agent mcp list-tools workiq
```

Start with read-only tools. Enable `WorkIQ.Act` only after separate admin
consent and retain per-action approval for mail, calendar, Teams, SharePoint,
and file changes.

### Microsoft Copilot Studio MCP

- Package: `@microsoft/copilot-studio-mcp@1.0.4`
- Corporate sign-in: successful
- Explicitly selected environment type: `Sandbox`
- Selected environment region: Germany West Central
- Sign-in and environment discovery: successful
- Agent inventory result: `notAccessible`
- No agent, workflow, connection, test, or publication was created

An administrator must add the user to the environment's Entra security group
and assign the Dataverse **Environment Maker** role, or a custom role with the
required Copilot and Copilot Subcomponent table privileges. Copilot Studio
author licensing or pay-as-you-go author access must also be enabled.

After access is granted, verify with read-only operations first:

```bash
cursor agent mcp list-tools copilot-studio
```

Use `discover_resources` and inventory actions before authoring. Save drafts
only after explicit approval. Testing, publishing, deployment, connection
setup, sharing, and deletion are separate approval boundaries.

## Cursor MCP configuration

The existing `chatpwc-local` server must remain configured. The added servers
contain no tokens or client secrets:

- `workiq`: remote HTTPS MCP with Entra OAuth
- `copilot-studio`: pinned Microsoft stdio package

Cursor stores OAuth credentials separately from `mcp.json`. Never paste token
values, authorization codes, or connector credentials into the repository or
chat.

## n8n integration boundary

GitHub Copilot isn't exposed as an OpenAI-compatible API. The current n8n
Assistant remains on local `gpt-oss-20b-n8n`.

A future n8n-to-Copilot bridge would require a dedicated fine-grained GitHub
token with **Copilot Requests** permission or another approved service
credential. The existing interactive OAuth token stays in the macOS keychain
and must not be copied into Docker. Until a dedicated credential is approved,
use Copilot CLI only from the host's Cursor terminal.

Any future bridge must:

- listen only on loopback or an authenticated internal transport;
- expose no host-network port to other devices;
- keep credentials outside Git and container images;
- require per-request tool and write approval;
- avoid logging prompts, company content, or model responses;
- present Copilot as a separate agent, never as a Cursor/n8n model provider.

## Removal

To disable a server without deleting configuration:

```bash
cursor agent mcp disable workiq
cursor agent mcp disable copilot-studio
```

To remove the integrations completely, restore the latest protected
`~/.cursor/mcp.json.backup-*` file and remove this project's Copilot rule.
Uninstall GitHub Copilot CLI with:

```bash
brew uninstall --cask copilot-cli
```
