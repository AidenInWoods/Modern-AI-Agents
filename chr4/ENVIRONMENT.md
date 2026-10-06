# Chapter IV: minimal interface migration

The original 16-cell structure is preserved. Installation commands, tool imports/calls,
and all three agent/model blocks have been migrated without changing their roles.

Use `.venv/bin/python` / **Python 3.11 (Chapter 4)**. Restart the notebook kernel once
because package versions were changed during the earlier attempt. Reopen the notebook
from disk if VS Code is holding an unsaved older version; preserve your own edits first.

## Changes

- `%pip` targets the notebook kernel. The package list now includes `langchain-classic`
  and `ddgs`, plus the original code's missing runtime dependencies.
- Existing `DuckDuckGoSearchRun`, `GoogleSerperAPIWrapper`, `WikipediaQueryRun` and
  `Tool` classes are retained. No custom tools, `@tool`, HTTP implementations or Ollama.
- Tool classes come from `langchain_community`; `Tool` comes from `langchain_core`.
  Tool `.run()` calls become `.invoke()`. `GoogleSerperAPIWrapper.run()` remains:
  that is the utility wrapper's own current method, not deprecated `AgentExecutor.run()`.
- Serper key input uses `getpass` so credentials are not stored in notebook source.
- Model loading uses `BitsAndBytesConfig(load_in_4bit=True)` passed as
  `quantization_config`, with automatic device placement. This changes the interface,
  not the model or its hardware requirements.
- Sampling is explicit (`do_sample=True`). `return_full_text=False` prevents the
  echoed prompt from being interpreted as the agent's newly generated decision.
- `initialize_agent` is replaced by `create_react_agent` plus `AgentExecutor` in
  `langchain_classic`. The explicit prompt reuses the library's original ReAct
  template constants. The decision chain and execution loop are now constructed separately.
- Four iterations allow the model to read a tool result and answer; one iteration
  could stop immediately after the first search. Calls now use
  `agent.invoke({"input": query})` and read `response["output"]`.
- The later Llama block uses `meta-llama/Llama-3.2-3B-Instruct`, the newest small
  text-only Llama that preserves the existing `AutoModelForCausalLM` structure.
  Llama 4 is multimodal and substantially larger, so adopting it would change this
  notebook's model class and hardware assumptions.
- The final Mistral block uses `mistralai/Ministral-8B-Instruct-2410`, a newer
  text-only Mistral-family checkpoint compatible with the same loading structure.
  Ministral 3 is multimodal and requires a different model/tokenizer path.

## Why the compatibility package?

This retains the book's text-based ReAct + local Transformers pipeline design.
It is not a migration to LangChain's recommended structured-tool-calling `create_agent`
architecture. That would require changing the model integration and teaching flow.
The original Phi-3 and benchmark model IDs are kept in this minimal pass; no claim
is made that those models are current. Benchmark logic and hand-entered scores are
also the book's original examples, with the limitations already discussed.

No packages are installed, models downloaded, or model inference executed by this
revision. Syntax and local interface compatibility are checked separately from real
model inference. First `from_pretrained` execution may still download multiple GB,
and working 4-bit execution depends on the model, hardware and bitsandbytes backend.

`requirements.txt` records intended direct package dependencies.
`requirements-macos-arm64.lock.txt` is the earlier full environment snapshot: it still
contains installed extras from the abandoned migration. It is not a minimal dependency
list. No package was uninstalled during this correction.
