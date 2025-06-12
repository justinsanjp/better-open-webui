
---

# ⚡ Codex WebUI – Enhanced Fork of Open WebUI

> 🚧 **Work in Progress**: This project is under active development. Until a stable release is ready, we recommend using the official [Open WebUI](https://github.com/open-webui/open-webui) for production use.

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

You can follow the original Open WebUI installation methods as they remain compatible with this fork.

### Docker (Recommended)

```bash
docker run -d -p 3000:8080 -v open-webui:/app/backend/data \
--name codex-webui --restart always ghcr.io/YOUR_USERNAME/codex-webui:main
```

> For GPU support, Stripe config, or Codex activation, see `.env.example`.

### Python pip

```bash
pip install open-webui
open-webui serve
```

> You may override the UI and backend paths to use the Codex-enhanced UI.

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

