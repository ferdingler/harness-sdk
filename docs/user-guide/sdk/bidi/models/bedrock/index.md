[Amazon Nova Sonic](https://docs.aws.amazon.com/nova/latest/nova2-userguide/using-conversational-speech.html) brings expressive, multilingual voice conversations to your agents. Users can speak naturally, interrupt a reply, and ask the agent to take action through tools.

Follow the [quickstart](/docs/user-guide/sdk/bidi/quickstart/index.md) to build a voice agent that listens, speaks, and calls tools.

## Installation

Python version

Nova Sonic requires Python 3.12 or later.

Install the Nova Sonic extra:

```bash
pip install "strands-agents[bidi]"
```

For local audio dependencies and device setup, see [Audio I/O](/docs/user-guide/sdk/bidi/io/index.md#audio-io).

## Credentials

Configure AWS credentials with permission to invoke Nova Sonic in a [supported region](https://docs.aws.amazon.com/bedrock/latest/userguide/model-card-amazon-nova-2-sonic.html). `BedrockNovaSonicModel` uses Boto3’s [credential chain](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/credentials.html), including environment variables, named profiles, and IAM roles. See [Setting up AWS credentials](/docs/user-guide/sdk/model-providers/amazon-bedrock/index.md#setting-up-aws-credentials) for shared setup instructions.

To use a named profile, pass a Boto3 session with its region:

```python
import boto3

from strands.bidi.models import BedrockNovaSonicModel

model = BedrockNovaSonicModel(
    model_id="amazon.nova-2-sonic-v1:0",
    boto_session=boto3.Session(profile_name="voice-agent", region_name="us-east-1"),
)
```

Without a custom session, you can pass `region` directly. If omitted, the model uses Boto3’s configured region, falling back to `us-east-1`.

## Configuration

Create a model with the voice and audio settings for your application, then pass it to `BidiAgent`. This example selects the `tiffany` voice and 24 kHz output audio:

```python
from strands.bidi.agent import BidiAgent
from strands.bidi.models import BedrockNovaSonicModel

model = BedrockNovaSonicModel(
    model_id="amazon.nova-2-sonic-v1:0",
    region="us-east-1",
    voice="tiffany",
    audio={"output": {"sample_rate": 24000}},
)
agent = BidiAgent(model=model)
```

Choose a voice from Nova Sonic’s [voices and languages](https://docs.aws.amazon.com/nova/latest/nova2-userguide/sonic-language-support.html). Audio uses mono PCM, with 16 kHz input and output by default. Set either stream’s sample rate through [`audio`](/docs/api/python/strands.bidi.models#strands.bidi.models.BedrockNovaSonicAudioConfig).

See the [`BedrockNovaSonicModel` API reference](/docs/api/python/strands.bidi.models#strands.bidi.models.BedrockNovaSonicModel.__init__) for all constructor options.

### Session settings

Pass Nova Sonic’s [session settings](https://docs.aws.amazon.com/nova/latest/nova2-userguide/sonic-input-events.html) through `params` to tune inference and turn detection. For example, `LOW` endpointing sensitivity allows longer pauses before the model decides the user has finished speaking:

```python
from strands.bidi.models import BedrockNovaSonicModel

model = BedrockNovaSonicModel(
    model_id="amazon.nova-2-sonic-v1:0",
    params={"turnDetectionConfiguration": {"endpointingSensitivity": "LOW"}},
)
```

Use the provider’s field names inside `params`, as shown above.

## Connection restarts

Nova Sonic limits each connection to eight minutes. By default, `BidiAgent` schedules a restart after seven minutes and replays conversation history into the new connection. See [Connection restarts](/docs/user-guide/sdk/bidi/agent/index.md#connection-restarts) to customize the timing and track the transition.

On startup and restart, the adapter replays text from the [conversation history](https://docs.aws.amazon.com/nova/latest/nova2-userguide/sonic-chat-history.html), including [completed transcripts](/docs/user-guide/sdk/bidi/events/index.md#messages). It truncates individual messages to 50 KiB and retains up to 200 KiB of recent history. These limits apply to the history sent when opening a connection.

## Related pages

- [Guardrails](/docs/user-guide/sdk/safety-security/guardrails/index.md) (2 shared tags)
- [Amazon Nova](/docs/user-guide/sdk/model-providers/amazon-nova/index.md) (2 shared tags)
- [Bedrock Knowledge Base Store](/docs/user-guide/sdk/memory/bedrock-knowledge-base/index.md) (2 shared tags)
- [Deploying Strands Agents to Amazon Bedrock AgentCore Runtime](/docs/user-guide/sdk/deploy/deploy_to_bedrock_agentcore/index.md) (2 shared tags)
- [Python Deployment to Amazon Bedrock AgentCore Runtime](/docs/user-guide/sdk/deploy/deploy_to_bedrock_agentcore/python/index.md) (2 shared tags)
- [TypeScript Deployment to Amazon Bedrock AgentCore Runtime](/docs/user-guide/sdk/deploy/deploy_to_bedrock_agentcore/typescript/index.md) (2 shared tags)
- [AgentCore evaluations](/docs/user-guide/evals-sdk/how-to/agentcore_evaluation_dashboard/index.md) (2 shared tags)
- [Amazon Bedrock](/docs/user-guide/sdk/model-providers/amazon-bedrock/index.md) (2 shared tags)
- [BidiAgent](/docs/user-guide/sdk/bidi/agent/index.md) (1 shared tag)
- [Bidirectional Streaming](/docs/user-guide/sdk/bidi/index.md) (1 shared tag)


## Implementation

### Python

- [harness-sdk/strands-py/src/strands/bidi/models/bedrock.py](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/bidi/models/bedrock.py)
- [harness-sdk/strands-py/src/strands/bidi/models/configs.py](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/bidi/models/configs.py)
