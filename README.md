# Patch MCP plugin for Grok Bot

Give Grok a phone line. This plugin connects Grok Bot and Grok Build to [Patch MCP](https://www.patchmcp.com), a hosted MCP server that makes authorized phone calls from **your own verified phone number**. Give the voice agent a task and acceptable fallbacks, then review the actual outcome, transcript and recording when available. You can listen live and stop the call.

**Texting is coming soon.** SMS, picture messages, inbox access and automatic text monitoring are unavailable. The plugin does not start text listeners or ask you to enable SMS consent.

The package contains one remote HTTP MCP connection and one skill. Nothing runs locally. [Manu Labs, LLC](https://www.patchmcp.com/terms) operates the hosted service; its source is separate from this repository.

## Installation

The official marketplace submission is [PR #778](https://github.com/xai-org/plugin-marketplace/pull/778). Until it is approved and merged, add this repository through Grok's GitHub plugin installation flow:

```text
https://github.com/manu-labs/patch-mcp-grok-plugin
```

Other compatible MCP clients can connect directly to `https://www.patchmcp.com/mcp`. Public discovery exposes tool definitions; account tools require OAuth.

### First connection

1. Enter your number on Patch's verification screen. Caller IDs support US/Canada (+1), UK (+44), Brazil (+55) and Norway (+47), including countries and territories sharing those calling codes; the verification UI lists eligible countries.
2. Verify ownership with an SMS code or an automated verification call. Landlines can use voice verification.
3. Accept the Terms and Privacy Policy and approve the OAuth connection.

Your verified number identifies your account. No password or API key is needed. Verification codes used to sign in are separate from agent texting.

## Usage

```text
Call the restaurant at [phone number] and ask whether it has a table for two on October 15 at 7:30 p.m. local time. Any time between 7 and 8:30 works. Ask about availability only; don't make a reservation yet.
```

```text
Call the courier at [phone number] about order 48213 and ask for the delivery status and next step. Don't authorize any extra charges.
```

Supply the destination, task, dates, local times, names, authority to make commitments and acceptable fallbacks. Grok asks for missing details, checks the route with `estimate_call`, offers the returned `listen_url` and follows `get_call` until the call ends. It reports actual outcomes, including partial results and failed attempts.

### Current tools

| Tool | What it does |
| --- | --- |
| `get_phone_number` | Read your connected caller ID and verification status without calling or sending a code. |
| `estimate_call` | Check route availability, estimated one-minute usage and remaining allowance without dialing. |
| `place_call` | Start an authorized call with an objective, optional context, callee name, user name and locale. Returns a call ID and listen link. |
| `get_call` | Read status, resolution, transcript and recording when available. Supports `wait_seconds` up to 50. |
| `list_calls` | Read recent calls; pass `next_cursor` as `cursor` for older history. |
| `end_call` | Stop an active call. Do not retry a call the user stopped. |
| `set_call_visibility` | Make a specific call public or private only at your explicit request. |

Calls support English, Spanish, Portuguese, French, Italian, Russian, Polish, Ukrainian, Romanian, Dutch and German. Pass a supported locale such as `en-US`, `es-MX`, `pt-BR`, `fr` or `de`; the live tool schema lists every supported locale. Without one, Patch uses the account locale, then US English. The selected language stays fixed during the call. The account UI lets you choose a separate voice for each locale.

Custom introductions and conversation style belong in **Account → Call agent instructions**. There is no per-call instructions argument or private channel for adding instructions after dialing. `max_duration_minutes` is a result field, not a supported `place_call` argument.

## Calling limits and privacy

- Destinations cover Central America, Europe, North America and South America, subject to route availability. Use `estimate_call` instead of assuming every number is reachable. Caribbean destinations are excluded.
- Free calls require both your caller ID and the destination to be US/Canadian. Eligible accounts receive **five free calls, up to five minutes each**, counted only when someone picks up and speaks. Calls involving any other supported number require a paid plan from the first call.
- Paid calls consume a monthly allowance based on provider costs, including unsuccessful attempts. Usage can update after the call. Calls end when estimated live usage exhausts the allowance. Billing is in USD; see the [current pricing](https://www.patchmcp.com/pricing).
- One call runs per account at a time. Calls can be placed at any time; you are responsible for fair use. Emergency, premium-rate, blocked and opted-out numbers are refused. Objective screening rejects prohibited uses, including harassment, impersonation and bulk telemarketing.
- The default opening names the user and task, followed by the recording disclosure before the first business question. The voice agent is instructed to confirm it is AI when asked and end the call if the recipient declines. Do not request hidden AI identity or bypass screening.
- Calls are private by default. Publishing exposes the objective, phone numbers, transcript, result and audio to anyone with the permanent, token-free link, including live audio. Grok explains that exposure before publishing and never changes visibility just to share a result with you. Making a call private stops public access; existing signed links remain valid until they expire.

Cached text requests return `texts_coming_soon`; Grok relays the message without retrying or asking you to opt in. It also relays call policy and entitlement errors without repeated retries.

## Security and data flows

- **Network:** the plugin connects to `https://www.patchmcp.com/mcp`. OAuth discovery, registration, authorization and token exchange use the same host. The package declares no telemetry endpoint.
- **Authentication:** OAuth 2.1 with PKCE and dynamic client registration. The client stores the bearer token for Patch's existing `calls` scope and sends it to Patch. No API key, environment variables or local files need to be supplied. Public discovery does not grant calling access.
- **Local execution:** none. No commands, agents, hooks, scripts, binaries or stdio servers.
- **Data sent:** destinations, objectives, context, names, locales, call IDs, history cursors and explicit visibility choices. Patch stores account and call records, transcripts, results and recordings as described in its Privacy Policy. Do not include passwords, one-time codes, payment-card details, government identifiers or protected health information in call instructions.
- **Providers:** xAI screens objectives and extracts results; the configured voice engine uses xAI or ElevenLabs. Telnyx carries calls and verification and stores recordings. See [Who we share it with](https://www.patchmcp.com/privacy#share) for current details. The plugin's single host does not mean the service has no subprocessors.
- **Policies:** [Terms](https://www.patchmcp.com/terms) · [Privacy Policy](https://www.patchmcp.com/privacy) · [Support](mailto:leko@patchmcp.com).

## Removal

Remove `patch-mcp` from your Grok plugins. To revoke server access, open [your account](https://www.patchmcp.com/account) → **Connected apps** → **Disconnect**. History remains subject to the Privacy Policy. Recipients can use the public [do-not-call page](https://www.patchmcp.com/opt-out).

## Repository layout

```text
.grok-plugin/plugin.json   plugin manifest
.mcp.json                  hosted HTTP MCP connection
skills/patch-mcp/SKILL.md  calling workflow
LICENSE                   MIT
```

## License and support

This plugin's manifest, skill and documentation use the [MIT License](LICENSE). The service has its own Terms of Use. Report plugin issues on [GitHub](https://github.com/manu-labs/patch-mcp-grok-plugin/issues) or email [leko@patchmcp.com](mailto:leko@patchmcp.com).
