# Fulcra Dynamics Demos

Jupyter notebooks showing how to work with your data in [Fulcra](https://docs.fulcradynamics.com/) from Python.

Fulcra is a user-owned context store for you and your AI agents. It collects data from your phone, apps, and devices (location, calendars, activity, and more). It also holds your own annotations, files, and custom records, which you or your agents can write. These notebooks read that data with the [`fulcra-api`](https://fulcradynamics.github.io/fulcra-api-python/) Python library. Everything they show is also available to agents through the MCP server and agent skills below.

To get started quickly in Colab, click here: <a target="_blank" href="https://colab.research.google.com/github/fulcradynamics/demos/blob/main/notebooks/00_Hello_Fulcra.ipynb">
  <img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/>
</a>

The notebooks are in the [notebooks directory](notebooks/).

## Connecting agents

- **[Fulcra MCP server](https://github.com/fulcradynamics/fulcra-context-mcp)**: gives Claude, ChatGPT, Codex, and other MCP clients access to your Fulcra data. Point your client at `https://mcp.fulcradynamics.com/mcp`, or run it locally with `uvx fulcra-context-mcp@latest`.
- **[Fulcra agent skills](https://github.com/fulcradynamics/agent-skills)**: teach your agent to use Fulcra for memory backup, custom data tracking, importing data exports, and coordinating with other agents. Install them with `npx skills add fulcradynamics/agent-skills`.
- **`fulcra` CLI**: ships with `fulcra-api`. Run `uvx fulcra-api --help`.

To connect any agent, paste this into it: `Use the following instructions to connect to Fulcra: https://docs.fulcradynamics.com/agent-get-started.txt`

## About `fulcra-api` and pandas

Recent versions of the base `fulcra-api` package no longer depend on pandas. The DataFrame-returning methods (`metric_time_series`, `sleep_cycles`, `sleep_stages`, `sleep_agg`) need the optional `pandas` extra, which is what these notebooks install:

```
pip install "fulcra-api[pandas]"
```

If you don't want pandas, each of those methods has a `*_rows` counterpart (for example `metric_time_series_rows`) that returns a plain list of dicts.

## More

- Developer docs: [https://docs.fulcradynamics.com](https://docs.fulcradynamics.com/)
- Python library guide and API reference: [https://fulcradynamics.github.io/fulcra-api-python/](https://fulcradynamics.github.io/fulcra-api-python/)

All code here is covered under the [Apache 2.0 license](LICENSE).
