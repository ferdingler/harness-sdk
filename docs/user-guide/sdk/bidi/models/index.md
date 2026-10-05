A `BidiAgent` talks to a realtime model: a hosted speech-to-speech service, such as Nova Sonic, Gemini Live, or OpenAI Realtime, that holds one connection open for the whole conversation and listens while it responds. In your code, a model provider class stands in for that service. It opens the connection and translates the service’s streaming protocol into Strands events, so tools, hooks, I/O streams, and event handling work the same whichever service you choose.

Realtime models are a separate family from the models a standard `Agent` uses, and each provider class works only with `BidiAgent`.

## Supported providers

| Provider | Model class | Input | Connection limit |
| --- | --- | --- | --- |
| [Amazon Nova Sonic](/docs/user-guide/sdk/bidi/models/bedrock/index.md) | `BedrockNovaSonicModel` | [Audio, text](https://docs.aws.amazon.com/nova/latest/nova2-userguide/using-conversational-speech.html) | [8 minutes](https://docs.aws.amazon.com/nova/latest/nova2-userguide/using-conversational-speech.html) |
| [Google Gemini Live](/docs/user-guide/sdk/bidi/models/google/index.md) | `GoogleGeminiLiveModel` | [Audio, text, images](https://ai.google.dev/gemini-api/docs/live) | [About 10 minutes](https://ai.google.dev/gemini-api/docs/live-session) |
| [OpenAI Realtime](/docs/user-guide/sdk/bidi/models/openai/index.md) | `OpenAIRealtimeModel` | [Audio, text, images](https://developers.openai.com/api/docs/guides/realtime-conversations) | [60 minutes](https://developers.openai.com/api/docs/guides/realtime-conversations) |

Each provider page covers credentials, client options, and behavior specific to that model.

## Configuration

Every provider accepts the same core options, so moving between providers keeps the shape of your configuration:

| Option | Purpose |
| --- | --- |
| `model_id` | The provider’s model identifier. Required. |
| `voice` | The voice the model speaks with. Voice names are provider-specific. |
| `params` | Provider-native session settings, such as temperature or turn detection. |
| `connection` | Connection restart timing. See [Connection limits](#connection-limits). |

`params` is the pass-through to the provider’s own API. Field names and casing follow that provider’s documentation, which each provider page links to.

```python
from strands.bidi.models import BedrockNovaSonicModel

model = BedrockNovaSonicModel(
    model_id="amazon.nova-2-5-sonic",
    voice="tiffany",
    params={"inferenceConfiguration": {"temperature": 0.7}},
)
```

To change configuration later, call `update_config()`. New values apply the next time the connection opens, not to the connection already in progress.

## Audio

Use the model’s audio configuration to set up capture and playback. The built-in providers implement [`AudioCapable`](/docs/api/python/strands.bidi.models#strands.bidi.models.AudioCapable), which exposes `get_audio_config()`. It returns separate `input` and `output` settings containing each stream’s sample rate, channel count, and encoding.

[`AudioIO`](/docs/user-guide/sdk/bidi/io/index.md#audio-io) reads these settings automatically. Custom [I/O streams](/docs/user-guide/sdk/bidi/io/index.md#custom-io) can read them to configure audio capture and playback. The built-in providers use mono PCM; sample rates and configuration options vary by provider. See each provider’s page for its supported settings.

## Connection limits

Every realtime provider limits how long a single connection stays open (see [Supported providers](#supported-providers)). The agent restarts the connection shortly before that limit, at a break between turns when it can, and the conversation continues on the new connection. Providers carry context across the restart in one of two ways:

-   **History replay**: Nova Sonic and OpenAI Realtime open a fresh connection and receive the conversation history.
-   **Session resumption**: Gemini Live reconnects to the same server-side session through [session resumption](/docs/user-guide/sdk/bidi/models/google/index.md#connection-restarts), falling back to history replay if resumption is unavailable.

Each provider sets its own default restart timing, and the `connection` option overrides it or turns automatic restarts off. For the restart sequence and the events it emits, see [Connection restarts](/docs/user-guide/sdk/bidi/agent/index.md#connection-restarts).

## Custom providers

To connect a realtime model Strands doesn’t support yet, subclass `BidiModel`. A provider opens and closes the connection, sends user input and tool results, and yields Strands events as the model responds, in the order described in [Event ordering](/docs/user-guide/sdk/bidi/events/index.md#event-ordering). See the [`BidiModel` API reference](/docs/api/python/strands.bidi.models#strands.bidi.models.BidiModel) for the contract, and the built-in providers in `strands/bidi/models/` for working examples.

## Related pages

- [BidiAgent](/docs/user-guide/sdk/bidi/agent/index.md) (1 shared tag)
- [Bidirectional Streaming](/docs/user-guide/sdk/bidi/index.md) (1 shared tag)
- [Google Gemini Live](/docs/user-guide/sdk/bidi/models/google/index.md) (1 shared tag)
- [I/O Streams](/docs/user-guide/sdk/bidi/io/index.md) (1 shared tag)
- [Input Content](/docs/user-guide/sdk/bidi/content/index.md) (1 shared tag)
- [Interrupts](/docs/user-guide/sdk/bidi/interrupts/index.md) (1 shared tag)
- [OpenAI Realtime](/docs/user-guide/sdk/bidi/models/openai/index.md) (1 shared tag)
- [Stream Events](/docs/user-guide/sdk/bidi/events/index.md) (1 shared tag)
- [Tools](/docs/user-guide/sdk/bidi/tools/index.md) (1 shared tag)
- [Bidirectional Streaming Observability](/docs/user-guide/sdk/bidi/observability/index.md) (1 shared tag)


## Implementation

### Python

- [harness-sdk/strands-py/src/strands/bidi/models/model.py](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/bidi/models/model.py)
- [harness-sdk/strands-py/src/strands/bidi/models/configs.py](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/bidi/models/configs.py)
