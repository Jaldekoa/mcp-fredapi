# MCP-FREDAPI

**FRED (Federal Reserve Economic Data) API integration with Model Context Protocol (MCP)**

## Table of Contents

- [Introduction](#introduction)
- [Installation](#installation)
- [Configuration](#configuration)
  - [FRED API Key](#fred-api-key)
  - [Claude/Cursor Configuration](#claudecursor-configuration)
- [Available Tools](#available-tools)
- [Parameters](#parameters)
- [Examples](#examples)
  - [Common Economic Series IDs](#common-economic-series-ids)
- [Contributing](#contributing)
- [License](#license)
- [References](#references)

## Introduction

MCP-FREDAPI provides access to economic data from the Federal Reserve Bank of St. Louis (FRED) through the Model Context Protocol. This integration allows AI assistants like Claude to retrieve economic time series data directly when used with Cursor or other MCP-compatible environments.

This package integrates with the [official FRED API](https://fred.stlouisfed.org/docs/api/fred/), focusing specifically on the [series_observations endpoint](https://fred.stlouisfed.org/docs/api/fred/series_observations.html) which provides time series data for economic indicators.

## Installation

There are two installation methods:

### Method 1: Using pip

Install the required dependencies:

```terminal
pip install "mcp[cli]" httpx python-dotenv
```

Clone this repository:

```terminal
git clone https://github.com/Jaldekoa/mcp-fredapi.git
cd mcp-fredapi
```

### Method 2: Using uv (Recommended)

This method is recommended as it matches the configuration shown in mcp.json.

1. First, install uv if you don't have it yet:

```terminal
pip install uv
```

2. Clone this repository:

```terminal
git clone https://github.com/Jaldekoa/mcp-fredapi.git
cd mcp-fredapi
```

3. Use uv to run the server (no need to install dependencies separately):

```terminal
uv run --with mcp --with httpx mcp run server.py
```

## Configuration

### FRED API Key

You'll need a FRED API key, which you can obtain from [FRED API](https://fred.stlouisfed.org/docs/api/api_key.html).

Create a `.env` file in the project root:

```
FRED_API_KEY=your_api_key_here
```

### Claude/Cursor Configuration

To configure Cursor to use this MCP server, add the following to your `~/.cursor/mcp.json` file:

```json
{
  "mcpServers": {
    "mcp-fredapi": {
      "command": "uv",
      "args": ["--directory", "/path/to/mcp-fredapi", "run", "--with", "mcp", "--with", "httpx", "mcp", "run", "server.py"]
    }
  }
}
```

Replace `/path/to/mcp-fredapi` with the actual path to the repository on your system. For example:

```json
{
  "mcpServers": {
    "mcp-fredapi": {
      "command": "uv",
      "args": ["--directory", "/path/to/mcp-fredapi", "run", "--with", "mcp", "--with", "httpx", "mcp", "run", "server.py"]
    }
  }
}
```

Note: On Windows, you can use either forward slashes `/` or double backslashes `\\` in the path.

## Available Tools

### get_fred_series_observations

Retrieves economic time series observations from FRED.

When using Claude in Cursor, you can access this tool directly with:

```
@mcp-fredapi:get_fred_series_observations
```

## Parameters

The `get_fred_series_observations` tool accepts the following parameters. For complete technical details about each parameter, please refer to the [official FRED API documentation](https://fred.stlouisfed.org/docs/api/fred/series_observations.html).

| Parameter | Type | Description | Allowed Values | Default Value | Status |
|-----------|------|-------------|---------------|---------------|--------|
| series_id | str | The ID of the economic series | - | (Required) | ✅ Works |
| sort_order | str | Sort order of observations | 'asc', 'desc' | 'asc' | ✅ Works |
| units | str | Data value transformation | 'lin', 'chg', 'ch1', 'pch', 'pc1', 'pca', 'cch', 'cca', 'log' | 'lin' | ✅ Works |
| frequency | str | Frequency of observations | 'd', 'w', 'bw', 'm', 'q', 'sa', 'a', 'wef', 'weth', 'wew', 'wetu', 'wem', 'wesu', 'wesa', 'bwew', 'bwem' | None | ✅ Works |
| aggregation_method | str | Aggregation method for frequency | 'avg', 'sum', 'eop' | 'avg' | ✅ Works |
| output_type | int | Output type of observations | 1, 2, 3, 4 | 1 | ✅ Works |
| realtime_start | str | Start of real-time period (YYYY-MM-DD) | - | None | ❌ Not working |
| realtime_end | str | End of real-time period (YYYY-MM-DD) | - | None | ❌ Not working |
| limit | int/str | Maximum number of observations to return | Between 1 and 100000 | 10 | ❌ Not working |
| offset | int/str | Number of observations to skip from the beginning | - | 0 | ❌ Not working |
| observation_start | str | Start date of observations (YYYY-MM-DD) | - | None | ❌ Not working |
| observation_end | str | End date of observations (YYYY-MM-DD) | - | None | ❌ Not working |
| vintage_dates | str | Comma-separated list of vintage dates | - | None | ❌ Not working |

> [!WARNING] **Note on Parameter Compatibility**
> 
> Due to current limitations with the MCP implementation, only certain parameters are working properly:
> - ✅ **Working parameters**: `series_id`, `sort_order`, `units`, `frequency` , aggregation_method`, and `output_type`.
> - ❌ **Non-working parameters**: `realtime_start`, `realtime_end`, `limit`, `offset`, `observation_start`, `observation_end`, and `vintage_dates`.
>
>For best results, stick with the working parameters in your queries. Future updates may resolve these limitations.

## Examples

### Getting US GDP Data

When using Claude in Cursor, you can ask for GDP data like this:

```
Can you get the latest GDP data from FRED?

@mcp-fredapi:get_fred_series_observations
{
  "series_id": "GDP"
}
```

### Getting GDP Data in Descending Order

```
Can you get the GDP data in descending order (newest first)?

@mcp-fredapi:get_fred_series_observations
{
  "series_id": "GDP",
  "sort_order": "desc"
}
```

### Getting Annual GDP Data

```
Can you get annual GDP data?

@mcp-fredapi:get_fred_series_observations
{
  "series_id": "GDP",
  "frequency": "a"
}
```

### Getting Inflation Rate

To get consumer price index data with percent change:

```
What's the recent inflation rate in the US?

@mcp-fredapi:get_fred_series_observations
{
  "series_id": "CPIAUCSL",
  "units": "pch",
  "frequency": "m"
}
```

### Different Output Format

```
Show me GDP data in a different format.

@mcp-fredapi:get_fred_series_observations
{
  "series_id": "GDP",
  "output_type": 2
}
```

## Contributing

Contributions are welcome. Please follow these steps:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Make your changes
4. Commit your changes (`git commit -m 'Add an amazing feature'`)
5. Push to the branch (`git push origin feature/amazing-feature`)
6. Open a Pull Request

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## References

- [FRED API Documentation - Series Observations](https://fred.stlouisfed.org/docs/api/fred/series_observations.html) - Official documentation for the FRED API endpoint used in this project.
- [FRED API](https://fred.stlouisfed.org/docs/api/api_key.html) - Information on obtaining an API key and general API documentation.
- [Model Context Protocol](https://modelcontextprotocol.github.io/) - Documentation for the Model Context Protocol.
