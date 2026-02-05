# Local Model Behavioral Testing for Plugin Validation

> Using a MacBook M4 with 128GB RAM to run local LLMs for free behavioral testing of Claude Code plugin agents and skills. Research conducted February 2026.

## Executive Summary

Instead of paying ~$3/run for Anthropic API credits (via `cc-plugin-eval`), we can run local models on Apple Silicon to validate that agent/skill context **measurably changes model behavior in the intended direction**. The recommended stack is **LM Studio** serving **Qwen3-coder-30B** (fast iteration, 85 tok/s) or **Qwen3-next-80B-A3B** (higher quality, 63 tok/s), evaluated through **promptfoo** for A/B comparison and **pytest + litellm** for programmatic assertions. A full 75-scenario A/B test suite completes in **6-15 minutes** at zero cost.

> **Why LM Studio over Ollama?** LM Studio uses native MLX on Apple Silicon, yielding ~20% higher throughput than Ollama's llama.cpp backend. All benchmarks in this document were measured using LM Studio. Both expose an OpenAI-compatible API, so testing frameworks work with either — just change the base URL (`localhost:1234` for LM Studio vs `localhost:11434` for Ollama).

---

## Hardware: MacBook M4 128GB RAM

### Measured Benchmarks

Benchmarks measured on the actual target hardware using **LM Studio** (MLX backend):

| Model | Parameters | Quant | tok/sec | Time to First Token | Role |
|-------|-----------|-------|---------|--------------------|----|
| **qwen/qwen3-coder-30b** | 30B | 8bit | **85.07** | 0.31s | **Primary: coding tasks** |
| **qwen3-next-80b-a3b-thinking-mlx** | 80B | 8bit | **63.50** | 3.56s | **Primary: universal/reasoning** |
| qwen/qwen3-30b-a3b-2507 | 30B-A3B | 8bit | 85.56 | 0.30s | Fast alternative |
| qwen3-30b-a3b-thinking-2507 | 30B-A3B | Q8_0 | 79.34 | 0.33s | With chain-of-thought |
| qwen3-14b | 14B | 8bit | 29.95 | 0.25s | Lightweight option |
| google/gemma-3-27b | 27B | 4bit | 28.57 | 0.66s | Alternative 27B |
| kimi-dev-72b | 72B | Q4_0 | 10.67 | 1.08s | Dense 72B option |
| kimi-dev-72b | 72B | 8bit | 6.16 | 4.61s | High-quality dense |
| apertus-70b-instruct-2509 | 70B | Q8_0 | 5.57 | 1.77s | Dense 70B option |
| apertus-8b-instruct-2509 | 8B | 8bit | 56.37 | 0.39s | Rapid prototyping |

**Key takeaway:** MoE models (Qwen3-next-80B-A3B, Qwen3-coder-30B) are dramatically faster than dense models of similar parameter counts because only a fraction of parameters activate per token. The 80B MoE model runs **10x faster** than a dense 70B model.

### Recommended Models for Plugin Testing

| Use Case | Model | Why |
|----------|-------|-----|
| **Agent behavioral tests (coding)** | qwen/qwen3-coder-30b (8bit, 85 tok/s) | Optimized for code generation, follows coding instructions well, very fast iteration |
| **Agent behavioral tests (general)** | qwen3-next-80b-a3b-thinking-mlx (8bit, 63 tok/s) | Best instruction following, more nuanced reasoning, good for Ansible/container agents |
| **Rapid prototyping** | qwen/qwen3-30b-a3b-2507 (8bit, 85 tok/s) | Fastest option with good quality |
| **Deep reasoning tests** | Qwen3-30B-A3B-Thinking (Q8, 79 tok/s) | Chain-of-thought for complex scenarios |

---

## Testing Approach: Differential Behavioral Testing

### Core Concept

The goal is **not** to replicate Claude's exact behavior. It is to verify that **agent/skill context measurably changes model behavior in the intended direction**.

```
              Same User Message
                    |
       +------------+------------+
       |                         |
  WITH Agent Context       WITHOUT Agent Context
  (system prompt)          (baseline prompt)
       |                         |
       v                         v
   Response A               Response B
       |                         |
       +------------+------------+
                    |
            Compare A vs B
         - Does A follow agent instructions?
         - Does A differ from B in expected ways?
         - Does A contain agent-specific patterns?
```

**Example — Python Developer Agent:**

| Scenario | WITHOUT agent | WITH agent (expected) |
|----------|--------------|----------------------|
| "Fetch data from an API" | `import requests` | `async def` + `httpx` |
| "Install dependencies" | `pip install` | `uv add` |
| "Build a web API" | Flask/Django | FastAPI |
| "Write a function" | No type hints | Full type annotations |

If the WITH-agent response consistently shows these differences, the agent markdown is working as intended.

---

## Test Suite Estimated Duration

Based on measured benchmarks (assuming ~500 output tokens per scenario):

| Model | tok/sec | Per Scenario | 75 Scenarios (A/B = 150 calls) | 15 Scenarios (smoke) |
|-------|---------|-------------|-------------------------------|---------------------|
| qwen/qwen3-coder-30b | 85 | ~6 sec | **~15 min** | ~3 min |
| qwen3-next-80b-a3b-thinking-mlx | 63 | ~8 sec | **~20 min** | ~4 min |
| qwen/qwen3-30b-a3b-2507 | 85 | ~6 sec | **~15 min** | ~3 min |
| kimi-dev-72b Q4 | 10 | ~50 sec | **~125 min** | ~25 min |

The MoE models make a full A/B test suite practical as a development-time workflow.

---

## Option A: promptfoo (A/B Comparison Matrix)

promptfoo is purpose-built for comparing prompt variants. It produces visual side-by-side comparisons and supports any OpenAI-compatible API (including LM Studio).

### Setup

```bash
# Install promptfoo
npm install -g promptfoo

# Load models in LM Studio:
# 1. Open LM Studio
# 2. Download qwen/qwen3-coder-30b (8bit) and/or qwen3-next-80b-a3b-thinking-mlx (8bit)
# 3. Start the local server (Developer tab → Start Server)
# LM Studio serves at http://localhost:1234/v1 by default
```

### Project Structure

```
tests/
  behavioral/
    promptfooconfig.yaml
    prompts/
      with-agent-context.txt
      without-agent-context.txt
    agents/
      python-developer.yaml
      ansible-expert.yaml
      ruff-expert.yaml
    skills/
      precommit.yaml
```

### Configuration — `tests/behavioral/promptfooconfig.yaml`

```yaml
description: "Claude Code Plugin Behavioral Tests"

prompts:
  - id: "with-agent"
    label: "WITH agent context"
    raw: "file://prompts/with-agent-context.txt"
  - id: "without-agent"
    label: "WITHOUT agent context (baseline)"
    raw: "file://prompts/without-agent-context.txt"

providers:
  - id: openai:chat:qwen/qwen3-coder-30b
    label: "Qwen3 Coder 30B (fast)"
    config:
      apiBaseUrl: "http://localhost:1234/v1"
      apiKey: "lm-studio"
      temperature: 0
      max_tokens: 2048

  - id: openai:chat:qwen3-next-80b-a3b-thinking-mlx
    label: "Qwen3 Next 80B (quality)"
    config:
      apiBaseUrl: "http://localhost:1234/v1"
      apiKey: "lm-studio"
      temperature: 0
      max_tokens: 2048

tests: file://agents/python-developer.yaml
```

### Agent System Prompt — `prompts/with-agent-context.txt`

```
You are a helpful AI assistant integrated into a developer's IDE.

The following agent definition has been loaded for this conversation:

---BEGIN AGENT CONTEXT---
{{agent_markdown}}
---END AGENT CONTEXT---

Respond to the user's message following the agent's instructions precisely.

User: {{user_message}}
```

### Baseline — `prompts/without-agent-context.txt`

```
You are a helpful AI assistant integrated into a developer's IDE.

Respond helpfully to the user's message.

User: {{user_message}}
```

### Test Cases — `agents/python-developer.yaml`

```yaml
- vars:
    agent_markdown: "file://../../plugins/python/agents/python-developer.md"
    user_message: "Create a function that fetches data from an API"
  assert:
    # WITH agent: should use async patterns
    - type: contains-any
      value: ["async def", "await", "asyncio"]
    # WITH agent: should use httpx or aiohttp, not requests
    - type: contains-any
      value: ["httpx", "aiohttp"]
    - type: not-contains
      value: "import requests"

- vars:
    agent_markdown: "file://../../plugins/python/agents/python-developer.md"
    user_message: "Set up a new web API project"
  assert:
    - type: icontains
      value: "fastapi"
    - type: contains-any
      value: ["uv", "uv init", "uv add"]
    - type: icontains
      value: "pyproject.toml"

- vars:
    agent_markdown: "file://../../plugins/python/agents/python-developer.md"
    user_message: "How should I manage dependencies?"
  assert:
    - type: icontains
      value: "uv"
    - type: not-icontains
      value: "pip install"

- vars:
    agent_markdown: "file://../../plugins/python/agents/python-developer.md"
    user_message: "Write a function that processes a list of user records"
  assert:
    # WITH agent: should include type hints with return annotation
    - type: regex
      value: "def \\w+\\(.*:.*\\).*->"
```

### Test Cases — `agents/ansible-expert.yaml`

```yaml
- vars:
    agent_markdown: "file://../../plugins/ansible/agents/ansible-expert.md"
    user_message: "Write an Ansible task to install nginx"
  assert:
    # WITH agent: should use fully qualified collection names
    - type: contains-any
      value: ["ansible.builtin.", "community."]
    - type: contains-any
      value: ["idempoten", "changed_when", "check mode"]

- vars:
    agent_markdown: "file://../../plugins/ansible/agents/ansible-expert.md"
    user_message: "I need to store database credentials in my playbook"
  assert:
    - type: icontains
      value: "vault"
    - type: not-icontains
      value: "plaintext"
```

### Running

```bash
cd tests/behavioral

# Run all tests with side-by-side comparison
promptfoo eval

# View results in browser (interactive matrix)
promptfoo view

# Export results
promptfoo eval --output results.json
promptfoo eval --output results.csv
```

---

## Option B: pytest + litellm (Programmatic Assertions)

For integration into a Python test suite with fine-grained control.

### Dependencies

```toml
# pyproject.toml
[project.optional-dependencies]
test-behavioral = [
    "pytest>=8.0",
    "litellm>=1.40",
]
```

### Fixtures — `tests/behavioral/conftest.py`

```python
"""Behavioral test fixtures for Claude Code plugin components."""

from __future__ import annotations

from pathlib import Path

import litellm
import pytest

PLUGIN_ROOT = Path(__file__).parent.parent.parent / "plugins"

# LM Studio serves at http://localhost:1234/v1 (OpenAI-compatible)
LMSTUDIO_BASE = "http://localhost:1234/v1"

MODELS = [
    "openai/qwen/qwen3-coder-30b",
    # Uncomment for thorough validation:
    # "openai/qwen3-next-80b-a3b-thinking-mlx",
]


def load_agent_markdown(plugin: str, agent: str) -> str:
    """Load agent markdown file content."""
    path = PLUGIN_ROOT / plugin / "agents" / f"{agent}.md"
    return path.read_text()


def load_skill_markdown(plugin: str, skill: str) -> str:
    """Load skill SKILL.md file content."""
    path = PLUGIN_ROOT / plugin / "skills" / skill / "SKILL.md"
    return path.read_text()


def query_model(
    model: str,
    user_message: str,
    system_prompt: str | None = None,
    temperature: float = 0.0,
    max_tokens: int = 2048,
) -> str:
    """Query a local model via litellm."""
    messages: list[dict[str, str]] = []
    if system_prompt:
        messages.append({"role": "system", "content": system_prompt})
    messages.append({"role": "user", "content": user_message})

    response = litellm.completion(
        model=model,
        messages=messages,
        temperature=temperature,
        max_tokens=max_tokens,
        api_base=LMSTUDIO_BASE,
        api_key="lm-studio",
    )
    return response.choices[0].message.content


def build_agent_prompt(agent_markdown: str) -> str:
    """Build system prompt with agent context injected."""
    return (
        "You are a helpful AI assistant integrated into a developer's IDE.\n\n"
        "The following agent definition has been loaded:\n\n"
        "---BEGIN AGENT CONTEXT---\n"
        f"{agent_markdown}\n"
        "---END AGENT CONTEXT---\n\n"
        "Follow the agent's instructions precisely when responding."
    )


BASELINE_PROMPT = (
    "You are a helpful AI assistant integrated into a developer's IDE.\n"
    "Respond helpfully to the user's message."
)


@pytest.fixture(params=MODELS)
def model(request: pytest.FixtureRequest) -> str:
    """Parametrize tests across models."""
    return request.param
```

### Agent Tests — `tests/behavioral/test_python_developer.py`

```python
"""Behavioral tests for the python-developer agent."""

from __future__ import annotations

import re

from conftest import (
    BASELINE_PROMPT,
    build_agent_prompt,
    load_agent_markdown,
    query_model,
)

AGENT_MD = load_agent_markdown("python", "python-developer")
AGENT_SYSTEM = build_agent_prompt(AGENT_MD)


class TestAsyncPreference:
    """Agent should prefer async patterns for I/O operations."""

    MSG = "Write a function that fetches data from a REST API endpoint"

    def test_with_agent_uses_async(self, model: str) -> None:
        response = query_model(model, self.MSG, system_prompt=AGENT_SYSTEM)
        assert any(
            kw in response.lower() for kw in ["async def", "await", "asyncio"]
        ), f"Agent should produce async code, got:\n{response[:300]}"

    def test_baseline_comparison(self, model: str) -> None:
        """Record baseline behavior for comparison."""
        response = query_model(model, self.MSG, system_prompt=BASELINE_PROMPT)
        has_async = any(kw in response.lower() for kw in ["async def", "await"])
        has_sync = "import requests" in response.lower()
        # Not an assertion — just documenting baseline behavior
        print(f"Baseline: async={has_async}, sync={has_sync}")


class TestToolchain:
    """Agent should recommend uv, not pip."""

    MSG = "How do I install dependencies for my Python project?"

    def test_with_agent_recommends_uv(self, model: str) -> None:
        response = query_model(model, self.MSG, system_prompt=AGENT_SYSTEM)
        assert "uv" in response.lower(), "Agent should recommend uv"

    def test_with_agent_avoids_pip(self, model: str) -> None:
        response = query_model(model, self.MSG, system_prompt=AGENT_SYSTEM)
        assert "pip install" not in response.lower(), "Agent should not recommend pip install"


class TestFramework:
    """Agent should default to FastAPI."""

    MSG = "I need to build a web API. What framework should I use?"

    def test_with_agent_recommends_fastapi(self, model: str) -> None:
        response = query_model(model, self.MSG, system_prompt=AGENT_SYSTEM)
        assert "fastapi" in response.lower(), "Agent should recommend FastAPI"


class TestTypeHints:
    """Agent should always include type hints."""

    MSG = "Write a function that processes a list of user records"

    def test_with_agent_includes_return_type(self, model: str) -> None:
        response = query_model(model, self.MSG, system_prompt=AGENT_SYSTEM)
        assert re.search(r"\)\s*->\s*\w+", response), (
            "Agent should produce code with return type annotations"
        )
```

### Running

```bash
# Ensure LM Studio local server is running (Developer tab → Start Server)
# Load your model in LM Studio first

# Run behavioral tests
uv run pytest tests/behavioral/ -v --tb=short

# Run only with a specific model
uv run pytest tests/behavioral/ -v -k "qwen3_coder"

# Run with timing info
uv run pytest tests/behavioral/ -v --durations=0
```

---

## Option C: Hybrid Approach (Recommended)

Use **both** tools for complementary strengths:

| Tool | Use For |
|------|---------|
| **promptfoo** | Visual A/B comparison matrix, exploring new test scenarios, sharing results with team |
| **pytest + litellm** | Hard pass/fail assertions, regression testing, integration with pre-commit |

---

## Statistical Approach for Non-Deterministic Outputs

Even at `temperature=0`, local models can produce slightly different outputs. Run each critical test multiple times:

```python
def run_n_times(
    model: str,
    user_message: str,
    system_prompt: str,
    check_fn: callable,
    n: int = 5,
) -> dict:
    """Run a test N times and report pass rate."""
    results = [
        check_fn(query_model(model, user_message, system_prompt=system_prompt))
        for _ in range(n)
    ]
    pass_rate = sum(results) / len(results)
    return {"pass_rate": pass_rate, "passed": sum(results), "total": n}


# Usage:
result = run_n_times(
    model="openai/qwen/qwen3-coder-30b",
    user_message="Create a function to fetch API data",
    system_prompt=AGENT_SYSTEM,
    check_fn=lambda r: "async" in r.lower(),
    n=5,
)
assert result["pass_rate"] >= 0.8, f"Expected 80%+ pass rate, got {result['pass_rate']}"
```

**Settings for maximum reproducibility:**
- `temperature=0`
- `seed=42` (if supported)
- `top_p=1.0`
- Run 3-5 times minimum for critical assertions

---

## Comparison Testing Metrics

### 1. Keyword Presence (Deterministic)

Check if agent-specific vocabulary appears in the response:

```yaml
# promptfoo: agent should use "uv", not "pip"
- type: contains-any
  value: ["uv", "uv add", "uv init"]
- type: not-contains
  value: "pip install"
```

### 2. Behavioral Delta (Custom)

Measure how different the WITH-agent response is from the baseline:

```yaml
# promptfoo: custom JavaScript assertion
- type: javascript
  value: |
    const keywords = ['async', 'uv', 'fastapi', 'ruff', 'type hint'];
    const hits = keywords.filter(k => output.toLowerCase().includes(k));
    return hits.length / keywords.length >= 0.6;
```

### 3. LLM-as-Judge (Model-Graded)

Use the same local model to grade adherence to agent instructions:

```yaml
# promptfoo: model grades the response
- type: llm-rubric
  value: |
    Does this response follow Python best practices for:
    1. Async I/O with httpx or aiohttp
    2. Type hints on all function signatures
    3. Using uv for dependency management
    4. Recommending FastAPI for web APIs
    Rate 1-5.
  provider: openai:chat:qwen3-next-80b-a3b-thinking-mlx
  config:
    apiBaseUrl: "http://localhost:1234/v1"
    apiKey: "lm-studio"
  threshold: 0.7
```

---

## What This Approach Cannot Test

1. **Exact Claude behavior** — No open model replicates Claude. These tests validate that agent context has a measurable effect, not that it produces identical output to Claude.

2. **Real tool invocation** — Claude Code's tool pipeline (Bash, Read, Write) cannot be tested locally. We test whether the model *describes* the right tools and patterns, not whether it *executes* them.

3. **Trigger phrase detection** — Whether Claude Code's routing system selects the right agent for a given prompt is an internal mechanism. Use `cc-plugin-eval` ($3/run) for this.

4. **Context window limits** — Long agent markdown files (300+ lines) plus system prompt plus response must fit in the model's context window. Monitor total token usage.

---

## Cost Comparison

| Approach | Cost/Run | 75 Scenarios (A/B) | Speed |
|----------|---------|-------------------|-------|
| **Local qwen/qwen3-coder-30b** | $0 | $0 (15 min) | Fast iteration |
| **Local qwen3-next-80b-a3b-thinking-mlx** | $0 | $0 (20 min) | Higher quality |
| Claude API Haiku | ~$0.01/scenario | ~$1.50 | ~5 min |
| Claude API Sonnet | ~$0.04/scenario | ~$6.00 | ~8 min |
| cc-plugin-eval (Sonnet) | ~$0.04/scenario | ~$3.00 (batched) | ~15 min |

Local testing is **free and unlimited** — ideal for iterative development.

---

## Quick Start

```bash
# 1. Open LM Studio, download and load your model
#    - qwen/qwen3-coder-30b (8bit) for coding agent tests
#    - qwen3-next-80b-a3b-thinking-mlx (8bit) for general agent tests
#    Start the local server: Developer tab → Start Server
#    Verify: curl http://localhost:1234/v1/models

# 2. Install promptfoo
npm install -g promptfoo

# 3. Install Python test deps
uv add --dev litellm pytest

# 4. Run promptfoo A/B evaluation
cd tests/behavioral && promptfoo eval && promptfoo view

# 5. Run pytest assertions
uv run pytest tests/behavioral/ -v
```

---

## Recommended Integration With Existing Tiers

This approach fits as **Tier 1.5** in the overall validation strategy from [`docs/skill-testing-research.md`](./skill-testing-research.md):

| Tier | Tool | When | Cost | What It Tests |
|------|------|------|------|---------------|
| 1 | `@carlrannaberg/cclint` | Every PR (CI) | Free | Structure, frontmatter, naming |
| **1.5** | **Local model (this doc)** | **Development-time** | **Free** | **Behavioral adherence to agent instructions** |
| 2 | `sjnims/cc-plugin-eval` | Weekly (CI) | ~$3/run | Trigger phrase accuracy via Claude API |
| 3 | Tessl publishing | Optional | Free tier | External evaluation + registry |

---

## Sources

### Model Runners
- [LM Studio](https://lmstudio.ai/) — Recommended. Native MLX backend on Apple Silicon, ~20% faster than Ollama. OpenAI-compatible API at `localhost:1234/v1`
- [Ollama](https://ollama.com/) — Alternative. Uses llama.cpp backend. OpenAI-compatible API at `localhost:11434/v1`
- [vllm-mlx](https://github.com/waybarrios/vllm-mlx) — High-throughput MLX-native serving for batch workloads

### Testing Frameworks
- [promptfoo](https://www.promptfoo.dev/) — LLM evaluation and A/B testing framework
- [promptfoo OpenAI-compatible provider](https://www.promptfoo.dev/docs/providers/openai/) — Works with LM Studio's API
- [litellm](https://docs.litellm.ai/docs/providers/openai_compatible) — Unified LLM API with OpenAI-compatible endpoint support
- [DeepEval](https://deepeval.com/) — LLM evaluation with pytest integration

### Models
- [Qwen3 series](https://huggingface.co/Qwen) — MoE and dense models with strong instruction following
- [Llama 3.3 70B](https://huggingface.co/meta-llama/Llama-3.3-70B-Instruct) — Best open dense model for instruction following

### Related Research
- [Skill Testing Research](./skill-testing-research.md) — Validation tools overview and CI workflow examples
- [Plugin Testing Ecosystem Analysis](./plugin-testing-ecosystem-analysis.md) — How official and community plugins test
- [obra/superpowers tests](https://github.com/obra/superpowers/tree/main/tests) — Community plugin behavioral testing via `claude -p`
