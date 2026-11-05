# Agentic AI with Local Models

Slides and demos from the November 5, 2026 workshop on running open-weight
language models locally and building agentic workflows around them.

Core for Computational Biomedicine (CCB) · Department of Biomedical Informatics · Harvard Medical School

---

Large language models are powerful tools for biomedical data analysis, but
they don't have to run on a commercial cloud service like ChatGPT or Claude.
With local inference, the model runs on your laptop, a desktop, or a server
inside the HMS network, so you control which model version you use and where
your data goes.

This workshop covers how to choose open-weight models and tools that suit a
task and your hardware, the performance, reproducibility, cost and privacy
trade-offs of local versus cloud inference, and how to wire a local model
into an agentic loop that calls real scientific tools. The same methods
carry over to larger models on GPU servers at HMS, such as the O2 and
Longwood clusters.

## Format

| | Part | What it covers |
|---|---|---|
| **1** | **Foundations and model selection** | A presentation on LLM basics, open-weight models, hardware, choosing a model size, quantization, specialization (e.g. Mixture of Experts), model hubs, and software options. |
| **2** | **A local multimodal chatbot and a tool-using agent** | Demos of a multimodal chatbot and an agent that retrieves gene expression data from the GTEx portal, both running entirely on a laptop. |
| **3** | **Performance, safety and next steps** | Customizing and optimizing local models, common failure modes and troubleshooting, the risks of letting agents act on your environment, and running larger models on the O2 and Longwood clusters. |

## Learning objectives

By the end of the workshop you will be able to:

- Explain the benefits and limitations of running an LLM on your local computer.
- Differentiate the open-weight models available online by size,
  specialization, quantization, format, and model hub.
- Run a model on your local hardware for chat, tool calling, and with agent
  harnesses.
- Know the settings, failure modes, and safety concerns that matter for
  local LLMs.

## What's here

| Path | Contents |
|---|---|
| [`slides/`](slides/) | The workshop presentation (PDF) |
| [`local-inference-example/`](local-inference-example/) | The Part 2 demos. Its [README](local-inference-example/README.md) covers setup, how the agent loop works, and notes for running it live |
| [`local-inference-example/docs/multimodal-chat.md`](local-inference-example/docs/multimodal-chat.md) | The multimodal chatbot: asking a local vision model about an image in Ollama or LM Studio, without uploading it anywhere |
| [`local-inference-example/src/local_inference_example/`](local-inference-example/src/local_inference_example/) | The tool-using agent: a small local model, a hand-written agent loop, five tools, and a browser chat and chart UI |
| [`local-inference-example/scripts/`](local-inference-example/scripts/) | `find_hardware_model.py` suggests models that fit your machine; `clean_models.py` deletes the downloaded weights afterwards |

## Get a head start

Downloading a model is the slowest step, so do it before the workshop if you
can. You need [uv](https://docs.astral.sh/uv/getting-started/installation/),
which works the same on macOS, Windows, and Linux.

```bash
git clone https://github.com/ccb-hms/agentic-ai-with-local-models.git
cd agentic-ai-with-local-models/local-inference-example
uv sync
uv run local-inference-example
```

The first run downloads
[Phi-3.5-mini-instruct](https://huggingface.co/microsoft/Phi-3.5-mini-instruct)
(about 7 GB, cached under `~/.cache/huggingface`), then opens
http://127.0.0.1:8000 with the model loaded. It runs on CPU, so no GPU is
needed. Ask it *"where is BRCA1 expressed?"* to check that it works. The
model runs offline once downloaded, but its tools query the public GTEx API
live, so answering a question needs an internet connection.

For the multimodal chatbot, install [Ollama](https://ollama.com/download) or
[LM Studio](https://lmstudio.ai/download) and download a local
vision-capable model such as `llama3.2-vision`.
[The guide](local-inference-example/docs/multimodal-chat.md) walks through
both apps, including how to avoid their paid cloud models.

## The demos

**A local multimodal chatbot.** Ollama and LM Studio give you a
general-purpose chat window backed by a model on your own machine. With a
vision-capable model you can drop in an image, such as a radiology scan, and
ask about it without the image ever leaving your laptop.

**A tool-using agent.** A small open-weight model runs on your laptop inside
a short agent loop written without any agent framework. Ask about a gene and
the model decides which tools to call, and in what order: resolve the gene
symbol, fetch per-sample expression across GTEx tissues, summarize it, and
plot it. Every step streams into the chat as it happens, so you see each
tool call, the exact API request it made, and the data that came back. Along
the way it shows:

- that what the model is told about each tool lives in plain markdown files
  (`skills/`), separate from the tool code (`tools/`), so you can change its
  instructions without touching Python;
- that constrained decoding guarantees every model output is a well-formed
  tool call, but not that it is the right one;
- why an agent needs a step limit and stall detection: a small model can
  repeat itself instead of answering;
- how to swap in another model with `--model`, or send requests to the HMS
  inference service with `--backend hms` instead of running the model
  locally.

## A note on data safety

Running a model locally keeps your prompts and its output on your machine,
but an agent's tools can still reach the network. In this demo the only
outbound requests go to the public GTEx API, and they carry nothing but a
gene symbol. Two choices change that: `--backend hms` sends your prompts to
the HMS inference service, and picking a *cloud* model in Ollama or LM
Studio runs your chat on that vendor's servers. More generally, an agent can
do whatever its tools and environment allow, so give it only the access it
needs, and check your data's classification and HMS guidance before using
sensitive data with any AI tool, local or not.

## Credits

- **Instructors:** Jane Adams, Machine Learning Engineer, who built the
  demo, and Grey Kuling, Curriculum Fellow.
- **Hosted by** the Core for Computational Biomedicine (CCB), Department of
  Biomedical Informatics, Harvard Medical School.
