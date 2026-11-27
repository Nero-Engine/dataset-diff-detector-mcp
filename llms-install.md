# Installing Dataset Diff & Change Detector (remote MCP server, no install needed)

Dataset Diff & Change Detector is a hosted remote MCP server. There is nothing to download, build or run, and no API key.

Add this to the MCP settings file (in Cline: MCP Servers, Configure, then cline_mcp_settings.json):

```json
{
  "mcpServers": {
    "dataset-diff-detector": {
      "type": "streamableHttp",
      "url": "https://dataset-diff-detector.nerolabs.workers.dev/mcp",
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

Save the file. The server's tools appear straight away. To check it works, ask the assistant to list the server's tools.

Docs: https://dataset-diff-detector.nerolabs.workers.dev/docs
