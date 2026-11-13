# Hunt Cover agent skills

Repository: https://github.com/Huntgroup-crypto/hunt-pay-skills

Skills that teach AI coding agents how to integrate Hunt Cover for a user: accept payments, receive signed webhooks, start Send and Withdraw requests that a person approves, read refunds and chargebacks, and connect to Hunt's read only MCP server.

The Hunt OpenAPI contract is the source of truth: https://www.huntcover.com/developers/openapi.yaml. Developer guides are at https://www.huntcover.com/developers. The page for these skills is https://www.huntcover.com/developers/skills. If a skill and the contract disagree, the contract wins.

Version 0.1.0 contains no scripts. Each skill is plain text.

This repository holds only what lets AI agents install and use Hunt. Nothing else is published here.

## What is included

| Skill | Use it for |
| --- | --- |
| hunt-overview | Entry point. Routes to the right skill and sets the shared rules. |
| hunt-accept-payments | Payment Links, hosted Checkout and payment reads in Sandbox. |
| hunt-webhooks | Verifying signed events and managing webhook endpoints. |
| hunt-send-and-withdraw | Building Send and Withdraw requests. A person approves in Hunt. |
| hunt-errors-and-retries | Error codes, idempotency, retries and rate limits. |
| hunt-mcp-connect | Connecting an AI client to the six read only Hunt tools. |
| hunt-refunds-and-disputes-read | Reading refunds and chargebacks and reconciling. |

## Safety model

- Sandbox first. Live only when the user says so in writing.
- Secrets stay with the user. The skills tell the agent to ask the user to set the key in their own environment. They never ask an agent to hold, print or log a key or a signing secret.
- A person approves every money movement. A Send or Withdraw create answers with pending human action and a link into Hunt. The person signs in to Hunt and confirms there under Hunt's normal controls. An API key cannot approve. The skills tell the agent to hand over the link and stop.
- The skills say plainly what the Hunt API cannot do instead of guessing.

## Install

Install the skills from this repository only. Do not use a copy from anywhere else.

Claude Code, in your terminal:

```
claude plugin marketplace add Huntgroup-crypto/hunt-pay-skills
claude plugin install hunt-cover@hunt-cover-plugins
```

Codex, in your terminal:

```
codex plugin marketplace add Huntgroup-crypto/hunt-pay-skills
```

Restart the app, open the Plugins Directory, choose hunt-cover-plugins and install hunt-cover.

As a fallback, copy the skills folder of this repository into .agents/skills in your project.

Cursor, from the root folder of your project:

```
git clone https://github.com/Huntgroup-crypto/hunt-pay-skills.git
mkdir -p .cursor/skills
cp -R hunt-pay-skills/skills/. .cursor/skills/
```

Any client that reads agent skills from a folder: clone this repository and copy the folders inside skills into the skills folder of your project. Cursor reads .cursor/skills and .agents/skills in a project. Codex reads .agents/skills in a project.

## Trust

Use only the copy in this repository. See TRADEMARKS.md. A copy from anywhere else may have been changed.

## Report a security problem

Do not open a public issue for a security problem. Use private vulnerability reporting on this repository, or write to info@huntcover.com. See SECURITY.md.

## Licence

MIT. See LICENSE. The licence does not cover the Hunt name or logo. Copyright Hunt Financials Limited.
