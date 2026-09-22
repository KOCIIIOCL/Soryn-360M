# Soryn-360M (Soric-360M)

English | [Русский](README.md)

SmolLM2-360M-Instruct fine-tuned for tool calling, math and Russian.

The weights here are GGUF Q8_0, ready to run out of the box with llama.cpp,
Ollama or any other engine. Nothing to compile, nothing to convert.

## What is inside

- Base model: HuggingFaceTB/SmolLM2-360M-Instruct (360M parameters)
- Format: GGUF, Q8_0 quantization
- File: `soryn-360m-q8_0.gguf` (368 MB)

Weights are hosted in [Releases](https://github.com/KOCIIIOCL/Soryn-360M/releases/tag/v1.0).
The repository itself only holds the README files and the Modelfile, the model
is a separate download.

This is a full fine-tune, not LoRA. It was trained on 4 GB of VRAM using bf16 +
8-bit AdamW with gradient checkpointing, so the hardware requirements for
reproducing it are modest.

## How to run

Simplest option, llama.cpp:

```bash
llama-cli -m soryn-360m-q8_0.gguf -sys "You are a helpful assistant." -p "What is 17 times 23?"
```

Or as a server:

```bash
llama-server -m soryn-360m-q8_0.gguf --port 8080
```

That gives you a regular OpenAI-compatible API on localhost:8080.

Ollama works too, just drop the Modelfile from this repo into your models folder.

## Benchmarks

Compared against the stock SmolLM2-360M-Instruct (also Q8_0). Same questions
for both, temperature 0.

### Tool calling (canonical system prompt, 5 prompts)

| Metric | SmolLM2-360M-Instruct | Soryn-360M (Soric-360M) |
|---|---|---|
| Valid JSON | 4/5 (80%) | **5/5 (100%)** |
| Correct function name | 3/5 (60%) | **5/5 (100%)** |
| Correct arguments | 2/5 (40%) | **5/5 (100%)** |

The base model often has no idea what you want from it. Sometimes it copies the
function description from the system prompt instead of calling it, sometimes it
just parrots the placeholder `{"name": "function_name", "arguments": {"arg": "value"}}`
from the example. Soryn-360M (Soric-360M) calls the right tool with the right arguments on
the first try.

### Arithmetic (12 prompts)

| Metric | SmolLM2-360M-Instruct | Soryn-360M (Soric-360M) |
|---|---|---|
| Correct answers | 8/12 (67%) | 8/12 (67%) |

A tie. Both models know the multiplication table and handle numbers under a
thousand fine, and both fall apart on 99x99 and 1234+5678. 360M parameters
simply is not enough to keep multi-digit operations in working memory.

### Arithmetic in Russian (5 prompts)

| Metric | SmolLM2-360M-Instruct | Soryn-360M (Soric-360M) |
|---|---|---|
| Correct answers | 2/5 (40%) | **4/5 (80%)** |

The base model does not even answer the question in Russian. It echoes it back
and goes off rambling about "why 25 times 4 does not work like that".
Soryn-360M (Soric-360M) reads the task and solves it.

### Russian chat

On open-ended Russian questions both models are about equally weak. Still 360M,
so the quality is "there is an answer, but parts of it are nonsense". The real
gains from the fine-tune are in instruction following and tool calling, not in
conversation.

## Limitations

- The model is small. Hard math, long reasoning and genuinely good chat are
  out of its reach.
- It reads Russian much better than the base model, but generation still
  hallucinates easily.
- It works best with an explicit system prompt. Without one it may start
  echoing the user's question back.

## License

Apache 2.0, same as the base SmolLM2.
