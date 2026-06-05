<div align="center">
  <img src="docs/images/parallax.png" width="720" alt="Parallax">

  <h1>Parallax</h1>

  <p><strong>Distributed LLM serving across Macs, Windows PCs, and GPU machines.</strong></p>
  <p>
    Build a private inference cluster with mixed-device setup,
    automatic node discovery, pipeline-parallel sharding, and a browser setup flow.
  </p>

  [![Latest release](https://img.shields.io/github/v/release/GradientHQ/parallax?label=release)](https://github.com/GradientHQ/parallax/releases)
  [![License](https://img.shields.io/github/license/GradientHQ/parallax.svg)](./LICENSE)
  [![Python](https://img.shields.io/badge/python-3.11--3.13-blue)](./pyproject.toml)
  [![Issues](https://img.shields.io/github/issues-raw/GradientHQ/parallax)](https://github.com/GradientHQ/parallax/issues)

  <p>
    <a href="#quickstart">Quickstart</a> ·
    <a href="#supported-models">Models</a> ·
    <a href="#dashboard">Dashboard</a> ·
    <a href="#news">News</a> ·
    <a href="#docs">Docs</a>
  </p>

  <p>
    <a href="https://gradient.network">Gradient</a> ·
    <a href="https://gradient.network/blog/parallax-the-sovereign-ai-os">Blog</a> ·
    <a href="https://discord.gg/parallaxai">Discord</a> ·
    <a href="https://x.com/tryParallax">X</a> ·
    <a href="https://arxiv.org/pdf/2509.26182v1">Paper</a>
  </p>

  <a href="https://www.producthunt.com/products/parallax-by-gradient?embed=true&utm_source=badge-top-post-badge&utm_medium=badge&utm_source=badge-parallax&#0045;by&#0045;gradient" target="_blank"><img src="https://api.producthunt.com/widgets/embed-image/v1/top-post-badge.svg?post_id=1030922&theme=light&period=daily&t=1761986433128" alt="Parallax by Gradient - Product Hunt Top Post" width="250" height="54" /></a>

  <p><strong>Works with</strong></p>
  <p>
    <img src="docs/images/sglang.png" alt="SGLang" height="28">
    &nbsp;&nbsp;&nbsp;
    <img src="docs/images/vllm.png" alt="vLLM" height="30">
    &nbsp;&nbsp;&nbsp;
    <img src="docs/images/qwen.avif" alt="Qwen" height="30">
    &nbsp;&nbsp;&nbsp;
    <img src="docs/images/deepseek.png" alt="DeepSeek" height="30">
    &nbsp;&nbsp;&nbsp;
    <img src="docs/images/kimi.png" alt="Kimi" height="30">
    &nbsp;&nbsp;&nbsp;
    <img src="docs/images/minimax.png" alt="MiniMax" height="30">
    &nbsp;&nbsp;&nbsp;
    <img src="docs/images/zai.svg" alt="ZAI" height="30">
  </p>
</div>

## News

- [2026/6] 🚀 Parallax now supports ModelScope downloads! Prefix any command with `USE_MODELSCOPE=1`.
- [2026/5] 🍎 Added Apple M5 series chip support for Mac workers.
- [2026/2] 🦞 Parallax now supports OpenClaw integration! See [Docs](./docs/user_guide/work_with_openclaw.md)
- [2025/10] 🔥 Parallax won #1 Product of The Day on Product Hunt!
- [2025/10] 🔥 Parallax version 0.0.1 has been released!

## About

A fully decentralized inference engine developed by [Gradient](https://gradient.network). Parallax lets you build your own AI cluster for model inference across distributed nodes despite their varying configuration and physical location. Its core features include:

- Host local LLMs on personal devices
- Cross-platform support (Macs, Windows PCs, Linux/WSL GPU hosts)
- Pipeline-parallel model sharding
- Paged KV cache management and continuous batching for Mac
- Dynamic request scheduling and routing for high performance
- Local serving endpoint compatible with most agent frameworks

The backend architecture:

- P2P communication powered by [Lattica](https://github.com/GradientHQ/lattica)
- GPU backend powered by [SGLang](https://github.com/sgl-project/sglang) and [vLLM](https://github.com/vllm-project/vllm)
- Mac backend powered by [MLX-LM](https://github.com/ml-explore/mlx-lm)

## Quickstart

This path starts a scheduler with the web UI, joins worker nodes, and sends a request through the OpenAI-compatible API.

### 1. Choose an install path

Different devices use different install paths. After installation, the cluster workflow is the same: start a scheduler, join workers, then call the OpenAI-compatible API.

| Device | Recommended path | Commands |
|:--|:--|:--|
| Apple Silicon macOS | Source install with Mac extras | `git clone https://github.com/GradientHQ/parallax.git`<br>`cd parallax`<br>`./install.sh --extras mac`<br>`source .venv/bin/activate` |
| Linux / WSL GPU host | Source install with GPU extras | `git clone https://github.com/GradientHQ/parallax.git`<br>`cd parallax`<br>`./install.sh --extras gpu`<br>`source .venv/bin/activate` |
| Windows PC | Windows app | Download [Parallax_Win_Setup.exe](https://github.com/GradientHQ/parallax_win_cli/releases/latest/download/Parallax_Win_Setup.exe), open Windows Terminal as administrator, then run `parallax install`. |
| Linux GPU container | Docker | `docker run -it --gpus all --network host gradientservice/parallax:latest bash` (use `:latest-spark` for DGX Spark / GB10). |

For macOS and Linux, the default source install also auto-selects the right extras:

```sh
git clone https://github.com/GradientHQ/parallax.git
cd parallax
./install.sh
source .venv/bin/activate
```

The installer creates `.venv`, installs Parallax, selects the default extras for your platform, and builds the `vllm-rs` frontend binary into `.venv/bin`.

### 2. Start the scheduler

Run this on the machine that should host the setup UI and API:

```sh
parallax run
```

Open [http://localhost:3001](http://localhost:3001), choose a model and node count, then continue to the join screen.

To expose the scheduler to other machines on your network:

```sh
parallax run --host 0.0.0.0
```

For nodes outside the same LAN, enable relay mode:

```sh
parallax run -r
```

### 3. Join workers

Run the generated join command on each worker node. For a local network cluster, the command is usually:

```sh
parallax join
```

For remote or public-network nodes, use the scheduler address shown in the UI or logs:

```sh
parallax join -s <scheduler-address>
```

When all nodes are connected, Parallax routes you to the chat interface.

### 4. Call the API

```sh
curl http://localhost:3001/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "<model>",
    "messages": [{"role": "user", "content": "hello"}],
    "max_tokens": 256,
    "stream": true
  }'
```

Use the model ID you chose in the setup UI, or fetch the active model from `GET /v1/models`.

### Headless quickstart

If you do not need the setup UI, start the scheduler with a model and expected worker count:

```sh
# terminal 1
parallax run -m Qwen/Qwen3-0.6B -n 2

# terminal 2 and each worker node
parallax join
```

To run a single standalone server without the dashboard or scheduler:

```sh
parallax serve --model-path Qwen/Qwen3-0.6B
```

### Downloading from ModelScope

Parallax downloads model weights from Hugging Face by default. To pull from ModelScope instead, set `USE_MODELSCOPE=1` on any Parallax process that resolves or loads the model. Use a model ID that exists on ModelScope.

```sh
USE_MODELSCOPE=1 parallax run -m Qwen/Qwen3-0.6B -n 2
USE_MODELSCOPE=1 parallax join -s <scheduler-address>
USE_MODELSCOPE=1 parallax serve --model-path Qwen/Qwen3-0.6B
```

## Supported Models

Parallax supports a growing set of open model families. On Apple Silicon, many public Hugging Face IDs are mapped to MLX-optimized variants automatically. Recent model work expanded Qwen3.6/Qwen3.5 architecture support, GLM-5.1, MiniMax-M2.7, Step-3.5-Flash, DeepSeek-V3.2, Kimi-K2 Thinking, and gpt-oss safeguard coverage.

| Family | Example model IDs |
|:--|:--|
| Qwen | `Qwen/Qwen3-0.6B`, `Qwen/Qwen3-32B`, `Qwen/Qwen3-Next-80B-A3B-Instruct`, `Qwen/Qwen3.6-27B`, `Qwen/Qwen3-235B-A22B-GPTQ-Int4` |
| DeepSeek | `deepseek-ai/DeepSeek-V3.2`, `deepseek-ai/DeepSeek-R1`, `deepseek-ai/DeepSeek-V3.1` |
| Kimi-K2 | `moonshotai/Kimi-K2-Instruct`, `moonshotai/Kimi-K2-Instruct-0905`, `moonshotai/Kimi-K2-Thinking` |
| MiniMax | `MiniMaxAI/MiniMax-M2`, `MiniMaxAI/MiniMax-M2.1`, `MiniMaxAI/MiniMax-M2.7` |
| GLM / Z.ai | `zai-org/GLM-4.7`, `zai-org/GLM-4.7-Flash`, `zai-org/GLM-5.1` |
| gpt-oss | `openai/gpt-oss-20b`, `openai/gpt-oss-120b`, `openai/gpt-oss-safeguard-20b`, `openai/gpt-oss-safeguard-120b` |
| Llama | `nvidia/Llama-3.1-8B-Instruct-FP8`, `nvidia/Llama-3.3-70B-Instruct-FP8` |
| StepFun | `stepfun-ai/Step-3.5-Flash` |

See [`src/backend/server/static_config.py`](./src/backend/server/static_config.py) for the current model map.

## Dashboard

Parallax ships with a browser setup and chat flow at `http://localhost:3001`, so the first cluster can be launched without hand-writing node configuration.

| Setup | Join workers | Chat |
|:--:|:--:|:--:|
| ![Model and node setup](./docs/images/node_config.png) | ![Node join command](./docs/images/node_join.png) | ![Chat interface](./docs/images/chat_interface.png) |

## Docs

- [Installation Guide](./docs/user_guide/install.md)
- [Getting Started](./docs/user_guide/quick_start.md)
- [Working with OpenClaw](./docs/user_guide/work_with_openclaw.md)
- [Contributing Guide](./docs/CONTRIBUTING.md)

## Contributing

Contributions are welcome. Please read the [Contributing Guide](./docs/CONTRIBUTING.md), run the relevant tests, and open a pull request with a clear description of the change.

## License

Parallax is licensed under the [Apache 2.0 License](./LICENSE).
