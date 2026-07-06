# Changelog

## Unreleased

### Added: "Did you mean...?" suggestions on no-match

When a spoken phrase doesn't clear the fuzzy-match `threshold` (e.g. "turn on
the pulog" instead of "turn on the plug"), closest_intent can now respond
with its two closest near-misses instead of silently forwarding to the
fallback agent:

```
You:    "turn on the pulog"
Assist: Sorry, I'm not sure I understood. Did you mean "turn on the plug" or "turn on the light"?
```

See [docs/suggestions.md](docs/suggestions.md) for the full behavior,
including exactly where the response shows up (Assist chat, voice
satellites, the Companion app, `conversation.process` calls) and when it
does/doesn't trigger.

**New option:** `suggestions` (default `true`). Set to `false` to restore
the previous behavior, where a no-match always falls straight through to
`fallback_agent`.

#### Files changed

| File | Change |
| --- | --- |
| `custom_components/closest_intent/matching.py` | Added `find_suggestions()` (best near-miss candidate per intent, ignoring the strict threshold) and `describe_candidate()` (renders a candidate as a readable phrase, filling in slots from the user's text when possible). |
| `custom_components/closest_intent/conversation.py` | Added `_build_suggestion_result()`, consulted after Hassil's own parse also fails and before falling through to the fallback agent. Wired the new `suggestions` option through `__init__`, `apply_options`, `async_setup_entry`, and the update listener. |
| `custom_components/closest_intent/const.py` | Added `CONF_SUGGESTIONS`, `DEFAULT_SUGGESTIONS`, `SUGGESTION_THRESHOLD_MARGIN`, `SUGGESTION_MIN_SCORE`. |
| `custom_components/closest_intent/config_flow.py` | Added the `suggestions` toggle to the UI config/options schema. |
| `custom_components/closest_intent/strings.json` | Added UI labels/descriptions for `suggestions`. |
| `README.md` | Documented the new option in the options table and YAML example. |
| `docs/suggestions.md` | New: full write-up of the feature. |
| `tests/conftest.py` | Added `async_set_speech` to the stubbed `IntentResponse`, needed to build the suggestion response in tests. |
| `tests/test_matching.py` | Unit tests for `find_suggestions()` and `describe_candidate()`. |
| `tests/test_conversation.py` | Integration tests: a suggestion is offered on a near-miss, and `suggestions=false` preserves the old passthrough-to-fallback behavior. |
