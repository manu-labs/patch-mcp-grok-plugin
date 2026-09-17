---
name: patch-mcp
description: |
  Make a real phone call on the user's behalf with the Patch MCP tools (place_call, get_call, list_calls, end_call). Use when the user wants someone phoned and the answer brought back: book a table or an appointment, ask a shop or clinic a question, confirm hours or stock, chase an order or a delivery, reschedule something, or reach a business that only answers the phone. Covers turning the request into an objective the voice agent can carry out, the one-time user_name step, waiting for the result, reading the resolution, and the limits: US and Canadian numbers only, calling hours at the destination, never for emergencies.
---

# Patch MCP: phone calls for Grok

Patch MCP places outbound calls **from the user's own verified number**. A real-time voice agent makes the call, opens by saying it is an AI calling on behalf of the user and that the call is recorded, handles phone menus and hold queues, and hangs up when the objective is met. You get back a structured resolution, the transcript and a recording.

## Before placing a call

Collect what the agent needs. Ask the user for anything missing rather than guessing.

- **Who to call.** `to` must be an E.164 number (`+14155551234`). If the user names a business, look the number up first and tell them which number you found.
- **What done looks like.** One clear outcome, with the specifics the callee will ask for and the fallbacks the user accepts.
- **Who the call is for.** The agent introduces itself as "calling on behalf of {user_name}". Patch keeps the name on the account after the first call; you only need to ask when `place_call` returns `user_name_required` (see below).

Unless the user already spelled out the number and the ask, confirm both in one line before dialing: it is a real call from their number, and it uses one of their free calls or paid minutes.

## Placing the call

```json
{
  "to": "+14155551234",
  "callee_name": "Zuni Café",
  "objective": "Book a table for 2 tonight at 19:30 under the name Sam. Any time between 19:00 and 20:30 is fine. If nothing is free, ask about tomorrow at the same time.",
  "context": "One guest is vegetarian. Callback number is the number this call comes from.",
  "max_duration_minutes": 5
}
```

Writing a good `objective`:

- Lead with the outcome, then the specifics (names, dates, times, quantities, order numbers), then the fallbacks ("if X is not possible, accept Y; otherwise just ask when it would be").
- Put facts the agent may reveal *only if asked* in `context`, not in the objective.
- One objective per call. Two unrelated errands are two calls.
- Leave out anything the user did not say. The agent will not invent details, so it cannot answer questions you did not brief it on.

`place_call` returns right away with a `call_id`, the applied `max_duration_minutes`, and a `listen_url`.

### `user_name_required`

If the account has no name on file, `place_call` is rejected with `user_name_required`. Ask the user what name the agent should give ("Sam", or "Sam at Acme"), then call `place_call` again with it in `user_name`. Never invent a name. It is saved to the account and not asked for again.

## While the call runs

- Give the user the `listen_url` straight away: it is a private page where they can hear the call live, read the transcript as it happens, and stop the call.
- Poll `get_call` with `wait_seconds: 50`. The server holds the request and returns on any status change, so a loop of long polls costs a handful of round trips. Calls typically take 2–5 minutes.
- Do not place a second call for the same errand while one is in progress. If the user changes their mind, call `end_call`: the agent wraps up politely and hangs up, and a partial resolution is still produced.

## Reading the result

Terminal statuses: `completed`, `no_answer`, `busy`, `voicemail`, `failed`, `canceled`, `declined_recording`, `opted_out`.

Read `resolution` first; it is the answer:

- `outcome`: `achieved`, `partially_achieved`, `not_achieved` or `no_conversation`
- `summary`: what happened, in a sentence or two
- `commitments[]`: what the callee agreed to (the booking, the refund, the callback)
- `follow_ups[]`: what the user still has to do
- `key_facts[]`: prices, hours, names, reference numbers the callee gave

Then report to the user in their terms: the commitment, the follow-ups, and anything surprising. Offer the transcript and the `recording_url` (the link expires after about an hour; `recording_status: processing` means it is still being prepared, so poll again). If the resolution lags the terminal status by a few seconds, poll once more before answering.

`ended_by: "user"` on a canceled call means the user stopped it from the listen page. Do not retry.

## Errors and limits

Rejections carry a machine-readable `error` and a message written for the user. Relay the message; do not retry blindly.

- **Calling hours.** Calls go out between 8:00 and 21:00 at the destination's local time. Outside that window the response says when calling reopens (`reset_at`); offer to place the call then.
- **Rate limits.** One call at a time per account and a small hourly cap; the response includes `reset_at` or retry guidance.
- **Plan limits.** New accounts get free calls capped at 3 minutes each; paid plans have a monthly pool of talk time. `subscription_required` and `plan_limit_reached` errors carry an `upgrade_url` or `manage_url`: a private, signed link to the user's account. Give the user that exact link rather than paraphrasing it.
- **Region.** US and Canadian numbers only. Premium-rate and other blocked ranges are refused.
- **Do-not-call.** Numbers on Patch's do-not-call list are refused for every user (`opted_out`). Never try to work around it.

## Rules

- Never use Patch for emergencies (911, poison control, crisis lines) or anything time-critical to someone's safety. Tell the user to call themselves.
- The agent always discloses that it is an AI and that the call is recorded. Do not ask it to pretend to be human, to impersonate anyone, or to hide the disclosure. If the callee does not want to talk to an AI, the agent ends the call.
- Only call numbers the user has a legitimate reason to reach. Patch is for the user's own errands, not outreach, marketing, polling or repeated calls to people who did not ask to be called.
- `list_calls` shows the user's recent calls with outcomes. Use it to recover a lost `call_id` or to answer "what happened with that call yesterday?" without dialing again.
