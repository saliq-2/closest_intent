# "Did you mean...?" suggestions

When nothing you say clears the fuzzy-match `threshold`, closest_intent can
respond with its two closest near-misses instead of silently giving up.

```
You:   "turn on the pulog"
Assist: Sorry, I'm not sure I understood. Did you mean "turn on the plug" or "turn on the light"?
```

## Where the suggestion shows up

The suggestion is not a popup, notification, or separate card — it's just the
normal spoken/text response of the conversation agent. It appears wherever an
Assist response normally appears, i.e. whichever surface you used to say the
phrase in the first place:

- The **Assist chat** sidebar/dialog in the Home Assistant frontend.
- A **voice satellite's** TTS playback (ESPHome voice kit, etc.), if the Assist
  pipeline that owns the satellite uses closest_intent as its conversation agent.
- The **Companion app's** Assist screen (iOS/Android).
- The response payload of a manual `conversation.process` service call, e.g.
  from Developer Tools -> Actions, or from within an automation/script.

It replaces whatever the `fallback_agent` would otherwise have said (by
default, Hassil's own "no intent match" error) — it does not add a second
message alongside it.

## When it triggers

A suggestion is only offered when *all* of the following are true:

1. closest_intent's own fuzzy matcher found nothing above `threshold`.
2. Forwarding your raw, unmodified text to Hassil also failed to match any
   intent (so a suggestion is never shown for a phrase Hassil could already
   handle on its own).
3. At least one candidate scored high enough to be worth mentioning: within
   30 points of `threshold`, and never below an absolute floor of 30. This
   keeps unrelated sentences (e.g. idle chatter aimed at an LLM fallback)
   from getting a nonsense suggestion.

Suggestions are deduplicated by intent, so you'll never see the same intent
phrased two different ways in the same prompt — up to two *different*
intents are offered, ranked by how closely they matched.

## Configuration

| Option | Default | Meaning |
| --- | --- | --- |
| `suggestions` | `true` | Enable/disable the "did you mean...?" prompt. When `false`, a no-match falls straight through to `fallback_agent` as it did before this feature existed. |

```yaml
closest_intent:
  suggestions: true
```

The toggle is also available in the integration's UI options
(**Settings -> Devices & services -> Closest Intent -> Configure**).

## Why you might turn it off

- You've configured an LLM as `fallback_agent` and would rather it always
  handle ambiguous phrases (e.g. for open-ended conversation) than get a
  literal-minded "did you mean" prompt first.
- You want the pre-existing passthrough behavior for automations that parse
  the fallback agent's exact response text.
