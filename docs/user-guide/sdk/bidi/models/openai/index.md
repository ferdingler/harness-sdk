[OpenAI Realtime](https://developers.openai.com/api/docs/guides/realtime) brings responsive voice conversations to your agents, with support for text and images alongside speech. Users can share context, interrupt a reply, and ask the agent to take action through tools.

Follow the [quickstart](/docs/user-guide/sdk/bidi/quickstart/index.md) to build a voice agent that listens, speaks, and calls tools.

## Installation

Install the OpenAI Realtime extra:

```bash
pip install "strands-agents[bidi-openai]"
```

For local audio dependencies and device setup, see [Audio I/O](/docs/user-guide/sdk/bidi/io/index.md#audio-io).

## Credentials

Create an [OpenAI API key](https://platform.openai.com/settings/organization/api-keys) and set it in your environment:

```bash
export OPENAI_API_KEY="your-api-key"
```

`OpenAIRealtimeModel` reads `OPENAI_API_KEY` automatically. You can also pass the key directly through `api_key`. For organization and project options, see the [constructor reference](/docs/api/python/strands.bidi.models#strands.bidi.models.OpenAIRealtimeModel.__init__).

## Configuration

Configure the Realtime model, user transcription, and output voice, then pass the model to `BidiAgent`:

```python
from strands.bidi.agent import BidiAgent
from strands.bidi.models import OpenAIRealtimeModel

model = OpenAIRealtimeModel(
    model_id="gpt-realtime-2.1",
    transcription_model_id="gpt-transcribe",
    voice="coral",
)
agent = BidiAgent(model=model)
```

The `transcription_model_id` argument is required: choose a [transcription model](https://developers.openai.com/api/docs/models/gpt-transcribe) for user speech transcripts, or pass `None` to disable them. The Realtime model processes audio directly, independently of this transcription.

Choose a voice from OpenAI’s [voice options](https://developers.openai.com/api/docs/guides/realtime-conversations#voice-options). This adapter requires mono PCM at 24 kHz for both audio input and output.

See the [`OpenAIRealtimeModel` API reference](/docs/api/python/strands.bidi.models#strands.bidi.models.OpenAIRealtimeModel.__init__) for all constructor options.

### Session settings

Pass OpenAI’s [session settings](https://developers.openai.com/api/reference/resources/realtime/client-events#session.update) through `params` to customize the conversation. The adapter uses [server voice activity detection](https://developers.openai.com/api/docs/guides/realtime-vad#server-vad) by default. This example allows 800 milliseconds of silence before detecting the end of speech:

```python
from strands.bidi.models import OpenAIRealtimeModel

model = OpenAIRealtimeModel(
    model_id="gpt-realtime-2.1",
    transcription_model_id="gpt-transcribe",
    params={"audio": {"input": {"turn_detection": {"silence_duration_ms": 800}}}},
)
```

Use OpenAI’s field names inside `params`. Nested settings merge with the defaults, and `params` takes precedence over direct options such as `voice`. Keep turn detection, automatic responses, and interruption enabled; the adapter requires them.

## Text output

The model produces speech and its transcript by default. To generate text without audio, set `output_modalities` through `params`:

```python
from strands.bidi.models import OpenAIRealtimeModel

model = OpenAIRealtimeModel(
    model_id="gpt-realtime-2.1",
    transcription_model_id=None,
    params={"output_modalities": ["text"]},
)
```

For a complete terminal conversation using this configuration, see [Console I/O](/docs/user-guide/sdk/bidi/io/index.md#console-io).

## Connection restarts

OpenAI limits Realtime sessions to [60 minutes](https://developers.openai.com/api/docs/guides/realtime-conversations#session-lifecycle-events). The adapter uses a 50-minute timeout to leave time for reconnection before that limit. By default, `BidiAgent` schedules a restart after 45 minutes, ahead of this timeout. Each restart opens a new session and replays conversation history, including text, completed transcripts, and tool calls and results.

Set `connection.restart_after_s` to customize the schedule. The `timeout_s` option controls the adapter’s timeout and defaults to its maximum of 3000 seconds (50 minutes). Keep the scheduled restart earlier than that timeout to allow time for the transition. See [Connection restarts](/docs/user-guide/sdk/bidi/agent/index.md#connection-restarts) for timing and events.

## Related pages

- [BidiAgent](/docs/user-guide/sdk/bidi/agent/index.md) (1 shared tag)
- [Bidirectional Streaming](/docs/user-guide/sdk/bidi/index.md) (1 shared tag)
- [Bidirectional Streaming Models](/docs/user-guide/sdk/bidi/models/index.md) (1 shared tag)
- [Google Gemini Live](/docs/user-guide/sdk/bidi/models/google/index.md) (1 shared tag)
- [I/O Streams](/docs/user-guide/sdk/bidi/io/index.md) (1 shared tag)
- [Input Content](/docs/user-guide/sdk/bidi/content/index.md) (1 shared tag)
- [Interrupts](/docs/user-guide/sdk/bidi/interrupts/index.md) (1 shared tag)
- [Stream Events](/docs/user-guide/sdk/bidi/events/index.md) (1 shared tag)
- [Tools](/docs/user-guide/sdk/bidi/tools/index.md) (1 shared tag)
- [Bidirectional Streaming Observability](/docs/user-guide/sdk/bidi/observability/index.md) (1 shared tag)


## Implementation

### Python

- [harness-sdk/strands-py/src/strands/bidi/models/openai.py](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/bidi/models/openai.py)
- [harness-sdk/strands-py/src/strands/bidi/models/configs.py](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/bidi/models/configs.py)
