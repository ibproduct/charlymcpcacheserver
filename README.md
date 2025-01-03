# Memory Cache Server

A Model Context Protocol (MCP) server that reduces token consumption by efficiently caching data between language model interactions. Works with any MCP client and any language model that uses tokens.

## Installation

1. Clone the repository:
```bash
git clone git@github.com:ibproduct/ib-mcp-cache-server
cd ib-mcp-cache-server
```

2. Install dependencies:
```bash
npm install
```

3. Build the project:
```bash
npm run build
```

4. Add to your MCP client settings:
```json
{
  "mcpServers": {
    "memory-cache": {
      "command": "node",
      "args": ["/path/to/ib-mcp-cache-server/build/index.js"]
    }
  }
}
```

5. The server will automatically start when you use your MCP client

## Verifying It Works

When the server is running properly, you'll see:
1. A message in the terminal: "Memory Cache MCP server running on stdio"
2. Improved performance when accessing the same data multiple times
3. No action required from you - the caching happens automatically

You can verify the server is running by:
1. Opening your MCP client
2. Looking for any error messages in the terminal where you started the server
3. Performing operations that would benefit from caching (like reading the same file multiple times)

## Configuration

The server can be configured through `config.json` or environment variables:

```json
{
  "maxEntries": 1000,        // Maximum number of items in cache
  "maxMemory": 104857600,    // Maximum memory usage in bytes (100MB)
  "defaultTTL": 3600,        // Default time-to-live in seconds (1 hour)
  "checkInterval": 60000,    // Cleanup interval in milliseconds (1 minute)
  "statsInterval": 30000     // Stats update interval in milliseconds (30 seconds)
}
```

### Configuration Settings Explained

1. **maxEntries** (default: 1000)
   - Maximum number of items that can be stored in cache
   - Prevents cache from growing indefinitely
   - When exceeded, oldest unused items are removed first

2. **maxMemory** (default: 100MB)
   - Maximum memory usage in bytes
   - Prevents excessive memory consumption
   - When exceeded, least recently used items are removed

3. **defaultTTL** (default: 1 hour)
   - How long items stay in cache by default
   - Items are automatically removed after this time
   - Prevents stale data from consuming memory

4. **checkInterval** (default: 1 minute)
   - How often the server checks for expired items
   - Lower values keep memory usage more accurate
   - Higher values reduce CPU usage

5. **statsInterval** (default: 30 seconds)
   - How often cache statistics are updated
   - Affects accuracy of hit/miss rates
   - Helps monitor cache effectiveness

## How It Reduces Token Consumption

The memory cache server reduces token consumption by automatically storing data that would otherwise need to be re-sent between you and the language model. You don't need to do anything special - the caching happens automatically when you interact with any language model through your MCP client.

Here are some examples of what gets cached:

### 1. File Content Caching
When reading a file multiple times:
- First time: Full file content is read and cached
- Subsequent times: Content is retrieved from cache instead of re-reading the file
- Result: Fewer tokens used for repeated file operations

### 2. Computation Results
When performing calculations or analysis:
- First time: Full computation is performed and results are cached
- Subsequent times: Results are retrieved from cache if the input is the same
- Result: Fewer tokens used for repeated computations

### 3. Frequently Accessed Data
When the same data is needed multiple times:
- First time: Data is processed and cached
- Subsequent times: Data is retrieved from cache until TTL expires
- Result: Fewer tokens used for accessing the same information

## Automatic Cache Management

The server automatically manages the caching process by:
- Storing data when first encountered
- Serving cached data when available
- Removing old/unused data based on settings
- Tracking effectiveness through statistics

## Optimization Tips

### 1. Set Appropriate TTLs
- Shorter for frequently changing data
- Longer for static content

### 2. Adjust Memory Limits
- Higher for more caching (more token savings)
- Lower if memory usage is a concern

### 3. Monitor Cache Stats
- High hit rate = good token savings
- Low hit rate = adjust TTL or limits

## Environment Variable Configuration

You can override config.json settings using environment variables in your MCP settings:

```json
{
  "mcpServers": {
    "memory-cache": {
      "command": "node",
      "args": ["/path/to/build/index.js"],
      "env": {
        "MAX_ENTRIES": "5000",
        "MAX_MEMORY": "209715200",  // 200MB
        "DEFAULT_TTL": "7200",      // 2 hours
        "CHECK_INTERVAL": "120000",  // 2 minutes
        "STATS_INTERVAL": "60000"    // 1 minute
      }
    }
  }
}
```

You can also specify a custom config file location:
```json
{
  "env": {
    "CONFIG_PATH": "/path/to/your/config.json"
  }
}
```

The server will:
1. Look for config.json in its directory
2. Apply any environment variable overrides
3. Use default values if neither is specified

## Testing the Cache in Practice

To see the cache in action, try these scenarios:

1. **File Reading Test**
   - Read and analyze a large file
   - Ask the same question about the file again
   - The second response should be faster as the file content is cached

2. **Data Analysis Test**
   - Perform analysis on some data
   - Request the same analysis again
   - The second analysis should use cached results

3. **Project Navigation Test**
   - Explore a project's structure
   - Query the same files/directories again
   - Directory listings and file contents will be served from cache

The cache is working when you notice:
- Faster responses for repeated operations
- Consistent answers about unchanged content
- No need to re-read files that haven't changed
