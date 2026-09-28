# Local Models

Quick reference for the recommended Ollama models.

| Model                   | Recommended use                                                        |
| ----------------------- | ---------------------------------------------------------------------- |
| `qwen3:8b`              | Fast everyday chat, summaries, translations, simple tasks              |
| `qwen3:14b`             | General-purpose quality model, CV/job matching, document analysis      |
| `qwen3-coder:30b` 	  | Coding, debugging, Docker, Bash, Python, JS/TS, SQL                    |
| `deepseek-r1:14b`       | Reasoning, complex analysis, troubleshooting, decision support         |
| `gpt-oss:20b`           | Advanced general use, reasoning, coding, agent/tool-oriented tasks     |

## Rule of thumb

- Default → `qwen3:8b`
- Better quality / documents / jobs → `qwen3:14b`
- Code → `qwen3-coder:30b`
- Hard reasoning → `deepseek-r1:14b`
- Advanced all-round / agent tasks → `gpt-oss:20b`

`gpt-oss:20b`, `qwen3-coder:30b` and `gpt-oss:120b` is heavier than the other models and may use system RAM in addition to GPU VRAM.


# Coding models
FEATURE			| MODEL				| ALIAS	  | SIZE
------------------------+-------------------------------+---------+-------
Codex-like builder	| `qwen3-coder:30b`		| qwen30  | 19 GB
Reasoning+tools+thinker | `gpt-oss:20b`			| gpt20	  | 14 GB
The big, heavy reviewer	| `gpt-oss:120b`		| gpt120  | 65 GB
