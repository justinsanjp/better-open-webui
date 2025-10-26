
---

# ⚡ Codex WebUI – Enhanced Fork of Open WebUI

> 🚧 **Work in Progress**: This project is under active development. |Codex Feature got implemented. Please report any issues


---

![GitHub stars](https://img.shields.io/github/stars/justinsanjp/better-open-webui?style=social)
![GitHub forks](https://img.shields.io/github/forks/justinsanjp/better-open-webui?style=social)
![GitHub watchers](https://img.shields.io/github/watchers/justinsanjp/better-open-webui?style=social)
![GitHub repo size](https://img.shields.io/github/repo-size/justinsanjp/better-open-webui)
![GitHub language count](https://img.shields.io/github/languages/count/justinsanjp/beter-open-webui)
![GitHub top language](https://img.shields.io/github/languages/top/justinsanjp/better-open-webui)
![GitHub last commit](https://img.shields.io/github/last-commit/justinsanjp/better-open-webui?color=red)

---

## 🔥 What's New in This Fork?
>these features are getting added soon!.

This fork aims to improve and extend Open WebUI with new capabilities while maintaining compatibility with its core.

### ✅ Key Enhancements

* **🔐 Improved User Management**: Fine-tuned user roles and permissions with a cleaner UI/UX.
* **💳 Subscriptions & Payments (optional)**:

  * Toggleable support for Stripe, PayPal, and more.
  * Disabled by default to maintain privacy-focused, offline-first behavior.
* **🧠 Codex Agents Platform (New)**:

  * A separate page for AI coding agents inspired by tools like OpenAI Codex.
  * Fully standalone, can be enabled or disabled.
  * Designed for code generation, agent workflows, and future development.

> All base features from Open WebUI are preserved and improved upon. See below for a full feature list.

---

## 🌟 Core Features from Open WebUI

Open WebUI is an **extensible**, **feature-rich**, and **offline-first** self-hosted AI platform. It supports **Ollama**, **OpenAI-compatible APIs**, and includes a **built-in inference engine** with **RAG** support.

### Highlights:

* ⚙️ Easy Setup (Docker, pip, K8s)
* 🤖 LLM integration via Ollama, OpenAI, LMStudio, GroqCloud, Mistral, etc.
* 🛡️ Role-based Access Control (RBAC)
* 📱 Responsive Design + PWA
* 📚 Local & Web RAG
* 🖼️ Image Generation (AUTOMATIC1111, DALL-E, etc.)
* 🔍 Web Search & Browsing (Google, Brave, Bing, etc.)
* ✒️ Markdown, LaTeX, and Custom Characters
* 📦 Plugin Framework & Python Function Calling
* 🌐 Multi-language UI (i18n)
* 🧩 Plugin System + Pipelines

For more, check the [Open WebUI docs](https://docs.openwebui.com/).

---

## 🚀 Installation

The fork stays compatible with the official Open WebUI deployment options. This section highlights the most common workflows and points you to the images that are already published for this project.

### Prerequisites

* Docker 24+ or Podman with Docker compatibility
* Optional: Docker Buildx plugin for multi-architecture builds
* Optional: Access to a GitHub Personal Access Token (PAT) with the `write:packages` scope for GHCR pushes

### Use the Published Image (recommended)

The easiest way to get started is to pull the maintained image from the GitHub Container Registry and run it directly:

```bash
docker pull ghcr.io/justinsanjp/better-open-webui:main

docker run -d -p 3000:8080 -v open-webui:/app/backend/data \
  --name better-open-webui --restart=always \
  ghcr.io/justinsanjp/better-open-webui:main
```

> Adjust the container name if `better-open-webui` is already taken on your host.

You can supply common configuration values through environment variables:

```bash
-e OPENAI_API_KEY=sk-... \
-e OLLAMA_BASE_URL=http://ollama:11434 \
-e WEBUI_SECRET_KEY=change-me \
-e PORT=8080
```

For persistent storage the command above mounts the `open-webui` named volume to `/app/backend/data`, which holds conversations and other user data.

### Build the Image Locally

If you prefer to build the container yourself, the provided Dockerfile already contains the necessary defaults. The following command sets the expected build arguments:

```bash
docker build \
  --build-arg BUILD_HASH="$(git rev-parse --short HEAD || echo dev-build)" \
  --build-arg USE_OLLAMA=false \
  --build-arg USE_CUDA=false \
  --build-arg USE_CUDA_VER=cu128 \
  --build-arg UID=1000 \
  --build-arg GID=1000 \
  -t ghcr.io/justinsanjp/better-open-webui:main .
```

Authenticate to GHCR before pushing a locally built image. Use a PAT with the `write:packages` scope:

```bash
echo "${GHCR_PAT}" | docker login ghcr.io -u justinsanjp --password-stdin

docker push ghcr.io/justinsanjp/better-open-webui:main
```

To build and push a multi-architecture image (linux/amd64 and linux/arm64) via Buildx:

```bash
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  --build-arg BUILD_HASH="$(git rev-parse --short HEAD || echo dev-build)" \
  --build-arg USE_OLLAMA=false \
  --build-arg USE_CUDA=false \
  --build-arg USE_CUDA_VER=cu128 \
  --build-arg UID=1000 \
  --build-arg GID=1000 \
  -t ghcr.io/justinsanjp/better-open-webui:main . \
  --push
```

### Deploy with Docker Compose

The repository includes a ready-to-use [`compose.yaml`](compose.yaml). Launch the stack with:

```bash
docker compose up -d
```

Modify the `environment:` section in the compose file to enable API keys, Ollama backends, or custom ports.

### Python (pip) Install

The upstream Python package is still available if you prefer a non-container setup:

```bash
pip install open-webui
open-webui serve
```

Refer to the upstream documentation for overriding the UI/backend paths to load the Codex enhancements.

---

## ⚙️ Codex Agents Page

This new module adds a **dedicated interface for developer-focused agents**, including:

* Inline code generation
* Agent memory and tools
* Multi-modal support (planned)
* File input/output

> Codex Agents is a separate interface and can be disabled via config.

---

## 📜 License

This fork adopts the same license as the original **Open WebUI** project: a [BSD-3-Clause license with additional branding restrictions](LICENSE).

### Open WebUI Components

All code and modifications based on Open WebUI are covered by its license. Branding must remain unless:

* You have explicit written permission,
* You are an enterprise customer, or
* You serve fewer than 50 users.

Attribution and original LICENSE terms must remain intact where applicable.

---

### 🧠 Codex Agents Platform

The Codex Agents module is an original creation in this fork:

* Not based on Open WebUI
* Not subject to Open WebUI branding restrictions
* May be licensed separately (TBD or custom license)
* Treated as an optional and separate component

See the `LICENSE` file for full details.

---

## 💬 Community & Support

Feel free to open issues or join the original [Open WebUI Discord](https://discord.gg/5rJgQTnV4s) for discussions. Contributions are welcome—especially for the new Codex Agents platform!

---

## 🌟 Star History

<a href="https://star-history.com/#open-webui/open-webui&Date">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/svg?repos=open-webui/open-webui&type=Date&theme=dark" />
    <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/svg?repos=open-webui/open-webui&type=Date" />
    <img alt="Star History Chart" src="https://api.star-history.com/svg?repos=open-webui/open-webui&type=Date" />
  </picture>
</a>

---

Created by justinsanjp – Forked from [Timothy Jaeryang Baek (Open WebUI)](https://github.com/tjbck)
Let’s build better AI tools together. 💪

