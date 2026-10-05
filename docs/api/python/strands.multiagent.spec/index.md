Multi-Agent Specification and Resolution.

Provides the authority-mode system for model-driven multi-agent patterns, where the model configures child agents at runtime. The developer sets axis policies that shape the parameters the model sees and govern what values it can supply:

-   `Fixed` — developer-pinned value; hidden from the model.
-   `Inherit` — value taken from the parent agent; hidden from the model.
-   `Open` — model supplies a free-form value.
-   `Choice` — model picks from a developer-supplied set.

`_resolve_spec` merges model-supplied arguments, preset defaults, and axis policies into a fully resolved `AgentSpec` used to build child agents. `Inherit` axes resolve to `None` (meaning “inherit all”); the class is a self-documenting marker.

## Fixed

```python
@dataclass(frozen=True)
class Fixed()
```

Defined in: [src/strands/multiagent/spec.py:39](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/multiagent/spec.py#L39)

The developer pins the value; the axis contributes no model-facing parameter.

## Inherit

```python
@dataclass(frozen=True)
class Inherit()
```

Defined in: [src/strands/multiagent/spec.py:46](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/multiagent/spec.py#L46)

The child takes the parent’s value; the axis contributes no model-facing parameter.

## Open

```python
@dataclass(frozen=True)
class Open()
```

Defined in: [src/strands/multiagent/spec.py:51](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/multiagent/spec.py#L51)

The model writes the value freely; the axis contributes a string parameter.

## Option

```python
@dataclass(frozen=True)
class Option()
```

Defined in: [src/strands/multiagent/spec.py:56](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/multiagent/spec.py#L56)

A selectable entry in a `Choice`.

## Choice

```python
@dataclass(frozen=True)
class Choice()
```

Defined in: [src/strands/multiagent/spec.py:65](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/multiagent/spec.py#L65)

The model picks from a developer-supplied set.

Each entry is a bare name or an `Option`. Set `multiple=True` to let the model pick more than one.

#### normalized

```python
def normalized() -> list[Option]
```

Defined in: [src/strands/multiagent/spec.py:75](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/multiagent/spec.py#L75)

The options as `Option`s with `value` resolved, wrapping any bare name.

#### to\_schema\_property

```python
def to_schema_property(description: str = "") -> dict[str, Any]
```

Defined in: [src/strands/multiagent/spec.py:87](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/multiagent/spec.py#L87)

Render this axis as a JSON Schema property for a tool’s `inputSchema`.

Produces a `string` enum (or `array` of enum when `multiple`), with per-option descriptions folded into the property’s `description`.

#### value\_for

```python
def value_for(name: str) -> Any
```

Defined in: [src/strands/multiagent/spec.py:109](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/multiagent/spec.py#L109)

The resolved value behind `name`, or `name` itself if unknown.

## Preset

```python
@dataclass(frozen=True)
class Preset()
```

Defined in: [src/strands/multiagent/spec.py:118](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/multiagent/spec.py#L118)

A named role: a partially applied child configuration selected via `agent_type`.

## AgentSpec

```python
@dataclass
class AgentSpec()
```

Defined in: [src/strands/multiagent/spec.py:128](https://github.com/strands-agents/harness-sdk/blob/main/strands-py/src/strands/multiagent/spec.py#L128)

The resolved child configuration handed to the builder.