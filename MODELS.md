# Local Models

Quick reference for the recommended Ollama models.

| Model                   | Recommended use                                                        |
| ----------------------- | ---------------------------------------------------------------------- |
| `qwen3:8b`              | Fast everyday chat, summaries, translations, simple tasks              |
| `qwen3:14b`             | General-purpose quality model, CV/job matching, document analysis      |
| `deepseek-coder-v2:16b` | Coding, debugging, Docker, Bash, Python, JS/TS, SQL                    |
| `deepseek-r1:14b`       | Reasoning, complex analysis, troubleshooting, decision support         |
| `gpt-oss:20b`           | Advanced general use, reasoning, coding, agent/tool-oriented tasks     |

## Rule of thumb

- Default → `qwen3:8b`
- Better quality / documents / jobs → `qwen3:14b`
- Code → `deepseek-coder-v2:16b`
- Hard reasoning → `deepseek-r1:14b`
- Advanced all-round / agent tasks → `gpt-oss:20b`

`gpt-oss:20b` is heavier than the other models and may use system RAM in addition to GPU VRAM.
