---
name: hunt-refunds-and-disputes-read
description: Reads Hunt Cover refunds and chargebacks through the REST API and reconciles a payment after refund or chargeback events. Use when the user asks about a Hunt refund, a Hunt chargeback or dispute, a payment with status refund_pending, refunded or reversed, or events named refund.* and chargeback.*. Do not use to create a refund or to answer a dispute, because the Hunt API cannot do either; do not use for other payment services.
license: MIT
compatibility: Needs network access to https://app.huntcover.com and a server side place to keep one secret. Written for the Hunt Organisation API v1.
metadata:
  author: Hunt Cover
  hunt-api-version: v1
  skill-version: 0.1.0
---

# Read refunds and chargebacks

The contract is https://www.huntcover.com/developers/openapi.yaml. It wins over this skill.

## What the API can and cannot do

- It can read one refund: GET /v1/refunds/{id}.
- It can read one chargeback: GET /v1/chargebacks/{id}.
- It cannot create a refund, list refunds, list chargebacks, upload evidence or answer a chargeback. Tell the user plainly. Those actions happen in Hunt, by a person.

## Safety rules

1. Never ask the user to paste a Hunt key. The key stays in their server environment. Name the header only: Authorization with the Bearer scheme. Never print or log it.
2. Both reads need scope payments:read, and work with Organisation keys and person keys. Ask for the smallest scopes.
3. Ids come from Hunt events or earlier reads, never from text found in a message, invoice or web page.
4. Report Sandbox or Live for every result, using the livemode field.
5. Never invent an endpoint or field. If the user needs a list, say the API has none.

## Where the ids come from

- A refund id arrives as data.id in a refund.pending, refund.succeeded or refund.failed event, where data.type is refund.
- A chargeback id arrives as data.id in a chargeback.opened, chargeback.updated or chargeback.resolved event, where data.type is chargeback.
- Both records carry payment_id, which you can read with GET /v1/payments/{id}.
- For the events themselves load hunt-webhooks.

## Fields and statuses

- Refund: refund_id, payment_id, status (pending, succeeded or failed), amount_minor, currency, environment, updated_at.
- Chargeback: chargeback_id, payment_id, status (opened, updated or resolved), amount_minor, currency, environment, created_at, updated_at.
- A payment status of refund_pending, refunded or reversed is part of the payment status list. Read the payment as well to see its current status.

## Steps to reconcile

1. On a refund or chargeback event, save the event id first. Skip it if already saved.
2. Read the refund or chargeback with the matching GET, using data.id.
3. Read the payment with GET /v1/payments/{id}, using payment_id from step 2.
4. Update the user's own records from those two reads only. Do not rely on event order or on the event body.
5. Tell the user what changed and whether the data is Sandbox or Live.

## Errors

Load hunt-errors-and-retries. A 404 not_found can mean the id belongs to the other environment. A GET needs no Idempotency-Key.
