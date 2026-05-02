Connect to the project's SQL Server database. Follow these steps exactly:

## Step 1 — Find the connection string

Read `appsettings.json` from the current project's API folder (look for a file matching `src/*/appsettings.json` or `appsettings.json` in the working directory tree). Extract:
- `ConnectionStrings:DefaultConnection`

Parse out: Server, Database, whether it uses Integrated Security or SQL auth (username/password).

## Step 2 — Check if mssql-mcp is installed

Run:
```
node -e "require('C:/Users/Ednmh/AppData/Roaming/npm/node_modules/mssql-mcp/package.json')" 2>&1
```

If that fails (module not found), install it:
```
npm install -g mssql-mcp
```

Wait for it to complete before continuing.

## Step 3 — Check if MCP server is configured in settings

Read `C:/Users/Ednmh/.claude/settings.json`.

If an `mcpServers.mssql` entry does not exist, add it using the connection details from Step 1:
- If Integrated Security: set `MSSQL_TRUSTED_CONNECTION=true`, `MSSQL_TRUST_SERVER_CERTIFICATE=true`
- If SQL auth: set `MSSQL_USER` and `MSSQL_PASSWORD` from the connection string

The entry should look like:
```json
"mcpServers": {
  "mssql": {
    "command": "node",
    "args": ["C:/Users/Ednmh/AppData/Roaming/npm/node_modules/mssql-mcp/dist/src/index.js"],
    "env": {
      "MSSQL_SERVER": "<server>",
      "MSSQL_DATABASE": "<database>",
      "MSSQL_TRUSTED_CONNECTION": "true",
      "MSSQL_TRUST_SERVER_CERTIFICATE": "true"
    }
  }
}
```

Write the updated settings back if any change was made.

## Step 4 — Verify connectivity

Run a quick smoke test using sqlcmd:
```
sqlcmd -S <server> -d <database> -E -Q "SELECT TOP 1 name FROM sys.tables ORDER BY name" -W
```

If sqlcmd is not available, try:
```
node -e "
const sql = require('C:/Users/Ednmh/AppData/Roaming/npm/node_modules/mssql-mcp/node_modules/mssql');
sql.connect({ server: '<server>', database: '<database>', options: { trustedConnection: true, trustServerCertificate: true } })
  .then(() => { console.log('connected'); sql.close(); })
  .catch(e => console.error(e.message));
"
```

## Step 5 — Report to the user

Tell the user:
- Which server and database you connected to
- Whether mssql-mcp was freshly installed or already present
- Whether the MCP config was added or already existed
- **Important note:** MCP tools (direct query access) only activate after a Claude Code restart. Until then, all queries will run via `sqlcmd` bash calls — which works fine for everything.
- List the first 5 table names found in the database so the user can see it's working.
