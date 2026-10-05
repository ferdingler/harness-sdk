Use `SnapshotSessionManager` to keep a conversation across application restarts. It saves the agent’s conversation state and restores it when you create another `BidiAgent` for the same session. Storage and checkpoint operations work the same way as [session management with `Agent`](/docs/user-guide/sdk/agents/session-management/index.md).

## Save and resume

Pass a `SnapshotSessionManager` to your agent to save its conversation history and application state. To resume later, create another agent with the same `session_id`, `agent_id`, and storage location.

This example uses the default `"message"` strategy, which saves a snapshot after each message addition or update. It stores a destination alongside the conversation and reads it back from a new agent:

```python
import asyncio

from strands.bidi.agent import BidiAgent
from strands.bidi.types import BidiResponseStopEvent
from strands.session import SnapshotSessionManager
from strands.storage import LocalFileStorage


def create_agent() -> BidiAgent:
    return BidiAgent(
        agent_id="voice-assistant",
        session_manager=SnapshotSessionManager(
            session_id="conversation-123",
            storage=LocalFileStorage("./sessions/"),
            bidi_agent_save_latest_on="message",
        ),
    )


async def main() -> None:
    async with create_agent() as agent:
        agent.state.set("destination", "Lisbon")
        await agent.send("I'm planning a trip to Lisbon.")

        async for event in agent.receive():
            if isinstance(event, BidiResponseStopEvent):
                break

    resumed_agent = create_agent()
    print(resumed_agent.state.get("destination"))  # Lisbon


asyncio.run(main())
```

Creating `resumed_agent` restores its `messages`, `state`, and `system_prompt` immediately, before opening a model connection. On its next `start()`, the agent sends the restored history to the model.

Use `agent.state` for application values you want to keep, such as the destination above. The manager does not save `invocation_state`.

For other storage options, including S3 and custom backends, see [Storage](/docs/user-guide/sdk/storage/index.md).

## Save strategy

Set `bidi_agent_save_latest_on` on `SnapshotSessionManager` to choose when `BidiAgent` updates the latest snapshot used to resume a session:

| Strategy | When the latest snapshot is saved |
| --- | --- |
| `"message"` (default) | After each message addition or replacement, after the agent stops, and before its connection restarts. |
| `"stop"` | After the agent stops and before its connection restarts. |
| `"trigger"` | After the agent stops and before its connection restarts, if `snapshot_trigger` returns `True`. |

The `"message"` strategy follows changes to `agent.messages`, including completed transcripts. Individual streaming deltas and response completion do not trigger saves on their own.

The `"trigger"` strategy lets `snapshot_trigger` decide whether to update the latest snapshot. The [example below](#checkpoints) also uses it to keep checkpoints you can restore later.

## Checkpoints

Keep checkpoints when you want to return to an earlier conversation state. A **checkpoint** is a saved snapshot of that state, retained under its own ID. Later saves update the **latest snapshot** used for automatic resume without overwriting existing checkpoints.

In this example, `snapshot_trigger` checks whether a destination is set. The manager evaluates it after stops and before connection restarts. With the `"trigger"` strategy, a `True` result saves a new checkpoint and updates the latest snapshot with the same state:

```python
import asyncio

from strands.bidi.agent import BidiAgent
from strands.bidi.types import BidiResponseStopEvent
from strands.session import SnapshotSessionManager
from strands.storage import LocalFileStorage


async def main() -> None:
    session_manager = SnapshotSessionManager(
        session_id="trip-checkpoints",
        storage=LocalFileStorage("./sessions/"),
        bidi_agent_save_latest_on="trigger",
        snapshot_trigger=lambda *, agent_data, **_: bool(
            agent_data.state.get("destination")
        ),
    )
    agent = BidiAgent(
        agent_id="voice-assistant",
        session_manager=session_manager,
    )

    for destination in ["Lisbon", "Paris"]:
        async with agent:
            agent.state.set("destination", destination)
            await agent.send(f"Let's plan a trip to {destination}.")

            async for event in agent.receive():
                if isinstance(event, BidiResponseStopEvent):
                    break

    print(agent.state.get("destination"))  # Paris

    snapshot_ids = await session_manager.list_snapshot_ids(agent)
    await session_manager.restore_snapshot(agent, snapshot_id=snapshot_ids[0])
    print(agent.state.get("destination"))  # Lisbon


asyncio.run(main())
```

`list_snapshot_ids()` returns checkpoint IDs from oldest to newest, so the example selects the Lisbon checkpoint. Restoring it changes the agent’s conversation history and destination back to Lisbon. It leaves the latest snapshot in storage unchanged, with the Paris plan.

The agent must be stopped when you restore a checkpoint. On its next `start()`, it sends the restored history to the model.

For more on saving and restoring checkpoints, see [Immutable snapshots](/docs/user-guide/sdk/agents/session-management/index.md#immutable-snapshots). The [Snapshots](/docs/user-guide/sdk/agents/snapshots/index.md) guide covers capturing and loading snapshots directly.

## Related pages

- [Persist state across sessions](/docs/user-guide/sdk/agents/session-management/index.md) (2 shared tags)
- [State Management](/docs/user-guide/sdk/agents/state/index.md) (2 shared tags)
- [Storage](/docs/user-guide/sdk/storage/index.md) (1 shared tag)
- [BidiAgent](/docs/user-guide/sdk/bidi/agent/index.md) (1 shared tag)
- [Bidirectional Streaming](/docs/user-guide/sdk/bidi/index.md) (1 shared tag)
- [Bidirectional Streaming Models](/docs/user-guide/sdk/bidi/models/index.md) (1 shared tag)
- [Google Gemini Live](/docs/user-guide/sdk/bidi/models/google/index.md) (1 shared tag)
- [I/O Streams](/docs/user-guide/sdk/bidi/io/index.md) (1 shared tag)
- [Input Content](/docs/user-guide/sdk/bidi/content/index.md) (1 shared tag)
- [Interrupts](/docs/user-guide/sdk/bidi/interrupts/index.md) (1 shared tag)


## Implementation

### Python

- [harness-sdk/strands-py/src/strands/bidi/agent/agent.py](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/bidi/agent/agent.py)
- [harness-sdk/strands-py/src/strands/bidi/agent/loop.py](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/bidi/agent/loop.py)
- [harness-sdk/strands-py/src/strands/session/snapshot_session_manager.py](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/session/snapshot_session_manager.py)
- [harness-sdk/strands-py/src/strands/types/_snapshot.py](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/types/_snapshot.py)
