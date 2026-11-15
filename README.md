# Fin API playground

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/SaaSGun/fin-api-playground/blob/main/playground.ipynb)

A guided, runnable tour of two live APIs from one portfolio project. Each
step calls the deployed services and shows what they guarantee, including
under concurrency.

- **[fin-order-management-api](https://github.com/SaaSGun/fin-order-management-api)**:
  orders, pre-trade compliance, fills and settlement on PostgreSQL.
- **[fin-analytics-api](https://github.com/SaaSGun/fin-analytics-api)**:
  holdings, sector exposures, performance and ESG from BigQuery.

## What you'll see

| # | Scenario | What it shows |
|---|---|---|
| 1 | Place an order | Cash is reserved in the same transaction; every compliance rule's result is stored |
| 2 | Restricted instrument | A compliance rejection is a `201` with the reason, not an error |
| 3 | Retry the same request | Idempotency keys: one order from ten simultaneous copies; `409` on a changed body |
| 4 | Ten PMs, one limit | Ten simultaneous 20-share orders against a 50-share limit: exactly two admitted |
| 5 | Ten cancels, one order | `ETag` / `If-Match`: one `200`, nine `412`; `428` without `If-Match` |
| 6 | Fill the order | Order, cash, position and settlement change in one commit |
| 7 | Settle it | Settlements are versioned too |
| 8 | Burst of 25 requests | Per-key rate limit: `429` + `Retry-After` |
| 9 | Clean up | Cancels what's still open and releases its cash |
| 10–15 | Analytics | Security profile, holdings, sector tilts, performance, cursor pagination, ESG, result caching, auth errors |

## Running it

1. Open the notebook in Colab with the badge above. Nothing needs to be
   installed.
2. Use the three values from your invitation email: `OMS_API_KEY`,
   `OMS_CLIENT_ID` and `ANALYTICS_API_KEY`. Add them as Colab secrets (🔑
   in the left sidebar), or paste each one when prompted.
3. **Runtime → Run all.** It takes a couple of minutes.

The order-management scenarios run on an account of your own, so you can
run them as often as you like without affecting anyone else. The keys
expire on their own after a week.

To run it locally instead: `pip install httpx notebook`, set the three
values as environment variables, and open `playground.ipynb` in Jupyter.

## Onboarding a reviewer (for the owner)

The first command runs in fin-order-management-api, with `DATABASE_URL`
pointing at Cloud SQL (as in its `loadtest/run_production.sh`). The second
runs in fin-analytics-api, with BigQuery write access.

```bash
python -m scripts.manage_reviewers create --name alice --days 7
python scripts/manage_api_keys.py create --client-name reviewer-alice \
    --scopes standard:read,confidential:read --days 7
```

The first prints the OMS client id and key. It creates a client
`reviewer-alice` whose account is funded like the demo account (EUR
100,000, USD 20,000, a 50-share AAPL limit). The second prints the
analytics key. Both keys stop working after seven days.

- **Reset** a reviewer's account:
  `python -m scripts.manage_reviewers reset --name alice`. This cancels
  their open orders, closes the account and opens a fresh one, which the
  notebook finds by itself.
- **Revoke** before expiry:
  `python -m scripts.manage_api_keys revoke --client-name reviewer-alice`
  in fin-order-management-api, and
  `python scripts/manage_api_keys.py revoke --client-name reviewer-alice`
  in fin-analytics-api.

A message to send:

> Here's access to the Fin API playground:
> https://github.com/SaaSGun/fin-api-playground. Open the notebook in
> Colab, then add these as secrets or paste them when asked:
>
> - `OMS_API_KEY`: …
> - `OMS_CLIENT_ID`: …
> - `ANALYTICS_API_KEY`: …
>
> The keys expire on {date}.
