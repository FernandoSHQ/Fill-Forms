# Ops Tools

A collection of shell scripts for searching and analyzing logs across ShipperHQ infrastructure, including both archived logs on instances and real-time logs in Loki endpoints.

## Getting Started (for support agents)

1) Clone the repo  
```bash
git clone https://github.com/FernandoSHQ/Fill-Forms.git
cd Fill-Forms
```

2) Install Python deps (if you plan to run `fill_form.py` or `loki_search.py`)  
```bash
pip install -r requirements.txt
```

3) Pull the latest fixes anytime  
```bash
git pull
```

4) Run the rate report helper (`fill_form.py`)  
- With a log file you already have:  
  ```bash
  python3 fill_form.py /path/to/logfile.log
  ```  
- With just a transaction ID (auto-fetches via `findlog.sh` and picks newest shipperws log):  
  ```bash
  python3 fill_form.py SHQ_YYYYMMDD_xxxxxxx
  ```
Output saves next to the log as `rate_analysis_<timestamp>.txt`.

## Main Scripts

### `archivelog.sh` - Search Archived Logs

Searches historical archived logs (compressed .gz files) and Loki endpoints for specified keywords or transaction IDs.

**Note:** If you have a transaction ID the findlog script will find the log faster, as the tID contains the timestamp.

**Usage:**
```bash
./archivelog.sh <domainName> <yyyy-MM> <transactionID>
```

**Example:**
```bash
./archivelog.sh example.com 2017-08 SHQ_20170801_1234567
```

**Options:**
- `-x` - Skip log trimmer and log splitter
- `-nl` - Disable Loki search

**Features:**
- Searches archived logs from specific month/year
- Queries multiple Loki endpoints (Oregon, Virginia, Ireland)
- Automatically applies log trimming and splitting for transaction IDs
- Handles time-based searches for transaction IDs
- Downloads and processes logs in batches

### `findlog.sh` - Search Current Logs

Searches current logs from Loki endpoints. Supports two modes of operation:

**Mode 1 - Transaction ID Search:**
```bash
./findlog.sh <transactionID>
```
Searches across all domains for the specified transaction ID.

**Mode 2 - Domain + Keyword Search:**
```bash
./findlog.sh <domainName> <keyword>
```
Searches logs for a specific domain with the given keyword.

**Options:**
- `-x` - Bypass log trimmer and log splitter
- `-nl` - Disable Loki search
- `-t=<time_range>` - Specify time range for Loki search (e.g., `-t=2h`, `-t=30m`, `-t=1d`)
- `-to` - Use Oredev (test environment)
- `-tv` - Use Virdev (test environment)
- `-ti` - Use Iredev (test environment)

**Features:**
- Searches Loki endpoints
- Time-based search optimization for transaction IDs
- Chunked queries for large time ranges (>10 days)
- Automatic log processing and splitting
- VPN connectivity detection

### `loki_find_store_url.sh` - Find Store URLs

Searches Loki endpoints to find store_url values matching a given pattern.

**Usage:**
```bash
./loki_find_store_url.sh <search_pattern>
```

**Examples:**
```bash
./loki_find_store_url.sh thebreather
./loki_find_store_url.sh myshopify
./loki_find_store_url.sh '.*\.com$'
```

**Options:**
- `-t=<time_range>` - Specify time range (default: 24h)
- `-d` - Use dev endpoint instead of prod endpoints

**Features:**
- Searches across all Loki endpoints
- Pattern matching with regex support
- Returns unique store_url values
- Configurable time ranges

## Helper Scripts

The following scripts are automatically called by the main scripts and don't need to be run manually:

- **`log_trimmer.sh`** - Trims logs around transaction IDs for focused analysis
- **`split_logs.sh`** - Splits and processes log files for better organization

## Prerequisites

- **VPN Connection**: Required for accessing Loki endpoints
- **jq**: Required for JSON parsing of Loki responses

## Output

All scripts create logs in the `logs/` directory structure:
- `logs/<domain_name>/` - Contains processed log files
- Files are named with descriptive patterns including keywords, timestamps, and sources

## Common Use Cases

1. **Debug Transaction Issues:**
   ```bash
   ./findlog.sh SHQ_20240115_1234567
   ```

2. **Search Domain-Specific Errors:**
   ```bash
   ./findlog.sh example.com "error 500"
   ```

3. **Find Historical Issues:**
   ```bash
   ./archivelog.sh example.com 2024-01 SHQ_20240115_1234567
   ```

4. **Discover Store URLs:**
   ```bash
   ./loki_find_store_url.sh shopify
   ```

## Notes

- Transaction IDs must be at least 5 characters long
- Domain names must be at least 5 characters long
- IP address searches are not supported in `archivelog.sh`
- Log trimming is automatically applied for transaction ID searches
- The tools support both macOS and Linux environments

-----

# Loki Search Tool

A fast and reliable command-line tool for searching logs in Grafana Loki with keyword or regex patterns. Features intelligent batching, parallel execution across multiple endpoints, retry logic, and comprehensive error handling.

## Features

- **Multi-endpoint search**: Searches multiple Loki instances in parallel (ShipperHQ prod/dev environments)
- **Fast searches**: Splits large time ranges into batches to avoid timeouts
- **Parallel execution**: Configurable concurrency across endpoints and time batches
- **Robust error handling**: Automatic retry with exponential backoff and jitter
- **Flexible search modes**: Keywords with contains/regex patterns
- **Multiple keyword logic**: OR (default) or AND operations
- **Progress reporting**: Real-time progress with rate and ETA across all endpoints
- **Dry-run mode**: Preview queries before execution
- **Easy time ranges**: Relative times like "4h" or absolute timestamps
- **Environment selection**: Built-in support for ShipperHQ prod and dev environments

## Installation

1. Ensure you have Python 3.7+ installed
2. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

## Usage

### Basic Examples

```bash
# Search for "error" in the last 4 hours (default) across all prod endpoints
./loki_search.py "error"

# Search for "error" OR "timeout" in the last 4 hours
./loki_search.py --relative "4h" "error" "timeout"

# Search for "error" AND "database" with AND logic
./loki_search.py --and "error" "database"

# Search dev environment (single endpoint)
./loki_search.py --env dev "error"

# Regex search for errors followed by timeouts
./loki_search.py --mode regex "error.*timeout"

# Search specific time range
./loki_search.py --start "2023-01-01 10:00" --end "2023-01-01 18:00" "error"

# Show planned queries without executing (dry run)
./loki_search.py --dry-run --relative "2h" "error"
```

### Environment Examples

```bash
# Search production (default) - searches 3 endpoints in parallel
./loki_search.py "error"

# Search development environment - single endpoint
./loki_search.py --env dev "error"

# Custom single endpoint
./loki_search.py --loki-url "https://custom-loki.com:3100" "error"

# Multiple custom endpoints
./loki_search.py --loki-url "https://loki1.com:3100" --loki-url "https://loki2.com:3100" "error"
```

### Advanced Examples

```bash
# Search with label filters
./loki_search.py --label "app=frontend" --label "env=prod" "error"

# High concurrency for faster searches across multiple endpoints
./loki_search.py --concurrency 15 --relative "24h" "error"

# Smaller batch sizes (2-hour batches instead of 4-hour default)
./loki_search.py --batch-size 7200 "error"

# JSON output for programmatic processing
./loki_search.py --output-format json "error"

# Verbose output with progress details and endpoint breakdown
./loki_search.py --verbose --relative "6h" "error"
```

## Command Line Options

### Required Arguments
- `keywords`: One or more keywords to search for

### Search Options
- `--env {prod,dev}`: Environment to search (default: prod)
- `--mode {contains,regex}`: Search mode (default: contains)
- `--and`: Use AND logic for multiple keywords (default: OR)

### Time Range Options
- `--relative TIME`: Relative time like "4h", "2d", "30m"
- `--start TIMESTAMP`: Start timestamp (various formats supported)
- `--end TIMESTAMP`: End timestamp (various formats supported)

### Loki Configuration
- `--loki-url URL`: Custom Loki URL (can be specified multiple times). If not provided, uses environment defaults.
- `--label LABEL`: Label selector (can be specified multiple times)

**Default Endpoints:**
- **Production**: 3 ShipperHQ endpoints (Oregon, Virginia, Ireland)
- **Development**: 1 ShipperHQ dev endpoint (Oregon Dev)

### Performance Options
- `--batch-size SECONDS`: Batch size in seconds (default: 14400 = 4 hours)
- `--concurrency N`: Max concurrent requests (default: 5)
- `--timeout SECONDS`: Request timeout (default: 30)
- `--max-retries N`: Maximum retries (default: 3)

### Output Options
- `--dry-run`: Show planned queries without executing
- `--verbose`: Verbose progress output
- `--output-format {text,json}`: Output format (default: text)
- `--limit N`: Max log lines per batch (default: 1000)

## Time Range Formats

### Relative Times
- `4h` - Last 4 hours
- `2d` - Last 2 days  
- `30m` - Last 30 minutes
- `last 4h` - Also supported with "last" prefix

### Absolute Times
- `2023-01-01` - Date only (time defaults to 00:00:00)
- `2023-01-01 10:00` - Date and time
- `2023-01-01 10:00:00` - Full timestamp
- `2023-01-01T10:00:00Z` - ISO format with timezone

## Label Selectors

Labels help filter logs to specific applications or environments:

```bash
# Single label
./loki_search.py --label "app=frontend" "error"

# Multiple labels (AND logic for labels)
./loki_search.py --label "app=frontend" --label "env=prod" "error"

# Complex label expressions
./loki_search.py --label 'job=~".*server.*"' "error"
```

## Search Modes

### Contains Mode (Default)
Searches for exact string matches:
```bash
./loki_search.py "database error"  # Finds logs containing "database error"
```

### Regex Mode
Uses regular expressions for pattern matching:
```bash
./loki_search.py --mode regex "error.*timeout"     # Error followed by timeout
./loki_search.py --mode regex "(error|fail)"       # Error OR fail
./loki_search.py --mode regex "user_id=\d+"        # Numeric user IDs
```

## Performance Tuning

### Batch Size
- **Larger batches** (e.g., 7200s = 2 hours): Fewer requests, but risk of timeouts
- **Smaller batches** (e.g., 1800s = 30 minutes): More requests, but more reliable

### Concurrency
- **Higher concurrency** (e.g., 10-15): Faster for large searches, more load on Loki
- **Lower concurrency** (e.g., 3-5): Gentler on Loki, good for shared environments

### Example for large time ranges:
```bash
# Search last 7 days with optimized settings
./loki_search.py --relative "7d" --batch-size 3600 --concurrency 8 "error"
```

## Output Formats

### Text Format (Default)
```
2023-01-01 10:15:23 | ERROR: Database connection failed
2023-01-01 10:15:22 | ERROR: Timeout occurred
```

### JSON Format
```json
{"timestamp": "1672574123000000000", "line": "ERROR: Database connection failed", "labels": {"app": "frontend"}}
```

## Error Handling

The tool provides structured error messages with suggestions:

```bash
Error: Failed to connect to Loki at http://localhost:3100: Connection refused
Suggestion: Check the URL, network connectivity, and Loki status

Error: Loki server error (HTTP 500): Query timeout
Suggestion: Try reducing batch size or time range

Error: Rate limited by Loki (HTTP 429)
Suggestion: Reduce concurrency or wait before retrying
```

## Troubleshooting

### Connection Issues
1. Verify Loki URL: `curl http://localhost:3100/ready`
2. Check network connectivity
3. Verify Loki is running and accessible

### Timeout Issues
1. Reduce batch size: `--batch-size 1800`
2. Reduce concurrency: `--concurrency 3`
3. Limit results per batch: `--limit 500`

### Performance Issues
1. Use label filters to reduce search scope
2. Increase concurrency for faster searches
3. Use regex mode carefully (can be slower)

### Memory Issues
1. Reduce batch size
2. Reduce limit per batch
3. Process results in smaller time windows

## Architecture

The tool is structured with clear separation of concerns:

- **CLI module** (`cli.py`): Argument parsing and validation
- **Time utilities** (`time_utils.py`): Time range handling and batching
- **Loki client** (`loki_client.py`): API interactions with retry logic
- **Executor** (`executor.py`): Parallel query execution
- **Progress reporting** (`progress.py`): User feedback and output formatting
- **Main module** (`main.py`): Orchestration and error handling
