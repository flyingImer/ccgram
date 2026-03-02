# Extension and Fix Playbook

## Common Extension Paths

Add a new provider:

1. Implement provider in `src/ccbot/providers/<name>.py` following `AgentProvider` contract.
2. Register provider in `src/ccbot/providers/__init__.py`.
3. Define capabilities accurately (resume/continue/hook/status behavior).
4. Add provider tests in `tests/ccbot/test_provider_contracts.py` and provider-specific tests.

Add a new Telegram command or callback:

1. Register command/callback in `src/ccbot/bot.py`.
2. Implement handler in `src/ccbot/handlers/`.
3. Add callback prefix/constant in `handlers/callback_data.py` if needed.
4. Add/adjust tests for routing + handler behavior.

Add session state fields:

1. Extend dataclasses/serialization in `src/ccbot/session.py`.
2. Ensure load path is backward compatible with missing keys.
3. Update migration logic if key semantics change (`window_resolver.py` / migration tests).

Adjust status or transcript parsing:

1. Keep parsing provider-specific where possible.
2. Preserve message queue ordering and tool-use/tool-result pairing semantics.
3. Validate with parser unit tests and monitor integration tests.

## Bug-Fix Triage

1. Localize the layer first:

- routing/state (`session.py`)
- monitor/parsing (`session_monitor.py`, providers, parsers)
- delivery/UI (`handlers/*`, `message_queue.py`)
- integration boundary (`tmux_manager.py`, `hook.py`)

2. Reproduce with narrow tests:

- start with module-local tests, then run broader suites.

3. Fix with architecture-safe changes:

- avoid bypassing SessionManager state model.
- avoid handler-to-handler tight coupling when shared helper/module fits better.

4. Re-run checks:

- `make fmt && make test && make lint`
- then `make typecheck` (or `make check` for full gate)

## Safe Change Checklist

- uses existing abstractions (`session_manager`, provider protocol, tmux manager).
- no regressions in topic<->window identity behavior.
- no direct raw-string `context.user_data` keys; use `handlers/user_state.py` constants.
- tests updated for changed behavior.
- formatting, lint, and tests pass.
