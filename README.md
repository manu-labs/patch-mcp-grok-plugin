# Patch MCP plugin for Grok Bot

Give Grok a phone line.

This plugin connects Grok Bot (and Grok Build) to [Patch MCP](https://www.patchmcp.com), a hosted MCP server that places real phone calls **from the user's own verified phone number**. A real-time voice agent makes the call, introduces itself as an AI calling on behalf of the user, gets the errand done, and comes back with a structured outcome, the transcript and a recording.

It bundles two things:

- **`.mcp.json`** – the hosted Patch MCP server at `https://www.patchmcp.com/mcp` (Streamable HTTP, OAuth). Nothing runs locally.
- **`skills/patch-mcp`** – a skill that teaches Grok how to brief the voice agent, wait for the call, and read the result.

The server itself is a hosted service operated by [Manu Labs, LLC](https://www.patchmcp.com/terms); its source is not part of this repository.

## Built on Grok

Patch MCP runs on xAI end to end. The voice on the call is Grok's [Voice Agent API](https://docs.x.ai/developers/model-capabilities/audio/voice-agent): real-time speech-to-speech over WebSocket, so there is no separate speech-to-text or text-to-speech hop, and the transcript of both sides comes out of the same session. A Grok text model screens every objective before the call is dialed and extracts the structured resolution once it ends. Telephony, recordings and the verification texts are carried by Telnyx.

## Installation

**Grok Bot** – open **Plugins → Marketplace**, search for **Patch MCP** and add it. Or add this repository directly as a plugin from GitHub: `https://github.com/manu-labs/patch-mcp-grok-plugin`.

**Grok Build** – run `/plugin`, search for `patch-mcp` and install it.

**Any other MCP client** – no plugin needed. Paste this line into the agent:

```text
Connect to https://www.patchmcp.com/mcp to make phone calls.
```

### First connection

The first time Grok uses a Patch tool, it opens `www.patchmcp.com` in your browser:

1. **Which number should calls come from?** Enter your mobile number (US or Canada).
2. **Enter the code we texted you.** This proves you control the number and registers it as your caller ID.
3. **Let Grok place calls for you?** Accept the Terms of Use and Privacy Policy and approve the connection.

That verified number *is* your account: there is no sign-up, no password and no API key. Coming back from a new browser, another text code gets you in.

## Usage

Ask naturally. Grok picks up the phone when the job needs one:

```text
Call Zuni Café and book a table for two tonight at 7:30 under Sam. Anywhere between 7 and 8:30 works.
```

```text
Phone the pharmacy on Valencia St and ask whether my prescription for Alex Rivera is ready for pickup.
```

```text
Call the courier about order 48213 and find out why it says "delivery attempted" when nobody rang the bell.
```

Grok returns the outcome, what the other side committed to, anything left for you to do, and a private link to the transcript and recording. While the call is running you get a **listen link** where you can hear it live, read the transcript as it happens, and stop the call.

### Tools

| Tool | What it does |
| --- | --- |
| `place_call` | Start a call from your verified number with an objective, optional context and callee name. Returns a `call_id` and a private `listen_url` immediately. |
| `get_call` | Status of a call and, once it ends, its `resolution` (outcome, summary, commitments, follow-ups, key facts), transcript and recording link. Supports server-side long-polling with `wait_seconds`. |
| `list_calls` | Your recent calls with statuses and outcomes. |
| `end_call` | Stop a call in progress. The agent wraps up politely and hangs up. |

## What the person you call experiences

- **Honest from hello.** Before anything else, the agent says it is an AI calling on behalf of you (by the name you chose) and that the call is recorded. It never pretends to be human. If the person would rather not continue, it thanks them and ends the call.
- **Your number on their screen.** Calls only ever go out from a number you proved you control, so people can call you back. Nothing is spoofed.
- **Easy to refuse.** Anyone can tell the agent not to call again, or enter their number on the public [do-not-call page](https://www.patchmcp.com/opt-out); after that no Patch MCP user can reach it.
- **Reasonable hours.** Calls are placed between 8:00 and 21:00 at the destination's local time. US and Canadian numbers only. Emergency, premium-rate and service ranges are blocked.

## Pricing

New accounts get **10 free calls** (up to 3 minutes each; a free call only counts once someone picks up and speaks). After that, **Starter** is $10/month for 20 minutes of talk time and **Pro** is $25/month for 60. Current details are on the [pricing page](https://www.patchmcp.com/pricing); Grok gets a private upgrade link in the tool response when a plan limit is reached.

## Security and privacy

For reviewers and anyone deciding whether to install:

- **Network endpoints.** The plugin contacts one host: `https://www.patchmcp.com` (OAuth discovery, authorization, token and the `/mcp` endpoint). No other endpoints, no telemetry.
- **Credentials.** None to configure. On first use the client completes OAuth 2.1 with PKCE and dynamic client registration against `www.patchmcp.com` and stores the resulting bearer token; the token is scoped to `calls` and is only ever sent to that host. No environment variables are read; no files on your machine are touched.
- **Local execution.** None. There are no hooks, scripts, binaries or `stdio` servers in this plugin; the MCP server is remote.
- **What leaves your machine.** Only the tool arguments Grok sends (`to`, `objective`, `context`, `callee_name`, `user_name`, `max_duration_minutes`). Patch MCP uses them to place the call and stores the resulting transcript, resolution and recording on your account so you can review them.
- **Sub-processors.** Behind `www.patchmcp.com`, call content is handled by xAI (the Grok voice agent and text models) and Telnyx (telephony and recording storage); the full provider table, with what each one receives, is in the Privacy Policy under [Who we share it with](https://www.patchmcp.com/privacy#share). Manu Labs never uses call content to train models, and the AI provider handles it only to conduct and review your calls.
- **Policies.** [Terms of Use](https://www.patchmcp.com/terms) · [Privacy Policy](https://www.patchmcp.com/privacy) · contact [leko@manulabs.xyz](mailto:leko@manulabs.xyz).

## Removal

1. Remove the plugin in Grok Bot (**Plugins → Yours → Patch MCP**) or, in Grok Build, uninstall `patch-mcp` from `/plugin`.
2. Optionally revoke the connection on the server side: open [your account](https://www.patchmcp.com/account) → **Connected apps** → **Disconnect**. That invalidates the tokens Grok holds. Your call history stays on the account until you ask for it to be deleted (see the Privacy Policy).

If you *received* a call and never want another, use the [do-not-call page](https://www.patchmcp.com/opt-out).

## Repository layout

```text
.grok-plugin/plugin.json   plugin manifest
.mcp.json                  hosted Patch MCP server (type: http)
skills/patch-mcp/SKILL.md  how to brief, run and read a call
LICENSE                    MIT
```

## License

This plugin (the manifest, skill and documentation in this repository) is released under the [MIT License](LICENSE). The Patch MCP service is governed by its own [Terms of Use](https://www.patchmcp.com/terms).

## Support

- [GitHub Issues](https://github.com/manu-labs/patch-mcp-grok-plugin/issues) for anything about the plugin
- [leko@manulabs.xyz](mailto:leko@manulabs.xyz) for the service itself
