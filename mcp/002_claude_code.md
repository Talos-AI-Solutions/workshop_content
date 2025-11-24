claude add --transport [stdio, sse, http] <name> --scope [local, project, user] <source>

claude mcp add -t stdio sequentialthinking -s user -- cmd /c "npx -y @modelcontextprotocol/server-sequential-thinking"
claude mcp add -t stdio context7 -s user -- cmd /c "npx -y @upstash/context7-mcp@latest"
claude mcp add -t stdio chadcn-ui-mcp -s user -- cmd /c "npx -y @jpisnice/shadcn-ui-mcp-server"
claude mcp add -t http figma https://mcp.figma.com/mcp