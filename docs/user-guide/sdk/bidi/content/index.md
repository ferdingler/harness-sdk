Provide user input to `BidiAgent` as text, images, or streaming audio. Text and image blocks form complete user messages, while audio deltas add samples to an ongoing input stream.

## Send input

Use `agent.send()` to provide input once the agent is running. A string is enough to send text:

```python
import asyncio

from strands.bidi.agent import BidiAgent
from strands.bidi.types import BidiResponseStopEvent


async def main() -> None:
    async with BidiAgent() as agent:
        await agent.send("Tell me about Lisbon.")

        async for event in agent.receive():
            if isinstance(event, BidiResponseStopEvent):
                break


asyncio.run(main())
```

`send()` returns after sending the input. Read the model’s response separately through [`agent.receive()`](/docs/user-guide/sdk/bidi/events/index.md#consume-events), as the example does.

[`BidiAgentInput`](/docs/api/python/strands.bidi.types#strands.bidi.types.BidiAgentInput) defines the forms you can send:

| Content | Accepted forms |
| --- | --- |
| Text | A string, [`TextBlock`](/docs/api/python/strands.types.content#strands.types.content.TextBlock), or `{"text": ...}`. |
| Image | [`ImageBlock`](/docs/api/python/strands.types.media#strands.types.media.ImageBlock) or `{"image": ...}`. |
| Streaming audio | [`AudioDelta`](/docs/api/python/strands.bidi.types#strands.bidi.types.AudioDelta) or `{"audio_delta": ...}`. |

A nonempty list of strings, text/image blocks, or their dictionary forms groups content into one user message, preserving block order.

For continuous input, pass [input streams](/docs/user-guide/sdk/bidi/io/index.md) to `agent.run()`. Each read returns one of these input forms.

## Streaming audio

Send live audio as individual [`AudioDelta`](/docs/api/python/strands.bidi.types#strands.bidi.types.AudioDelta) inputs, outside text and image lists. Each delta carries a chunk of audio samples without explicitly ending the user’s turn; the model provider determines when the turn is complete.

The built-in models expect raw PCM bytes matching their configured input sample rate and channel count. This example forwards chunks from an asynchronous audio source to a running agent:

(( tab "Typed content" ))
```python
from collections.abc import AsyncIterable

from strands.bidi.agent import BidiAgent
from strands.bidi.types import AudioDelta


async def send_audio(agent: BidiAgent, chunks: AsyncIterable[bytes]) -> None:
    async for chunk in chunks:
        await agent.send(AudioDelta(format="pcm", source={"bytes": chunk}))
```
(( /tab "Typed content" ))

(( tab "Dictionaries" ))
```python
from collections.abc import AsyncIterable

from strands.bidi.agent import BidiAgent


async def send_audio(agent: BidiAgent, chunks: AsyncIterable[bytes]) -> None:
    async for chunk in chunks:
        await agent.send({
            "audio_delta": {"format": "pcm", "source": {"bytes": chunk}},
        })
```
(( /tab "Dictionaries" ))

For microphone input, [`AudioIO`](/docs/user-guide/sdk/bidi/io/index.md#audio-io) creates these deltas using the model’s audio configuration.

## Model messages

Your application sends user input through `agent.send()`, which prepares it for [`BidiModel.send()`](/docs/api/python/strands.bidi.models#strands.bidi.models.BidiModel.send). Text and images become a [`BidiMessage`](/docs/api/python/strands.bidi.types#strands.bidi.types.BidiMessage) containing an ordered list of typed blocks; audio deltas pass through individually.

The agent also uses `BidiMessage` to return tool results to the model, packaging each tool’s output as a [`ToolResultBlock`](/docs/api/python/strands.types.tools#strands.types.tools.ToolResultBlock). It handles this delivery automatically, so tool results aren’t part of `BidiAgentInput`. You can track tool calls and results through [tool execution events](/docs/user-guide/sdk/bidi/events/index.md#tool-execution).

See [Messages](/docs/user-guide/sdk/bidi/events/index.md#messages) for how the agent records user input and tool results alongside content assembled from stream events.

## Related pages

- [BidiAgent](/docs/user-guide/sdk/bidi/agent/index.md) (1 shared tag)
- [Bidirectional Streaming](/docs/user-guide/sdk/bidi/index.md) (1 shared tag)
- [Bidirectional Streaming Models](/docs/user-guide/sdk/bidi/models/index.md) (1 shared tag)
- [Google Gemini Live](/docs/user-guide/sdk/bidi/models/google/index.md) (1 shared tag)
- [I/O Streams](/docs/user-guide/sdk/bidi/io/index.md) (1 shared tag)
- [Interrupts](/docs/user-guide/sdk/bidi/interrupts/index.md) (1 shared tag)
- [OpenAI Realtime](/docs/user-guide/sdk/bidi/models/openai/index.md) (1 shared tag)
- [Stream Events](/docs/user-guide/sdk/bidi/events/index.md) (1 shared tag)
- [Tools](/docs/user-guide/sdk/bidi/tools/index.md) (1 shared tag)
- [Bidirectional Streaming Observability](/docs/user-guide/sdk/bidi/observability/index.md) (1 shared tag)


## Implementation

### Python

- [harness-sdk/strands-py/src/strands/bidi/agent/agent.py](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/bidi/agent/agent.py)
- [harness-sdk/strands-py/src/strands/bidi/types/agent.py](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/bidi/types/agent.py)
- [harness-sdk/strands-py/src/strands/bidi/types/content.py](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/bidi/types/content.py)
- [harness-sdk/strands-py/src/strands/bidi/types/media.py](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/bidi/types/media.py)
- [harness-sdk/strands-py/src/strands/bidi/models/model.py](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/bidi/models/model.py)
- [harness-sdk/strands-py/src/strands/types/content.py](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/types/content.py)
- [harness-sdk/strands-py/src/strands/types/media.py](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/types/media.py)
