# AGENTS.md - Fulcra

> Fulcra: a user-owned context store shared by humans and their agents.

## About

[Fulcra](https://docs.fulcradynamics.com/) is a place for a person and their AI agents to store, share, and act on context. It holds:

- **Data you and your agents write**: files, custom record types, annotations (notes, moods, counts, anything the user wants to log), and agent memory and preferences.
- **Data collected for you**: location, calendars, and phone and device data, synced by the [Fulcra iOS app](https://apps.apple.com/us/app/context-personal-data-kit/id1633037434) and connected services.

The data belongs to the user, not to any one agent, so several agents can share it and use it to coordinate over time. The user can also browse it in the [web app](https://context.fulcradynamics.com/).

### Agentic/Programmatic Access
* [Main developer docs](https://docs.fulcradynamics.com). For agents: [agent-get-started.txt](https://docs.fulcradynamics.com/agent-get-started.txt)
* [MCP server](https://mcp.fulcradynamics.com/mcp): public endpoint, Streamable HTTP transport with OAuth2 authorization. Source: [fulcradynamics/fulcra-context-mcp](https://github.com/fulcradynamics/fulcra-context-mcp).
* [Agent skills](https://github.com/fulcradynamics/agent-skills): ready-made skills for onboarding, memory backup, custom data tracking, importing data exports, situational awareness, and agent teams. Install with `npx skills add fulcradynamics/agent-skills`.
* [Python client library and CLI](https://fulcradynamics.github.io/fulcra-api-python/) (`pip install fulcra-api`, or `uvx fulcra-api --help` for the CLI). Handles authentication for you.
* [OpenAPI spec](https://api.fulcradynamics.com/openapi.json)

### For Agents and LLMs: Authorization Tips

#### Code-first agents

If you can run shell commands or Python, use the `fulcra` CLI or the `fulcra-api` Python module. When calling `.authorize()`, the output will include a URL that you can send to the operator (the user). The user then opens that link in a browser and authorizes the API instance to make calls. This uses the OAuth2 Device Authorization Flow.

Example interactive session (`fulcra-api` module must be installed first):
```
Python 3.14.2 (main, Dec  9 2025, 19:29:30) [Clang 21.1.4 ] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> from fulcra_api.core import FulcraAPI
>>> fulcra = FulcraAPI()
>>> fulcra.authorize()

            Use your browser to log in to Fulcra.  If the tab does not open
            automatically, visit this URL to authenticate: https://fulcra.us.auth0.com/activate?user_code=DBNV-DBQV
```

The `authorize()` call will poll while the user reaches the URL. If the call times out, call `authorize()` again to get a new URL.

#### Text-first agents

For agents that can't run code, use the [MCP server](https://github.com/fulcradynamics/fulcra-context-mcp). Its tools reach the same data the API does.

The user can either use the public MCP server at `https://mcp.fulcradynamics.com/mcp`, or run it locally. It is published as the `fulcra-context-mcp` PyPI module.

You can run it locally (stdio transport) with `uvx fulcra-context-mcp@latest`. See the [PyPI page](https://pypi.org/project/fulcra-context-mcp/) for more docs.

#### MCP Client Configuration Examples

Remote connection using proxy (for clients that only support stdio):
```json
{
    "mcpServers": {
        "fulcra_context": {
            "command": "npx",
            "args": [
                "-y",
                "mcp-remote",
                "https://mcp.fulcradynamics.com/mcp"
            ]
        }
    }
}
```

Local connection using `uvx`:
```json
{
    "mcpServers": {
        "fulcra_context": {
            "command": "uvx",
            "args": [
                "fulcra-context-mcp@latest"
            ]
        }
    }
}
```

## MCP tools and tips

Start with discovery tools to see what the user has: `get_user_info`, `get_data_catalog`, `annotations_catalog`, `list_files`, and `get_data_updates` (what changed since you last looked). Then read and write with the specific tools:

- **Files**: `list_files`, `read_file`, `write_file`, `delete_file`, `restore_file`
- **Records and custom data types**: `get_records`, `record_data`, `create_data_type`, `archive_data_type`, `restore_data_type`
- **Sharing and groups**: `create_share`, `list_shares`, `get_groups`, `create_group`, `join_group`
- **Collected data**: `get_time_series`, `get_location_at_time`, `get_location_time_series`, `get_calendars`, `get_calendar_events`, `get_sleep`, `get_workouts`

All time parameters must include time zones (ISO 8601 format). Always translate result timestamps to the user's local time zone when known.

## Available Data

### Files and Custom Records
Arbitrary files (notes, agent memory, documents, exports) stored in the user's Fulcra file space, and custom record types that agents or users define with a schema and then write records to. Use these for anything an agent should remember or share across sessions and across agents.

### Annotations
User-logged events and values: moods, notes, medications, habits, counts, and custom events. They come in moment, duration, boolean, numeric, and scale flavors, and can be correlated with any other stream.

### Location and Calendar
Historical location data (both `CLLocationUpdate` frequent GPS pings and `CLVisit` place-based time ranges) and calendar events from the user's connected calendars.

### Device and Activity Data
Time series and samples synced from the phone, wearables, and connected services: steps, activity, workouts, sleep, heart rate, and more. Call `get_data_catalog` (MCP) or `metrics_catalog()` (Python) for the full list.

## Best Practices for Agents

- **Write things down.** Fulcra is a shared, durable place to store context. If something should outlive the session or be visible to the user's other agents, store it as a file or record.
- **Check what's new.** Use `get_data_updates` (or `fulcra data-updates`) to notice new data, files, and messages since your last loop.
- **Use appropriate sample rates.** When querying time series data, choose a sample rate that balances resolution with performance. For daily overviews, 3600 seconds (hourly) works well. For detailed analysis, use 60-300 seconds.
- **Ranges can span midnight.** Overnight data (e.g. sleep) starts on day N and ends on day N+1; extend your date range accordingly.
- **Correlate across domains.** The value of Fulcra comes from combining streams: calendar with location, annotations with activity, agent notes with what actually happened.

### Example: Querying Data with the Python Client

```python
from fulcra_api.core import FulcraAPI

fulcra = FulcraAPI()
fulcra.authorize()

# Discover available metrics
catalog = fulcra.metrics_catalog()

# Get step count for a day (hourly resolution) as a list of dicts
rows = fulcra.metric_time_series_rows(
    metric="StepCount",
    start_time="2025-01-01T00:00:00-08:00",
    end_time="2025-01-02T00:00:00-08:00",
    sample_rate=3600
)

# Or as a pandas DataFrame (requires `pip install "fulcra-api[pandas]"`)
df = fulcra.metric_time_series(
    metric="StepCount",
    start_time="2025-01-01T00:00:00-08:00",
    end_time="2025-01-02T00:00:00-08:00",
    sample_rate=3600
)
```

The base `fulcra-api` package doesn't depend on pandas. The DataFrame-returning methods (`metric_time_series`, `sleep_cycles`, `sleep_stages`, `sleep_agg`) need the `pandas` extra; their `*_rows` counterparts don't.

### Jupyter Notebook Demos

Ready-to-run demo notebooks are in this repository, the [Fulcra demos repository](https://github.com/fulcradynamics/demos). They walk through connecting, reading time series, annotations, calendars, and location, and correlating data across sources. They can also be opened directly in [Google Colab](https://colab.research.google.com/) for one-click, zero-install demos.

## Support

- **Email:** support@fulcradynamics.com
- **Discord:** [Context Social Discord](https://discord.gg/fulcra)
- **GitHub:** [github.com/fulcradynamics](https://github.com/fulcradynamics)
- **Live Web Chat:** Available on fulcradynamics.com

## Official domains
* fulcradynamics.com
* context.fulcradynamics.com
* mcp.fulcradynamics.com
* fulcra.ai
