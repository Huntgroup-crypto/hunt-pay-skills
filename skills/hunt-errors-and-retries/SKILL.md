---
name: hunt-errors-and-retries
description: Explains how to read Hunt Cover API errors, retry safely, respect rate limits and recover from unknown outcomes with idempotency keys. Use when a Hunt API call returns an error, times out, returns 409, 429 or 503, when the user asks how to retry a Hunt request, or when writing retry code for the Hunt API. Do not use for errors from other services.
license: MIT
compatibility: Written for the Hunt Organisation API v1 at https://app.huntcover.com.
metadata:
  author: Hunt Cover
  hunt-api-version: v1
  skill-version: 0.1.0
---

# Hunt errors, retries and idempotency

The contract is https://www.huntcover.com/developers/openapi.yaml. Its error registry wins over this skill.

## Response shape

- Success: ok true, data, meta (request_id and server_time), api_version v1 and livemode.
- Error: ok false, error (code, message, retryable and request_id), api_version v1 and livemode. Before authentication, livemode is null.
- Every response has Cache-Control no-store and X-Request-Id. The request id is safe to share with Hunt support. Keys and full request bodies are not.
- Branch on HTTP status and error.code. Never branch on error.message.

## Idempotency

- Every POST and PATCH needs an Idempotency-Key header: a lowercase UUIDv7. The one exception is DELETE of a webhook endpoint, which takes none.
- Generate the key, save it with the exact request body, then send. Never generate a key after you sent.
- A key lives 72 hours from the time inside it and may be at most five minutes ahead of Hunt's clock. A valid request returns Idempotency-Key-Expires-At.
- After a timeout or an unknown outcome, send the same key and the same body. Never make a replacement money request because a reply is missing.
- Retry the same key at most 3 times in a row. This cap is this skill's rule, not an API limit. After that, read the resource and show the user the X-Request-Id.

## Error codes (public registry)

- invalid_request, 400: fix the fields. Use a new key only if the meaning of the command changed.
- invalid_idempotency_key, 400: use the original current lowercase UUIDv7 key. Check the system clock.
- unauthorized, 401: the key is wrong, revoked or not for this environment. Stop. Do not try other keys.
- access_denied, 403: the key lacks the scope, the operation is not allowed for this key kind, or access is restricted. Tell the user which scope the call needs. Do not widen scopes yourself. A person key on an Organisation only operation gets this code.
- access_denied, 503, retryable: restrictions could not be confirmed. Check the original outcome, then retry with the same key.
- not_found, 404: check the id and that you are in the right environment.
- request_timeout, 408: send the unchanged command with the same key once the body is complete.
- idempotency_key_expired, 409: read the original resource or give the user the request id. Do not make a replacement automatically.
- idempotency_conflict, 409: the key was used with a different body. Stop. Show the user both bodies. Do not send a third variant.
- idempotency_in_progress, 409: read the original resource before doing anything else.
- conflict, 409: read the current resource, then decide.
- attempt_pending, 409: a webhook attempt already exists. Wait.
- secret_already_revealed, 409: rotate the webhook signing secret.
- event_unavailable, 410: read the resource the event pointed at.
- request_too_large, 413: body is over 65,536 bytes.
- unsupported_media_type, 415: send uncompressed UTF-8 application/json for a non empty POST or PATCH.
- rate_limited, 429, retryable: wait the whole seconds in Retry-After, then retry with the same key and body.
- dependency_unavailable, 503, retryable: read the original operation first if you can, then retry with the same key.

## Rate limits

- Every request uses all applicable counters in a sliding one second window per Organisation and environment, per key and per operation family. The strictest exceeded limit wins.
- Read X-RateLimit-Limit, X-RateLimit-Remaining and X-RateLimit-Reset (Unix time) on responses.
- Restarting the process, changing IP or rotating the key does not reset the budget.
- The numbers are in the contract under x-rate-limits in the OpenAPI file at https://www.huntcover.com/developers/openapi.yaml. Do not copy them from here.
- Pause between status reads. Do not poll faster than the headers allow.

## Request limits

At most 65,536 bytes, five seconds to read the body, 16 levels of JSON nesting. Unknown fields, duplicate query parameters and an invalid currency are rejected.

## Money rule

A failure is never a reason to create a second money request. Read the first one. Only the user, in chat, can ask for a new request, and it gets a new key and a fresh yes.

## Steps when a call fails

1. Note the HTTP status, error.code and X-Request-Id. Never print the key.
2. Look the code up above and do exactly what it says.
3. If the outcome is unknown, read the resource before any retry.
4. Stop after 3 tries and report the request id to the user.
