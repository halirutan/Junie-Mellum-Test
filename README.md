# Free AI coding with Junie Lite and Mellum 2.1

A minimal Python template for building a desktop calculator one milestone at a time. **Junie Lite** plans and reviews the work; a **Mellum 2.1** worker implements small tasks locally through Ollama. **No paid AI subscription or API credits** are needed.
Junie Lite runs on JetBrains infrastructure and requires a free JetBrains account; Mellum uses your own hardware and electricity.

**A word of caution:** This here is specifically to test if a local model can be used as a simple worker subagent for trivial tasks
that is controlled by a more capable model. Additionally, I wanted to know if these small models can use MCP tools consistently.
The main agent basically hand-feeds the worker everything it needs to do and by no means can this be used in a real-world project yet.
But if you are curious and interested in this technology, it's a nice showcase, because agents, subagents, commands, skills, mcp definitions,
and model selection works very similar in most coding agents nowadays (with the usual caveat that the devil is in the details).

## 1. Clone and install

You need Linux, an NVIDIA graphics card with enough VRAM to run the local model on the GPU, Git, [uv](https://docs.astral.sh/uv/getting-started/installation/) for managing the Python environment, and PyCharm 2025.2 or newer.

```bash
git clone <REPOSITORY_URL>
cd Junie-Mellum-Test
uv sync
curl -fsSL https://junie.jetbrains.com/install.sh | bash
```

See the [Junie CLI installation guide](https://junie.jetbrains.com/docs/junie-cli.html).

## 2. Install Air to use Junie in the IDE

The Air plugin lets you use any coding agent directly in the IDE, either in the TUI you're probably used to or in graphical mode
where your agent is controlled through the [agent client protocol (ACP)](https://agentclientprotocol.com/get-started/introduction).

1. Open **Settings → Plugins**, search for [Air](https://plugins.jetbrains.com/plugin/33314-air), and install it if it is not already installed. See [Managing plugins](https://www.jetbrains.com/help/pycharm/managing-plugins.html).
2. Open the **Air** tool window and select **Manage Agents…** from the **New Session** drop-down menu.
3. Enable Junie in **Terminal** mode if you prefer a terminal interface.
4. Go to **Settings → Tools → Air → Providers** and sign in with your JetBrains account. Junie Lite requires no subscription.

You can now start a Junie session in the IDE. Before building the calculator, set up the local Mellum worker.

## 3. Run Mellum2.1 with Ollama

Mellum2.1 is fast coding model that can be run locally and was [announced in October 2026](https://blog.jetbrains.com/ai/2026/10/mellum2-1-gets-to-work-a-fast-open-model-for-coding-agents/).
But before you can use it, you need to install [Ollama for Linux](https://docs.ollama.com/linux) and download the model used by this project's worker:

```bash
curl -fsSL https://ollama.com/install.sh | sh
sudo systemctl start ollama
ollama pull hf.co/JetBrains/Mellum2.1-12B-A2.5B-Thinking-GGUF:Q4_K_M
```
To be clear, `ollama` is just ONE way to run the model, but it's pretty easy to set up and I already had `ollama` on my machine.
The model download is approximately 8.1 GB; inference needs additional memory. To allocate its full 128K context window, run `sudo systemctl edit ollama` and add:

```ini
[Service]
Environment="OLLAMA_CONTEXT_LENGTH=131072"
```

Apply the change:

```bash
sudo systemctl daemon-reload
sudo systemctl restart ollama
```

If memory is insufficient, use a smaller context such as `65536`. This is only 6.25% of a 1M-token context window, so keep worker tasks small and provide only the context they need.

Once the Ollama service is running, test the model in a terminal:

```bash
ollama run hf.co/JetBrains/Mellum2.1-12B-A2.5B-Thinking-GGUF:Q4_K_M
```

Send a prompt, then run `ollama ps` in another terminal to check the allocated context and CPU/GPU usage. For example:

```text
NAME                      ID            SIZE    PROCESSOR  CONTEXT  RUNNER    UNTIL
...JetBrains/Mellum2.1...  df6baa3ade7e  8.2 GB  100% GPU   131072   llamacpp  4 minutes from now
```

The included [.junie/models/mellum-local.json](.junie/models/mellum-local.json) connects to Ollama at `http://localhost:11434/v1/chat/completions`. The [worker configuration](.junie/agents/worker.md) selects it with `model: custom:mellum-local`.

## 3. Configure PyCharm MCP

Open the cloned repository in PyCharm, then:

1. Go to **Settings → Tools → MCP Server**, select **Enable MCP Server**, confirm, and click **Apply**. If this page is missing, enable the bundled **MCP Server** plugin under **Settings → Plugins**.
2. In the **Manual Client Configuration** section, click **Copy HTTP Stream Config**.
3. Use the copied configuration to update the connection details in [.junie/mcp/mcp.json](.junie/mcp/mcp.json). The checked-in port and project path are machine-specific; `IJ_MCP_SERVER_PROJECT_PATH` must point to your clone.
4. Under **Settings → Tools → MCP Server → Exposed Tools**, clear **Enabled** for every unused tool, including database tools. Keep the file, search, editing, refactoring, inspection, build, and run configuration tools referenced in [AGENTS.md](AGENTS.md) enabled.
5. Restart Junie to load the configuration. Keep PyCharm running with this project open.

See the [PyCharm MCP setup guide](https://www.jetbrains.com/help/pycharm/mcp-server.html#external-client-setup).

## 4. Build the calculator step by step

The calculator requirements are in [docs/calculator_spec.md](docs/calculator_spec.md); milestones are in [docs/implementation-plan.md](docs/implementation-plan.md).

For a fresh run, remove the existing **Status** and **Completed** entries from the implementation plan, or restore the plan to its initial repository version. Remove any calculator code and tests generated by earlier runs so the exercise starts from an empty template.

Start a new Junie session and select **Junie Lite** with `/model`. Then run:

```text
/next-task
```

You can read the command's instructions in [.junie/commands/next-task.md](.junie/commands/next-task.md).

Junie delegates small tasks to the local worker, verifies one milestone, marks it done, and stops. Review the changes, then use `/new` to start a session with a fresh context window and run `/next-task` again for the next milestone. Repeat until all milestones are complete.
