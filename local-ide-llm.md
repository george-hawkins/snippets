Running an LLM locally to help in your IDE
==========================================

I wanted to run an LLM locally and use it in an IDE in the same way that I use Claude Code.

After a short conversation with Gemini, I went with the [Ollama](https://ollama.com/) model manager, the `deepseek-r1:8b`, the [Continue plugin](https://www.continue.dev/) and [VS Code](https://code.visualstudio.com/).

Installing Ollama and deepseek-r1:8b
------------------------------------

The following steps came from the [Ollama with Continue guide](https://docs.continue.dev/guides/ollama-guide).

```
$ brew install ollama
$ ollama serve
Couldn't find '/Users/georgehawkins/.ollama/id_ed25519'. Generating new private key.
Your new public key is: 

ssh-ed25519 ....................................................................

time=2026-01-01T17:56:14.862+01:00 level=INFO source=routes.go:1554 msg="server config" env="map[HTTPS_PROXY: HTTP_PROXY: NO_PROXY: OLLAMA_CONTEXT_LENGTH:4096 OLLAMA_DEBUG:INFO OLLAMA_FLASH_ATTENTION:false OLLAMA_GPU_OVERHEAD:0 OLLAMA_HOST:http://127.0.0.1:11434 OLLAMA_KEEP_ALIVE:5m0s OLLAMA_KV_CACHE_TYPE: OLLAMA_LLM_LIBRARY: OLLAMA_LOAD_TIMEOUT:5m0s OLLAMA_MAX_LOADED_MODELS:0 OLLAMA_MAX_QUEUE:512 OLLAMA_MODELS:/Users/georgehawkins/.ollama/models OLLAMA_MULTIUSER_CACHE:false OLLAMA_NEW_ENGINE:false OLLAMA_NOHISTORY:false OLLAMA_NOPRUNE:false OLLAMA_NUM_PARALLEL:1 OLLAMA_ORIGINS:[http://localhost https://localhost http://localhost:* https://localhost:* http://127.0.0.1 https://127.0.0.1 http://127.0.0.1:* https://127.0.0.1:* http://0.0.0.0 https://0.0.0.0 http://0.0.0.0:* https://0.0.0.0:* app://* file://* tauri://* vscode-webview://* vscode-file://*] OLLAMA_REMOTES:[ollama.com] OLLAMA_SCHED_SPREAD:false http_proxy: https_proxy: no_proxy:]"
time=2026-01-01T17:56:14.862+01:00 level=INFO source=images.go:493 msg="total blobs: 0"
time=2026-01-01T17:56:14.862+01:00 level=INFO source=images.go:500 msg="total unused blobs removed: 0"
time=2026-01-01T17:56:14.863+01:00 level=INFO source=routes.go:1607 msg="Listening on 127.0.0.1:11434 (version 0.13.5)"
time=2026-01-01T17:56:14.863+01:00 level=INFO source=runner.go:67 msg="discovering available GPUs..."
time=2026-01-01T17:56:14.864+01:00 level=INFO source=server.go:429 msg="starting runner" cmd="/opt/homebrew/Cellar/ollama/0.13.5/bin/ollama runner --ollama-engine --port 56340"
time=2026-01-01T17:56:22.340+01:00 level=INFO source=types.go:42 msg="inference compute" id=0 filter_id=0 library=Metal compute=0.0 name=Metal description="Apple M2" libdirs="" driver=0.0 pci_id="" type=discrete total="10.7 GiB" available="10.7 GiB"
time=2026-01-01T17:56:22.340+01:00 level=INFO source=routes.go:1648 msg="entering low vram mode" "total vram"="10.7 GiB" threshold="20.0 GiB
```

It took about 8s to fully start and runs in the foreground so you have to switch to another terminal window. And there...

```
$ ollama --version
ollama version is 0.13.5
$ curl http://localhost:11434
Ollama is running
```

Now, pull a model - after consulting with Gemini, I chose `deepseek-r1:8b` as about the best model that would run reasonably on my 16GB 2022 M2 MacBook Air.

It pulled about 5.2GB:

```
$ ollama pull deepseek-r1:8b
pulling manifest 
pulling e6a7edc1a4d7: 100% ▕███████████████████████████████████████████▏ 5.2 GB
pulling c5ad996bda6e: 100% ▕███████████████████████████████████████████▏  556 B
pulling 6e4c38e1172f: 100% ▕███████████████████████████████████████████▏ 1.1 KB
pulling ed8474dc73db: 100% ▕███████████████████████████████████████████▏  179 B
pulling f64cd5418e4b: 100% ▕███████████████████████████████████████████▏  487 B
verifying sha256 digest 
writing manifest 
success
```

Continue with VS Code
---------------------

Download and install [VS Code](https://code.visualstudio.com/download), I prefer IDEs from JetBrains but while there is a JetBrains Continue plugin, the one for VS Code came first so I thought I'd try it initially on the assumption that it's probably the most mature.

Then install the [Continue plugin](https://marketplace.visualstudio.com/items?itemName=Continue.continue).

Open some existing project that you have, e.g. a small Python project and then open one of the your `.py` files.

VS Code is quite keen on pushing its own AI coding chat feature so close that first if its open.

Click the new Continue icon in the left-side VS Code gutter.

Instead of the login options, opt for the stay local option. It'll show options for entering API keys for the major cloud model providers.

Instead, select the option for other providers and select Ollama.

For the _Install Provider_ step, you don't have to do anything - the provided link to Ollama is just needed if you haven't already installed it.

The model dropdown should be set to _autodetect_, so just press _Continue_ and it'll find Ollama and the `deepseek-r1:8b` model.

Select a function in your open `.py` file, and press command-L - this will copy the code to the Continue chat window.

Then ask it to explain the code. Your machine will have a small heartattack but after a few seconds it'll explain things.

My machine became very sluggish even after the reply was completed - basically you have to exit everything non-essential to have enough RAM for this model.

Quitting VS Code and killing `ollama server` with control-C brought my system back to life.

Configuration
-------------

If you click the cog wheel icon at the top-right of the Continue chat window, then select the models section (the icon that's an isometric view of a cube) and then click the cog wheel beside the dropdown showing `deepseek-r1:8b`, it'll open a `config.yaml` file containing:

```
name: Local Config
version: 1.0.0
schema: v1
models:
  - name: Autodetect
    provider: ollama
    model: AUTODETECT
```

This is basically the autodetect configuration described [here](https://docs.continue.dev/guides/ollama-guide#method-2:-using-autodetect) except it doesn't have the `roles` property,

According to the [documentation](https://docs.continue.dev/reference#models), if the `roles` property is missing, it defaults to `[chat, edit, apply, summarize]`.

This means that `[autocomplete, embed, rerank]` are not enabled by default. `embed` and `rerank` sounds fairly obscure (see [here](https://docs.continue.dev/customize/model-roles/00-intro)) but autocomplete is obviously useful but needs a model that's been trained for "Fill-in-the-Middle" (FIM) - Qwen Coder is the open-source model recommended by Continue.

Note: [`qwen3-coder`](https://ollama.com/library/qwen3-coder) has a 480 billion parameter version that requires 250GB of memory but also has a cloud variant - something that `deepseek-r1` and other open source models don't have.
