---
name: Write a Salesforce account plan with Poggio
description: >-
  Use the Poggio REST API v2 to create or refresh a Salesforce account, generate
  its intelligence pages, and write an AI-generated account plan back into
  Salesforce for one or many accounts.
api: https://poggio.io/docs/api/v2
base_url: https://api.poggio.io/v2
operations: [create_or_refresh_salesforce_account, trigger_account_pages, write_account_plan, batch_write_account_plan, batch_create_or_refresh_salesforce_accounts]
generated: '2026-07-20'
method: generated
---

# Write a Salesforce account plan with Poggio

The Poggio REST API v2 (`https://api.poggio.io/v2`) lets you drive account
intelligence and account-plan writeback into Salesforce/Agentforce workflows.
Authenticate with OAuth 2.0 (authorization code, client credentials, or refresh
token); the organization is inferred from the access token. A Salesforce
integration user must be registered first (`v2_register_salesforce_integration_user`).

## Steps

1. **Register the Salesforce connection** (one-time) with
   `v2_register_salesforce_integration_user` so Poggio can connect to your
   Salesforce org.
2. **Create or refresh the account.** Call `create_or_refresh_salesforce_account`
   for a single account, or `batch_create_or_refresh_salesforce_accounts` to
   process many in one request.
3. **Generate the intelligence pages.** Call `trigger_account_pages` to build
   the account's POVs, relationship maps, and plans.
4. **Write the plan back.** Call `write_account_plan` for one account or
   `batch_write_account_plan` to write/update multiple Salesforce account plans
   in a single request.

## Rules

- All endpoints require an OAuth Bearer access token
  (`authentication/poggio-labs-authentication.yml`).
- `v2_create_account` is upsert: it returns `201` when a new account is created
  and `200` when the account already exists — treat both as success.
- Use the batch operations to stay within rate limits when processing account lists.
- See `conventions/poggio-labs-conventions.yml` for versioning and discovery details.
