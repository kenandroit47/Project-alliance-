# HUE Windows Local Bridge

This directory defines the local, permissioned bridge between the HUE browser/orchestrator and Windows hardware/software.

## Responsibilities

- Run locally on the trusted Windows machine.
- Accept only authenticated HUE events.
- Keep device identifiers and permissions local.
- Expose a narrow localhost API/WebSocket interface to the HUE UI.
- Forward approved events to capability adapters (audio, MIDI/OSC, DAW, voice, controller).
- Emit health/status events without exposing secrets.

## Security boundary

The browser is not trusted with hardware credentials. The bridge is the privileged boundary.

Default policy:

- bind to localhost only;
- deny unknown devices;
- require an authenticated session for privileged events;
- validate every event against `../schema/hue-event.schema.json`;
- rate-limit repeated control events;
- log security-relevant events locally;
- never store passwords, API keys, OAuth refresh tokens, or voice samples in this repository.

## Adapter model

```text
Browser / HUE UI
       |
 localhost WebSocket / HTTP
       |
 HUE Windows Bridge
       |
 +-----+--------+---------+---------+
 |              |         |         |
Audio          MIDI      DAW       Voice
Adapter        /OSC      Adapter   Adapter
```

## Initial endpoints (design contract)

`GET /health` — local process health only.

`GET /session` — current non-secret session status.

`POST /events` — submit a validated HUE event; privileged events require an authenticated session.

`WS /events` — stream normalized events and status updates to the local UI.

Actual hardware access is intentionally not implemented by this documentation-only step. Each adapter must be implemented and permission-tested independently before it is enabled.
