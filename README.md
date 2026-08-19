# Nomupay Payout MCP Server

A [Model Context Protocol](https://modelcontextprotocol.io/) server for the
**Nomupay Payout API**.

The server runs locally and signs every request with your own key ID and
EC P-256 private key. The private key never leaves your machine, and there is
no hosted intermediary between your AI client and the Nomupay Payout API.

This README covers running the server from the published npm package.

## Credentials

You need three values from Nomupay support:

- `NOMUPAY_KID` — your key ID (KSUID), issued against your EC P-256 public key
- `NOMUPAY_PRIVATE_KEY` — your EC P-256 private key PEM (escaped as `\n` in
  single-line form). PKCS8 (`BEGIN PRIVATE KEY`) and SEC1
  (`BEGIN EC PRIVATE KEY`) are both accepted.
- `NOMUPAY_ACCOUNT_ID` — your root entity ID (`EID-...`)

To generate a key pair, send Nomupay support the **public** key and keep the
private key:

```bash
openssl ecparam -name prime256v1 -genkey -noout -out private-key.pem
openssl ec -in private-key.pem -pubout -out public-key.pem   # send this one to Nomupay
```

Prefer a dedicated key pair for the MCP server so it can be revoked
independently of your production integration.

Optional:

- `NOMUPAY_BASE_URL` — defaults to sandbox
  (`https://payout-api.sandbox.nomupay.com`); live is
  `https://payout-api.nomupay.com`. Point at live only when you mean it.
- `NOMUPAY_PRIVATE_KEY_PATH` — path to a `.pem` file, as an alternative to
  `NOMUPAY_PRIVATE_KEY`.

The key must keep its line breaks. In `NOMUPAY_PRIVATE_KEY` write them as
literal `\n` (one line: `-----BEGIN ...-----\nMHcC...\n-----END ...-----`).
If you see `DECODER routines::unsupported`, the PEM lost its line breaks —
the header, body, and footer were pasted on one line with spaces or nothing
between them. Re-paste with `\n` separators, or use
`NOMUPAY_PRIVATE_KEY_PATH` to point at the `.pem` file instead.

## Prerequisite: Configure npm authentication

This package is hosted on GitHub Packages. Authentication is required even
when the package is public.

Create a GitHub token with `read:packages` permission, then configure npm in
your user profile (`~/.npmrc`):

```text
registry=https://registry.npmjs.org/
@nomupay-labs:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken=${GITHUB_PAT}
```

Set `GITHUB_PAT` in your environment before using npm:

```bash
export GITHUB_PAT=your-github-token
```

Keep the token private and do not commit `.npmrc` to a repository.

## Install and run from package

Package name:

- `@nomupay-labs/nomupay-payout-mcp`

Current registry for this package:

- GitHub Packages (`https://npm.pkg.github.com`)

Install:

```bash
npm install @nomupay-labs/nomupay-payout-mcp
```

Run directly without adding to dependencies:

```bash
npx @nomupay-labs/nomupay-payout-mcp
```

Reference: [GitHub Docs: Working with the npm registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-npm-registry)

## VS Code MCP configuration

Create or update `.vscode/mcp.json`:

```json
{
  "inputs": [
    {
      "id": "nomupay-kid",
      "type": "promptString",
      "description": "Nomupay Payout Key ID (KSUID, 27 chars)"
    },
    {
      "id": "nomupay-private-key",
      "type": "promptString",
      "description": "Nomupay EC P-256 private key, full PEM with \\n-escaped newlines",
      "password": true
    },
    {
      "id": "nomupay-account-id",
      "type": "promptString",
      "description": "Nomupay root entity id (EID-...)"
    }
  ],
  "servers": {
    "nomupay-payout-mcp": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@nomupay-labs/nomupay-payout-mcp"],
      "env": {
        "NOMUPAY_KID": "${input:nomupay-kid}",
        "NOMUPAY_PRIVATE_KEY": "${input:nomupay-private-key}",
        "NOMUPAY_ACCOUNT_ID": "${input:nomupay-account-id}",
        "NOMUPAY_BASE_URL": "https://payout-api.sandbox.nomupay.com"
      }
    }
  }
}
```

## Claude Code configuration

Add the server with `claude mcp add`, passing credentials with `--env` and
separating the server command with `--`:

```bash
claude mcp add --env NOMUPAY_KID=your-kid \
  --env NOMUPAY_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\nMIGH...\n-----END PRIVATE KEY-----" \
  --env NOMUPAY_ACCOUNT_ID=EID-your-root-entity \
  --env NOMUPAY_BASE_URL=https://payout-api.sandbox.nomupay.com \
  --transport stdio nomupay-payout-mcp \
  -- npx -y @nomupay-labs/nomupay-payout-mcp
```

By default this adds the server at local scope (visible only to you, in the
current project). Add `--scope project` to share it with your team through a
committed `.mcp.json` file, or `--scope user` to make it available across all
your projects.

Alternatively, create or update a project-scoped `.mcp.json` directly:

```json
{
  "mcpServers": {
    "nomupay-payout-mcp": {
      "command": "npx",
      "args": ["-y", "@nomupay-labs/nomupay-payout-mcp"],
      "env": {
        "NOMUPAY_KID": "your-kid",
        "NOMUPAY_PRIVATE_KEY": "-----BEGIN PRIVATE KEY-----\nMIGH...\n-----END PRIVATE KEY-----",
        "NOMUPAY_ACCOUNT_ID": "EID-your-root-entity",
        "NOMUPAY_BASE_URL": "https://payout-api.sandbox.nomupay.com"
      }
    }
  }
}
```

Verify the server connected:

```bash
claude mcp get nomupay-payout-mcp
```

Committing `.mcp.json` with real credentials is not recommended. Prefer the
`claude mcp add` command per machine, or use environment variable expansion
(`${NOMUPAY_KID}`, etc.) in a committed `.mcp.json` and export the actual
secrets in your shell.

## Cursor configuration

Create or update `~/.cursor/mcp.json` (global) or `.cursor/mcp.json`
(per project):

```json
{
  "mcpServers": {
    "nomupay-payout-mcp": {
      "command": "npx",
      "args": ["-y", "@nomupay-labs/nomupay-payout-mcp"],
      "env": {
        "NOMUPAY_KID": "your-kid",
        "NOMUPAY_PRIVATE_KEY": "-----BEGIN PRIVATE KEY-----\nMIGH...\n-----END PRIVATE KEY-----",
        "NOMUPAY_ACCOUNT_ID": "EID-your-root-entity",
        "NOMUPAY_BASE_URL": "https://payout-api.sandbox.nomupay.com"
      }
    }
  }
}
```

Then enable the server under Cursor Settings → MCP.

## GitHub Copilot CLI configuration

Add the server with `copilot mcp add`, which writes to
`~/.copilot/mcp-config.json`:

```bash
copilot mcp add nomupay-payout-mcp \
  --type local \
  --env NOMUPAY_KID=your-kid \
  --env NOMUPAY_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\nMIGH...\n-----END PRIVATE KEY-----" \
  --env NOMUPAY_ACCOUNT_ID=EID-your-root-entity \
  --env NOMUPAY_BASE_URL=https://payout-api.sandbox.nomupay.com \
  --tools "*" \
  -- npx -y @nomupay-labs/nomupay-payout-mcp
```

Alternatively, edit `~/.copilot/mcp-config.json` directly:

```json
{
  "mcpServers": {
    "nomupay-payout-mcp": {
      "type": "local",
      "command": "npx",
      "args": ["-y", "@nomupay-labs/nomupay-payout-mcp"],
      "tools": ["*"],
      "env": {
        "NOMUPAY_KID": "your-kid",
        "NOMUPAY_PRIVATE_KEY": "-----BEGIN PRIVATE KEY-----\nMIGH...\n-----END PRIVATE KEY-----",
        "NOMUPAY_ACCOUNT_ID": "EID-your-root-entity",
        "NOMUPAY_BASE_URL": "https://payout-api.sandbox.nomupay.com"
      }
    }
  }
}
```

To share this configuration with a team instead, put the same `mcpServers`
block in a project-level `.mcp.json` (or `.github/mcp.json`) at the repository
root. Copilot CLI loads workspace MCP servers automatically once you trust the
folder.

Verify the server and its tools:

```bash
copilot mcp list
copilot mcp get nomupay-payout-mcp
```

## Tools reference

This section lists all tools currently exposed by the MCP server.

| Tool                            | Method | Endpoint                                          | Notes                                   |
| ------------------------------- | ------ | ------------------------------------------------- | --------------------------------------- |
| `payout_list_accounts`          | GET    | `/v1alpha1/accounts/{eid}`                        | Child accounts of an entity             |
| `payout_get_balance`            | GET    | `/v1alpha1/balances/{eid}`                        | Available balances, optional currency   |
| `payout_list_payouts`           | GET    | `/v1alpha1/receipts/account/{eid}`                | Payouts and fundings; status and dates  |
| `payout_get_payment`            | GET    | `/v1alpha1/payments/{id}`                         | One payment (`PMT-`) by id              |
| `payout_explain_status`         | —      | static catalog, no API call                       | Explains statuses and webhook codes     |
| `payout_create_sub_account`     | POST   | `/v1alpha1/sub-account`                           | Onboards a sub-account (`SID-`)         |
| `payout_create_transfer_method` | POST   | `/v1alpha1/sub-account/{sid}/transfer-method`     | Onboards a payout destination (`TRM-`)  |
| `payout_create_quote`           | POST   | `/v1alpha1/payment-quote`                         | FX rate and fee quote; no money moves   |
| `payout_validate_payment`       | POST   | `/v1alpha1/payments/validate`                     | Dry-run a payout; no money moves        |
| `payout_create_payment`         | POST   | `/v1alpha1/payments`                              | **Moves money**                         |

Every tool that takes an `eid` or `sourceId` defaults to `NOMUPAY_ACCOUNT_ID`
when it is omitted.

### Minimal parameter guide

#### Query tools

| Tool                    | Required      | Optional                                          |
| ----------------------- | ------------- | ------------------------------------------------- |
| `payout_list_accounts`  | -             | `eid`, `limit`, `page`                            |
| `payout_get_balance`    | -             | `eid`, `currency`                                 |
| `payout_list_payouts`   | -             | `eid`, `status`, `date_from`, `date_to`, `limit`, `page` |
| `payout_get_payment`    | `paymentId`   | -                                                 |
| `payout_explain_status` | `code`        | -                                                 |

#### Onboarding tools

| Tool                            | Required                                    | Optional                                             |
| ------------------------------- | ------------------------------------------- | ---------------------------------------------------- |
| `payout_create_sub_account`     | `clientSubAccountId`, `profile`             | `accountId`                                          |
| `payout_create_transfer_method` | `sid`, `countryCode`, `currencyCode`, `type` | `displayName`, `bankAccount`, `eWallet`, `profile`  |

`profile`, `bankAccount` and `eWallet` are corridor-specific objects and are
passed through to the API as given. See the
[Nomupay Payouts documentation](https://docs.nomupay.com/payouts) for the
fields required in each corridor.

#### Quote and payment tools

| Tool                      | Required                                                                   | Optional                                                       |
| ------------------------- | -------------------------------------------------------------------------- | -------------------------------------------------------------- |
| `payout_create_quote`     | `sourceCurrencyCode`, `destinationCurrencyCode`, `amount` (or `payoutAmount` + `payoutCurrency`) | `sourceId`, `includeFee`, `fxPolicy`     |
| `payout_validate_payment` | `destinationId`, `paymentReference`, `amount`, `currencyCode`, `purpose`   | `sourceId`, `description`, `internalMemo`, `releaseOn`, `expireOn` |
| `payout_create_payment`   | `destinationId`, `paymentReference`, `amount`, `currencyCode`, `purpose`   | `sourceId`, `description`, `internalMemo`, `releaseOn`, `expireOn` |

The quoted FX rate from `payout_create_quote` is applied to a subsequent
payout on the same source→destination currency pair.

## Money-movement policy

`payout_create_payment` creates a real payout instruction and **moves money**.
It uses `NOMUPAY_BASE_URL`, which defaults to sandbox; only the live URL moves
real funds.

- Call `payout_validate_payment` first, and review the result, before calling
  `payout_create_payment`.
- `payout_create_sub_account` and `payout_create_transfer_method` create real
  records but do not move money.
- `payout_create_quote` and all `GET` tools are read-only.

Payment statuses: `PENDING → PROCESSING → PROCESSED → DELIVERED`, with exits
`FAILED`, `RETURNED` and `CANCELLED`. `PROCESSED` means the instruction was
sent to the partner; only `DELIVERED` confirms the recipient was paid. Failed
or returned payouts are remediated with a new instruction, never by retrying
the same `PMT-` id.

## References

- [Nomupay Payouts documentation](https://docs.nomupay.com/payouts)
- [Payout statuses](https://docs.nomupay.com/payouts/response-codes/payment-statuses)
  and [webhook reason codes](https://docs.nomupay.com/payouts/response-codes/webhook-reason-codes)
- [Request signing](https://docs.nomupay.com/payouts/integration/security)
- [MCP Specification](https://modelcontextprotocol.io/)
