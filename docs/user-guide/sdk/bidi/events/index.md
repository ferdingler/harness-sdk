Use stream events to react to a conversation as it unfolds. `BidiAgent` emits events that carry new content or signal a change in state. An event can deliver an audio chunk, add text to a transcript, mark the start or end of a response, or report a tool result.

Your application consumes these events to play audio, display text, and track ongoing work. A single response can produce many events, while user transcripts and background tool results can arrive independently. All of them flow through the same event stream.

## Consume events

Read events with `agent.receive()` after starting the agent. To forward events to output streams, use [`agent.run()`](/docs/user-guide/sdk/bidi/io/index.md), which manages the agent lifecycle and sends each event to every configured output stream.

Use `isinstance()` to access typed properties, or check the `type` string and read dictionary fields. The example below demonstrates both styles by sending a text prompt and printing completed transcripts until the first model response ends:

(( tab "Typed events" ))
```python
import asyncio

from strands.bidi.agent import BidiAgent
from strands.bidi.types import (
    BidiResponseStopEvent,
    BidiTranscriptBlockEvent,
)


async def main() -> None:
    async with BidiAgent() as agent:
        await agent.send("Say hello in one sentence.")

        async for event in agent.receive():
            if isinstance(event, BidiTranscriptBlockEvent):
                print(f"{event.role}: {event.transcript}")
            elif isinstance(event, BidiResponseStopEvent):
                break


asyncio.run(main())
```
(( /tab "Typed events" ))

(( tab "Dictionary access" ))
```python
import asyncio

from strands.bidi.agent import BidiAgent


async def main() -> None:
    async with BidiAgent() as agent:
        await agent.send("Say hello in one sentence.")

        async for event in agent.receive():
            event_type = event.get("type")
            if event_type == "bidi_transcript_block":
                print(f"{event['role']}: {event['transcript']}")
            elif event_type == "bidi_response_stop":
                break


asyncio.run(main())
```
(( /tab "Dictionary access" ))

## Event families

Model adapters emit connection starts, response boundaries, content streams, barge-in signals, tool requests, and usage reports. The agent adds completed blocks, tool results, restart notifications, and connection stops.

Provider support and configuration determine which events appear. See the [event API reference](/docs/api/python/strands.bidi.types) for complete class definitions and fields.

### Connections

Use connection events to track when the model is ready, a restart is approaching, or the agent is shutting down.

| Event | Meaning |
| --- | --- |
| [`BidiConnectionStartEvent`](/docs/api/python/strands.bidi.types#strands.bidi.types.BidiConnectionStartEvent) | The model connection is ready. Includes its `connection_id` and `model`. |
| [`BidiConnectionWarningEvent`](/docs/api/python/strands.bidi.types#strands.bidi.types.BidiConnectionWarningEvent) | A scheduled restart is approaching. `time_left_s` estimates the time remaining. |
| [`BidiConnectionRestartEvent`](/docs/api/python/strands.bidi.types#strands.bidi.types.BidiConnectionRestartEvent) | The agent is replacing the connection. `reason` identifies a scheduled restart or timeout. |
| [`BidiConnectionStopEvent`](/docs/api/python/strands.bidi.types#strands.bidi.types.BidiConnectionStopEvent) | The agent is shutting down. Includes `connection_id`. |

### Responses

Use response events to track a model response to user input, conceptually similar to a single `Agent` invocation. Match the start and stop events by `response_id`.

| Event | Meaning |
| --- | --- |
| [`BidiResponseStartEvent`](/docs/api/python/strands.bidi.types#strands.bidi.types.BidiResponseStartEvent) | A model response begins. |
| [`BidiResponseStopEvent`](/docs/api/python/strands.bidi.types#strands.bidi.types.BidiResponseStopEvent) | The response’s output ends. Buffered audio may still be playing. |

Background [tool execution](#tool-execution) can continue after response stop, and tool results can trigger a follow-up model response.

### Audio

Use audio events to play the assistant’s speech. Events for one audio stream share a `content_id`.

| Event | Meaning |
| --- | --- |
| [`BidiAudioStartEvent`](/docs/api/python/strands.bidi.types#strands.bidi.types.BidiAudioStartEvent) | An audio stream begins. |
| [`BidiAudioDeltaEvent`](/docs/api/python/strands.bidi.types#strands.bidi.types.BidiAudioDeltaEvent) | The next audio chunk is in `audio`, encoded as base64. |
| [`BidiAudioStopEvent`](/docs/api/python/strands.bidi.types#strands.bidi.types.BidiAudioStopEvent) | The audio stream ends. |

Each delta also includes the audio `format`, `sample_rate`, and `channels`. Buffered audio may still be playing when `BidiAudioStopEvent` arrives. See [`AudioIO`](/docs/user-guide/sdk/bidi/io/index.md#audio-io) for playback handling.

### Transcripts

Use transcript events to display user or assistant speech as text. Each event includes the speaker’s `role` (`"user"` or `"assistant"`) and the stream’s `content_id`.

| Event | Meaning |
| --- | --- |
| [`BidiTranscriptStartEvent`](/docs/api/python/strands.bidi.types#strands.bidi.types.BidiTranscriptStartEvent) | A transcript stream begins. |
| [`BidiTranscriptDeltaEvent`](/docs/api/python/strands.bidi.types#strands.bidi.types.BidiTranscriptDeltaEvent) | The next transcript fragment is in `delta`. |
| [`BidiTranscriptStopEvent`](/docs/api/python/strands.bidi.types#strands.bidi.types.BidiTranscriptStopEvent) | The transcript stream ends. |
| [`BidiTranscriptBlockEvent`](/docs/api/python/strands.bidi.types#strands.bidi.types.BidiTranscriptBlockEvent) | The complete transcript is in `transcript`. |

### Text

Use text events for written assistant output. Events for one text stream share a `content_id`.

| Event | Meaning |
| --- | --- |
| [`BidiTextStartEvent`](/docs/api/python/strands.bidi.types#strands.bidi.types.BidiTextStartEvent) | A text stream begins. |
| [`BidiTextDeltaEvent`](/docs/api/python/strands.bidi.types#strands.bidi.types.BidiTextDeltaEvent) | The next text fragment is in `delta`. |
| [`BidiTextStopEvent`](/docs/api/python/strands.bidi.types#strands.bidi.types.BidiTextStopEvent) | The text stream ends. |
| [`BidiTextBlockEvent`](/docs/api/python/strands.bidi.types#strands.bidi.types.BidiTextBlockEvent) | The complete text is in `text`. |

### Reasoning

Use reasoning events for reasoning text or thought summaries exposed by the model. Events for one reasoning stream share a `content_id`.

| Event | Meaning |
| --- | --- |
| [`BidiReasoningStartEvent`](/docs/api/python/strands.bidi.types#strands.bidi.types.BidiReasoningStartEvent) | A reasoning stream begins. |
| [`BidiReasoningDeltaEvent`](/docs/api/python/strands.bidi.types#strands.bidi.types.BidiReasoningDeltaEvent) | The next reasoning fragment is in `delta`. |
| [`BidiReasoningStopEvent`](/docs/api/python/strands.bidi.types#strands.bidi.types.BidiReasoningStopEvent) | The reasoning stream ends. |
| [`BidiReasoningBlockEvent`](/docs/api/python/strands.bidi.types#strands.bidi.types.BidiReasoningBlockEvent) | The complete reasoning is in `text`. |

### Tools

Use tool events to track requests, intermediate output, and completed results.

| Event | Meaning |
| --- | --- |
| [`BidiToolUseBlocksEvent`](/docs/api/python/strands.bidi.types#strands.bidi.types.BidiToolUseBlocksEvent) | The model requests a group of tool calls in `tool_uses`. Each call includes `toolUseId`, `name`, and `input`. |
| `ToolStreamEvent` | A running tool yields intermediate output. `tool_stream_event` contains `tool_use` and `data`. |
| `ToolResultEvent` | A tool finishes. `tool_result` contains its `toolUseId`, `status`, and result `content`. |
| `ToolResultMessageEvent` | The agent records all results from a tool group in a history `message`. |

### Barge-in

[`BidiBargeInEvent`](/docs/api/python/strands.bidi.types#strands.bidi.types.BidiBargeInEvent) signals an interruption to model output, whether audio, text, or other content. It carries no fields beyond `type`. See [Barge-in](#barge-in-1) for responding to interrupted output.

### Usage

Use [`BidiUsageEvent`](/docs/api/python/strands.bidi.types#strands.bidi.types.BidiUsageEvent) to track token usage during a conversation. Each event reports additional `input_tokens`, `output_tokens`, and `total_tokens`; sum these counts to track conversation totals.

Optional `input_token_details` and `output_token_details` provide breakdowns by modality, cache, or reasoning, depending on the provider. See [Tracking token usage](/docs/user-guide/sdk/bidi/observability/index.md#tracking-token-usage) for an example.

## Event ordering

Follow these ordering rules when consuming events or implementing a custom model. The diagram illustrates an audio response with an assistant transcript and a separate user transcript.

Audio and transcript event orderingTwo columns show a user transcript and an assistant response containing audio and transcript events. Each content stream follows its own start, delta, and stop sequence. Transcript blocks follow transcript stop. All assistant content finishes before response stop, with token usage shown immediately before it. Nova Sonic can report usage outside response boundaries. User transcripts can interleave independently; aligned rows do not imply timing. Breaks in the lifelines indicate that the conversation can continue before a connection-stop event signals the end of the conversation.User transcriptAssistant responseBidiConnectionStartEventBidiConnectionStopEventBidiTranscriptStartEventBidiTranscriptDeltaEventBidiTranscriptStopEventBidiTranscriptBlockEventBidiResponseStartEventBidiAudioStartEventBidiTranscriptStartEventBidiAudioDeltaEventBidiTranscriptDeltaEventBidiAudioStopEventBidiTranscriptStopEventBidiTranscriptBlockEventBidiUsageEventBidiResponseStopEventConversation continues

Nova Sonic usage timing

[Amazon Bedrock Nova Sonic](/docs/user-guide/sdk/bidi/models/bedrock/index.md) currently reports usage independently of response boundaries. Its usage events can arrive before, during, or after a response.

User transcripts have their own lifecycle and can start or finish before, during, or after an assistant response. Aligned rows do not imply timing, and a transcript start does not mark the exact moment speech began. Process assistant audio and tool requests without waiting for a completed user transcript.

1.  **Connection starts.** `BidiConnectionStartEvent` precedes output from that model connection. Track the connection by `connection_id`. A connection can contain multiple responses.
2.  **Response starts.** `BidiResponseStartEvent` precedes the response’s assistant content and tool requests. Match response start and stop by `response_id`.
3.  **Content streams.** Each stream emits a start, zero or more deltas, and a stop. Text, reasoning, and other streams can interleave with the audio and transcripts shown, and streams can finish in a different order from their starts. Track each stream by its `content_id`, unique within the connection. A response can contain multiple streams and tool groups.
4.  **Completed blocks follow.** The agent emits each completed block after its content stop, such as `BidiTextBlockEvent` after `BidiTextStopEvent`. Custom models supply start, delta, and stop events; the agent assembles the block. Audio has no completed block event.
5.  **Usage is reported.** `BidiUsageEvent` provides token counts before response stop.
6.  **Response stops.** `BidiResponseStopEvent` follows all assistant content stops and completed blocks in that response. User transcripts and background tool results are independent of this boundary.
7.  **Connection stops.** `BidiConnectionStopEvent` signals the end of the conversation; the event iterator then ends.

Use deltas for live updates. A completed block contains the full text for its stream, so replace the displayed partial text when it arrives. See [Messages](#messages) for how this content becomes conversation history.

The model’s events stay in order, but background tool results and restart notifications can appear between them, even between a content stop and its block. Correlate events by their identifiers rather than adjacency.

A stop, connection failure, or forced restart can leave content or a response unfinished. Clean up application state without requiring a final content stop, block, or response stop event.

### Tool execution

Track tool calls by `toolUseId`; results can arrive out of order or during another response. In this example, the tool group finishes after the response that requested it:

Tool execution can continue after response stopAn assistant response requests a group of tools, which run concurrently. Tools can emit intermediate output. In this example the response stops before the tools finish. Each tool emits a result, then the agent records all results in a tool result message.Assistant responseTool executionBidiResponseStartEventBidiToolUseBlocksEventToolStreamEventZero or more per toolBidiResponseStopEventToolResultEventOne per toolToolResultMessageEventAll results recordedStart requested tools

1.  **Request the group.** `BidiToolUseBlocksEvent` contains complete calls in provider order. The agent starts them concurrently.
2.  **Receive tool output.** Tools can emit intermediate output through `ToolStreamEvent`. Each `ToolResultEvent` arrives as a tool finishes.
3.  **Record the results.** After all calls finish, `ToolResultMessageEvent` exposes their results in request order. It confirms the history update, not delivery to the model.

Tool errors reported through `ToolResultEvent` use `status: "error"`. Failures that escape the tool runner raise from `receive()`.

### Barge-in

Use `BidiBargeInEvent` to stop presenting the current model output while the conversation and any background tools continue. The example below shows interrupted audio, but the same signal applies to text and other output:

Barge-in during a responseBarge-in signals interrupted model output, including audio, text, or other content. This audio example shows response start, audio start, and audio delta events, followed by BidiBargeInEvent, audio stop, and response stop. For audio output, clear buffered audio when barge-in arrives. Barge-in must occur between response start and response stop.Assistant responseBidiResponseStartEventBidiAudioStartEventBidiAudioDeltaEventBidiBargeInEventBidiAudioStopEventBidiResponseStopEvent

Clear buffered audio as soon as the event arrives so playback stops promptly. [`AudioIO.output()`](/docs/user-guide/sdk/bidi/io/index.md#audio-io) does this automatically; a custom output stream must clear its own playback queue.

### Connection restarts

Use restart events to track connection replacement and identify interrupted turns. With automatic restart enabled, the agent can replace its model connection on a schedule or after a timeout. It emits `BidiConnectionRestartEvent` before replacement and `BidiConnectionStartEvent` when the new connection is ready:

Scheduled and timeout connection restartsTwo alternative paths lead to a replacement connection. A scheduled restart is preceded by a warning. A timeout restart needs no warning. Either path emits a restart event before replacing the connection. A successful replacement emits a connection start with a new connection ID.Scheduled restartTimeoutBidiConnectionWarningEventBidiConnectionRestartEventreason: "scheduled"BidiConnectionRestartEventreason: "timeout"BidiConnectionStartEventNew connection\_idReplace connection

-   **Scheduled restart.** A warning estimates the time remaining in `time_left_s`; conversation events can occur before the restart. The restart event has `reason: "scheduled"` and `timeout_error: None`.
-   **Timeout.** No warning is required. The restart event has `reason: "timeout"`, and `timeout_error` contains the model’s `ConnectionTimeoutError`.

When `turn_interrupted` is `True`, the restart cut off an active response or an unanswered turn. Replayed history supplies context but does not request an answer to that turn, so notify the user or send another prompt once the new connection is ready.

If your application tracks unfinished content streams, stop treating them as active when the new connection starts. You can keep partial content for display, but treat new streams separately. Background tools can continue across the restart.

## Messages

`BidiAgent` builds conversation state in `agent.messages` as it processes streamed events. For each text, reasoning, or transcript stream, it groups events by `content_id` and assembles a separate message:

1.  **Start.** The agent appends a placeholder message with an empty `content` list and a status of `"pending"`.
2.  **Delta.** It accumulates text in an internal buffer while the pending message stays empty.
3.  **Stop.** It fills the placeholder with the accumulated content, marks it `"complete"`, and emits the corresponding completed block event.

Reserving each message at stream start keeps history in that order, even when streams finish in a different order. A single response can contribute several messages.

For these streamed messages, `message["metadata"]["custom"]["bidi"]` records the content `kind` (`"text"`, `"reasoning"`, or `"transcript"`) and completion `status`.

If a connection ends before a stream stops, the agent marks its message `"incomplete"`. Text and reasoning retain their partial content; unfinished transcripts contain `[Transcript unavailable.]`. The agent does not emit completed blocks for unfinished streams.

The messages list stores assembled content alongside user input and tool exchanges:

| Content | Representation in history |
| --- | --- |
| User input sent as text or images | A user message containing those blocks. |
| User or assistant transcript | A user or assistant message containing a `text` block. |
| Assistant text | An assistant message containing a `text` block. |
| Reasoning | An assistant message containing a `reasoningContent` block. |
| Tool activity | Assistant messages containing `toolUse` blocks, paired with user messages containing `toolResult` blocks. |

Audio chunks are not stored in history. Speech appears as transcript text when the model provides it.

Tool requests add an exchange containing the calls and dispatch acknowledgements. After all calls in the group finish, the agent appends a second exchange with the same calls and their results, then emits `ToolResultMessageEvent`.

## Related pages

- [BidiAgent](/docs/user-guide/sdk/bidi/agent/index.md) (1 shared tag)
- [Bidirectional Streaming](/docs/user-guide/sdk/bidi/index.md) (1 shared tag)
- [Bidirectional Streaming Models](/docs/user-guide/sdk/bidi/models/index.md) (1 shared tag)
- [Google Gemini Live](/docs/user-guide/sdk/bidi/models/google/index.md) (1 shared tag)
- [I/O Streams](/docs/user-guide/sdk/bidi/io/index.md) (1 shared tag)
- [Input Content](/docs/user-guide/sdk/bidi/content/index.md) (1 shared tag)
- [Interrupts](/docs/user-guide/sdk/bidi/interrupts/index.md) (1 shared tag)
- [OpenAI Realtime](/docs/user-guide/sdk/bidi/models/openai/index.md) (1 shared tag)
- [Tools](/docs/user-guide/sdk/bidi/tools/index.md) (1 shared tag)
- [Bidirectional Streaming Observability](/docs/user-guide/sdk/bidi/observability/index.md) (1 shared tag)


## Implementation

### Python

- [harness-sdk/strands-py/src/strands/bidi/types/events.py](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/bidi/types/events.py)
- [harness-sdk/strands-py/src/strands/bidi/models/model.py](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/bidi/models/model.py)
- [harness-sdk/strands-py/src/strands/bidi/agent/agent.py](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/bidi/agent/agent.py)
- [harness-sdk/strands-py/src/strands/bidi/agent/loop.py](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/bidi/agent/loop.py)
- [harness-sdk/strands-py/src/strands/bidi/agent/_blocks.py](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/bidi/agent/_blocks.py)
- [harness-sdk/strands-py/src/strands/bidi/io/audio.py](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/bidi/io/audio.py)
- [harness-sdk/strands-py/src/strands/types/_events.py](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/types/_events.py)
