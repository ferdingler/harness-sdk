[Google Gemini Live](https://ai.google.dev/gemini-api/docs/live) brings multilingual voice conversations to your agents, with support for text and images alongside speech. Users can share context, interrupt a reply, and ask the agent to take action through tools.

Follow the [quickstart](/docs/user-guide/sdk/bidi/quickstart/index.md) to build a voice agent that listens, speaks, and calls tools.

## Installation

Install the Google Gemini Live extra:

```bash
pip install "strands-agents[bidi-google]"
```

For local audio dependencies and device setup, see [Audio I/O](/docs/user-guide/sdk/bidi/io/index.md#audio-io).

## Credentials

Create an API key in [Google AI Studio](https://aistudio.google.com/app/apikey) and set it in your environment:

```bash
export GOOGLE_API_KEY="your-api-key"
```

`GoogleGeminiLiveModel` reads `GOOGLE_API_KEY` automatically. You can also pass the key through `client_args={"api_key": "your-api-key"}`. For other client options, see the [Google GenAI client reference](https://googleapis.github.io/python-genai/genai.html#genai.client.Client).

## Configuration

Configure the Gemini Live model, output voice, and input audio, then pass the model to `BidiAgent`:

```python
from strands.bidi.agent import BidiAgent
from strands.bidi.models import GoogleGeminiLiveModel

model = GoogleGeminiLiveModel(
    model_id="gemini-3.8-live",
    voice="Kore",
    audio={"input": {"sample_rate": 48000}},
)
agent = BidiAgent(model=model)
```

Choose a voice from Google’s [voice options](https://ai.google.dev/gemini-api/docs/live-guide#change-voice-and-language). Audio uses mono PCM, with 16 kHz input by default and fixed 24 kHz output. Set `audio.input.sample_rate` to match your input source; Gemini resamples incoming audio as needed. The adapter enables transcripts for both user speech and model responses.

See the [`GoogleGeminiLiveModel` API reference](/docs/api/python/strands.bidi.models#strands.bidi.models.GoogleGeminiLiveModel.__init__) for all constructor options.

### Session settings

Pass Google’s [`LiveConnectConfig` settings](https://googleapis.github.io/python-genai/genai.html#genai.types.LiveConnectConfig) through `params` to customize the conversation. Gemini uses [automatic voice activity detection](https://ai.google.dev/gemini-api/docs/live-guide#configure-automatic-vad) by default. This example allows 800 milliseconds of silence before detecting the end of speech:

```python
from strands.bidi.models import GoogleGeminiLiveModel

model = GoogleGeminiLiveModel(
    model_id="gemini-3.8-live",
    params={
        "realtime_input_config": {
            "automatic_activity_detection": {"silence_duration_ms": 800},
        },
    },
)
```

Use the Google GenAI SDK’s snake\_case field names inside `params`. Nested settings merge with the defaults, and `params` takes precedence over direct options such as `voice`.

## Connection restarts

Gemini limits each connection to [about 10 minutes](https://ai.google.dev/gemini-api/docs/live-session#maximum-session-duration). By default, `BidiAgent` schedules a restart after nine minutes. The adapter uses [session resumption](https://ai.google.dev/gemini-api/docs/live-session#session-resumption) to reconnect to the same server-side conversation. If resumption is unavailable or fails, it opens a new session and replays text from conversation history, including completed transcripts.

The adapter also enables [context window compression](https://ai.google.dev/gemini-api/docs/live-session#context-window-compression) by default, allowing the session to continue as its history grows. Set `connection.restart_after_s` to customize the restart schedule. See [Connection restarts](/docs/user-guide/sdk/bidi/agent/index.md#connection-restarts) for timing and events.

## Related pages

- [BidiAgent](/docs/user-guide/sdk/bidi/agent/index.md) (1 shared tag)
- [Bidirectional Streaming](/docs/user-guide/sdk/bidi/index.md) (1 shared tag)
- [Bidirectional Streaming Models](/docs/user-guide/sdk/bidi/models/index.md) (1 shared tag)
- [I/O Streams](/docs/user-guide/sdk/bidi/io/index.md) (1 shared tag)
- [Input Content](/docs/user-guide/sdk/bidi/content/index.md) (1 shared tag)
- [Interrupts](/docs/user-guide/sdk/bidi/interrupts/index.md) (1 shared tag)
- [OpenAI Realtime](/docs/user-guide/sdk/bidi/models/openai/index.md) (1 shared tag)
- [Stream Events](/docs/user-guide/sdk/bidi/events/index.md) (1 shared tag)
- [Tools](/docs/user-guide/sdk/bidi/tools/index.md) (1 shared tag)
- [Bidirectional Streaming Observability](/docs/user-guide/sdk/bidi/observability/index.md) (1 shared tag)


## Implementation

### Python

- [harness-sdk/strands-py/src/strands/bidi/models/google.py](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/bidi/models/google.py)
- [harness-sdk/strands-py/src/strands/bidi/models/configs.py](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/bidi/models/configs.py)
