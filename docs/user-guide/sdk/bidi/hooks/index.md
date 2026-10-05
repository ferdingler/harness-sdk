Use hooks to run application code when `BidiAgent` updates its conversation history, calls a tool, or restarts its connection. Each callback receives a typed event containing the agent and details about what happened. You can use that information to record activity, update application state, or adjust a tool call before it runs.

Hooks use the same registration system as [Agent hooks](/docs/user-guide/sdk/agents/hooks/index.md). They run as part of the agent’s processing, while [stream events](/docs/user-guide/sdk/bidi/events/index.md) carry output to your application for playback or display.

## Register hooks

This example passes a logger through `BidiAgent`’s `hooks` argument to log when the agent initializes and stops:

```python
from typing import Any

from strands import LocalAgent
from strands.bidi.agent import BidiAgent
from strands.bidi.hooks import BidiAgentStopEvent
from strands.hooks import AgentInitializedEvent, HookRegistry


class LifecycleLogger:
    def on_initialized(self, event: AgentInitializedEvent[LocalAgent]) -> None:
        print(f"Agent {event.agent.name} initialized")

    def on_stop(self, event: BidiAgentStopEvent) -> None:
        print(f"Agent {event.agent.name} stopped")

    def register_hooks(self, registry: HookRegistry, **kwargs: Any) -> None:
        registry.add_callback(AgentInitializedEvent, self.on_initialized)
        registry.add_callback(BidiAgentStopEvent, self.on_stop)


agent = BidiAgent(hooks=[LifecycleLogger()])
```

`LocalAgent` is the interface shared by `Agent` and `BidiAgent`. For shared initialization, message, and tool-call events, use annotations such as `MessageAddedEvent[LocalAgent]` when a callback should work with either agent type. Events prefixed with `Bidi` already type `event.agent` as `BidiAgent` and don’t take a type parameter.

For other ways to register these callbacks, including individual registration and event type inference, see [Agent hooks](/docs/user-guide/sdk/agents/hooks/index.md).

## Hook events

Choose hooks for the part of the conversation you want to observe or customize. Every hook event carries `agent`. Import shared events from `strands.hooks` and events prefixed with `Bidi` from `strands.bidi.hooks`.

### Initialization and shutdown

Use these hooks to set up application resources when the agent is created and release them when it stops:

| Event | When it runs |
| --- | --- |
| [`AgentInitializedEvent`](/docs/api/python/strands.hooks.events#strands.hooks.events.AgentInitializedEvent) | After the agent is initialized, before a model connection opens. |
| [`BidiAgentStopEvent`](/docs/api/python/strands.bidi.hooks#strands.bidi.hooks.BidiAgentStopEvent) | At the end of `stop()`. Callbacks with the same [priority](/docs/user-guide/sdk/agents/hooks/index.md#callback-ordering) run in reverse registration order. |

### Messages

Use message hooks to follow changes to `agent.messages`:

| Event | When it runs |
| --- | --- |
| [`MessageAddedEvent`](/docs/api/python/strands.hooks.events#strands.hooks.events.MessageAddedEvent) | After the agent adds a message. `message` contains the new entry. |
| [`MessageUpdatedEvent`](/docs/api/python/strands.hooks.events#strands.hooks.events.MessageUpdatedEvent) | After the agent replaces a message. Includes its `tracking_id` and replacement `message`. |

Streamed text, reasoning, and transcripts first add an empty placeholder, then update it when their content stream ends. Subscribe to both hooks to observe those history changes. Updates can also mark unfinished content as incomplete; see [Messages](/docs/user-guide/sdk/bidi/events/index.md#messages) for how the agent assembles and stores streamed content.

### Tools

Use tool hooks to inspect or modify individual tool calls:

| Event | When it runs |
| --- | --- |
| [`BeforeToolCallEvent`](/docs/api/python/strands.hooks.events#strands.hooks.events.BeforeToolCallEvent) | Before a tool runs. Use `tool_use` to inspect its name and arguments. |
| [`AfterToolCallEvent`](/docs/api/python/strands.hooks.events#strands.hooks.events.AfterToolCallEvent) | After a tool call. Includes its `result` and any `exception`. |

These hooks support the same [tool interception](/docs/user-guide/sdk/agents/hooks/index.md#tool-interception), [result modification](/docs/user-guide/sdk/agents/hooks/index.md#result-modification), and [retry](/docs/user-guide/sdk/agents/hooks/index.md#tool-call-retry) patterns as `Agent`.

### Responses

Use response hooks to track completion and interruptions:

| Event | When it runs |
| --- | --- |
| [`BidiResponseStopEvent`](/docs/api/python/strands.bidi.hooks#strands.bidi.hooks.BidiResponseStopEvent) | When the model ends a response. Includes `response_id`. |
| [`BidiBargeInEvent`](/docs/api/python/strands.bidi.hooks#strands.bidi.hooks.BidiBargeInEvent) | When the model signals an interruption to its output, such as audio or text. |

The agent runs these hooks before delivering the corresponding [stream events](/docs/user-guide/sdk/bidi/events/index.md#responses) to your application.

### Connection

Use connection hooks to react to restarts and inspect their outcome:

| Event | When it runs |
| --- | --- |
| [`BidiBeforeConnectionRestartEvent`](/docs/api/python/strands.bidi.hooks#strands.bidi.hooks.BidiBeforeConnectionRestartEvent) | Before the agent restarts its model connection. |
| [`BidiAfterConnectionRestartEvent`](/docs/api/python/strands.bidi.hooks#strands.bidi.hooks.BidiAfterConnectionRestartEvent) | After the restart attempt, whether it succeeds or fails. |

Both events include a `reason` of `"scheduled"` or `"timeout"`. For a model timeout, the before event also includes `timeout_error`. The after event’s `exception` is `None` on success and contains the error on failure.

See [Connection restarts](/docs/user-guide/sdk/bidi/agent/index.md#connection-restarts) for configuration and context preservation.

## Related pages

- [BidiAgent](/docs/user-guide/sdk/bidi/agent/index.md) (1 shared tag)
- [Bidirectional Streaming](/docs/user-guide/sdk/bidi/index.md) (1 shared tag)
- [Bidirectional Streaming Models](/docs/user-guide/sdk/bidi/models/index.md) (1 shared tag)
- [Google Gemini Live](/docs/user-guide/sdk/bidi/models/google/index.md) (1 shared tag)
- [I/O Streams](/docs/user-guide/sdk/bidi/io/index.md) (1 shared tag)
- [Input Content](/docs/user-guide/sdk/bidi/content/index.md) (1 shared tag)
- [Interrupts](/docs/user-guide/sdk/bidi/interrupts/index.md) (1 shared tag)
- [OpenAI Realtime](/docs/user-guide/sdk/bidi/models/openai/index.md) (1 shared tag)
- [Stream Events](/docs/user-guide/sdk/bidi/events/index.md) (1 shared tag)
- [Build a custom plugin](/docs/user-guide/sdk/plugins/custom-plugins/index.md) (1 shared tag)


## Implementation

### Python

- [harness-sdk/strands-py/src/strands/bidi/hooks/events.py](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/bidi/hooks/events.py)
- [harness-sdk/strands-py/src/strands/bidi/agent/agent.py](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/bidi/agent/agent.py)
- [harness-sdk/strands-py/src/strands/bidi/agent/loop.py](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/bidi/agent/loop.py)
- [harness-sdk/strands-py/src/strands/hooks/events.py](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/hooks/events.py)
- [harness-sdk/strands-py/src/strands/hooks/registry.py](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/hooks/registry.py)
- [harness-sdk/strands-py/src/strands/types/agent.py](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/types/agent.py)
