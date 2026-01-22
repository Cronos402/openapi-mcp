![Cronos402 Logo](https://raw.githubusercontent.com/Cronos402/assets/main/Cronos402-logo-light.svg)

# Cronos402 OpenAPI-MCP Converter

Automatically convert OpenAPI specifications to Model Context Protocol (MCP) tools.

Production URL: https://openapi.cronos402.dev

## Overview

The OpenAPI-MCP service bridges the gap between traditional REST APIs and the Model Context Protocol. It reads OpenAPI/Swagger specifications and dynamically generates MCP tools, allowing AI agents to interact with any REST API through the standardized MCP interface. This enables instant AI agent integration with existing APIs without manual wrapper development.

## Architecture

- **Framework**: Express.js with TypeScript
- **Parser**: OpenAPI 3.x specification parser
- **Generator**: Dynamic MCP tool generation
- **Validation**: JSON Schema validation for requests/responses
- **Caching**: Specification caching for performance
- **Protocol**: Model Context Protocol (MCP) HTTP transport

## Features

- Automatic OpenAPI to MCP tool conversion
- Support for OpenAPI 3.x specifications
- Dynamic tool generation from API endpoints
- Request/response validation
- Authentication header forwarding
- Parameter type mapping
- Error handling and transformation
- Specification caching
- Compatible with all MCP clients

## Quick Start

### Development

```bash
pnpm install
pnpm dev
```

Server runs on `http://localhost:3001`

### Build

```bash
pnpm build
pnpm start
```

### Environment Variables

```env
PORT=3001
CACHE_ENABLED=true
CACHE_TTL=3600
```

## Usage Examples

### Convert OpenAPI Spec to MCP

Provide an OpenAPI specification URL:

```bash
curl -X POST https://openapi.cronos402.dev/convert \
  -H "Content-Type: application/json" \
  -d '{
    "openApiUrl": "https://api.example.com/openapi.json"
  }'
```

The service generates an MCP endpoint at:
```
https://openapi.cronos402.dev/mcp/{spec-id}
```

### Connect with CLI

```bash
npx cronos402 connect --urls https://openapi.cronos402.dev/mcp/your-spec-id
```

### Claude Desktop Configuration

```json
{
  "mcpServers": {
    "My REST API": {
      "command": "npx",
      "args": [
        "cronos402",
        "connect",
        "--urls",
        "https://openapi.cronos402.dev/mcp/spec-id",
        "--api-key",
        "your_api_key"
      ]
    }
  }
}
```

### Programmatic Access

```typescript
import { Client } from '@modelcontextprotocol/sdk/client/index.js';
import { StreamableHTTPClientTransport } from '@modelcontextprotocol/sdk/client/streamableHttp.js';

const transport = new StreamableHTTPClientTransport(
  new URL('https://openapi.cronos402.dev/mcp/spec-id')
);

const client = new Client(
  { name: 'my-app', version: '1.0.0' },
  { capabilities: {} }
);

await client.connect(transport);

// List all tools generated from OpenAPI spec
const tools = await client.listTools();

// Call REST API endpoint as MCP tool
const result = await client.callTool({
  name: 'get_users',
  arguments: { page: 1, limit: 10 }
});
```

## API Endpoints

### POST /convert
Convert an OpenAPI specification to MCP.

Request:
```json
{
  "openApiUrl": "https://api.example.com/openapi.json",
  "name": "My API",
  "authentication": {
    "type": "bearer",
    "token": "optional_token"
  }
}
```

Response:
```json
{
  "specId": "abc123",
  "mcpUrl": "https://openapi.cronos402.dev/mcp/abc123",
  "toolCount": 15
}
```

### GET /mcp/:specId
MCP endpoint for converted specification.

### GET /specs/:specId
Retrieve specification metadata.

### DELETE /specs/:specId
Remove cached specification.

## How It Works

### Conversion Process

1. **Parse**: Read OpenAPI specification
2. **Validate**: Ensure spec is valid OpenAPI 3.x
3. **Generate**: Create MCP tools for each endpoint
   - Tool name from `operationId`
   - Parameters from path/query/body params
   - Schema from request/response definitions
4. **Cache**: Store specification and tools
5. **Serve**: Provide MCP endpoint for tool calls

### Tool Mapping

OpenAPI endpoints map to MCP tools:

```yaml
# OpenAPI
paths:
  /users/{id}:
    get:
      operationId: getUser
      parameters:
        - name: id
          in: path
          schema:
            type: string
```

Becomes MCP tool:
```json
{
  "name": "getUser",
  "description": "Get user by ID",
  "inputSchema": {
    "type": "object",
    "properties": {
      "id": { "type": "string" }
    },
    "required": ["id"]
  }
}
```

### Request Flow

```
MCP Client → OpenAPI-MCP → REST API
              ↓
         Convert tool call to HTTP request
              ↓
         Execute REST call
              ↓
         Convert response to MCP format
```

## Authentication

The converter supports authentication forwarding:

```json
{
  "openApiUrl": "https://api.example.com/openapi.json",
  "authentication": {
    "type": "bearer",
    "token": "your_token"
  }
}
```

Token is forwarded in `Authorization` header to the API.

## Testing

```bash
# Run all tests
pnpm test

# Watch mode
pnpm test:watch

# Coverage
pnpm test:coverage
```

## Deployment

### Production Build

```bash
pnpm build
NODE_ENV=production pnpm start
```

### Docker

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package.json pnpm-lock.yaml ./
RUN npm install -g pnpm && pnpm install --frozen-lockfile
COPY . .
RUN pnpm build
EXPOSE 3001
CMD ["pnpm", "start"]
```

### Environment Requirements

- Node.js 20+
- Redis for caching (optional)
- HTTPS endpoint in production

## Monitoring

- `GET /health` - Service health check
- `GET /metrics` - Prometheus metrics
- `GET /specs` - List all cached specifications

## Limitations

- OpenAPI 3.x only (2.0 not supported)
- Large specifications may have performance impact
- Authentication types limited to bearer/API key
- File upload endpoints not fully supported

## Resources

- **Documentation**: [docs.cronos402.dev/openapi-mcp](https://docs.cronos402.dev)
- **SDK**: [npmjs.com/package/cronos402](https://www.npmjs.com/package/cronos402)
- **GitHub**: [github.com/Cronos402/openapi-mcp](https://github.com/Cronos402/openapi-mcp)
- [OpenAPI Specification](https://swagger.io/specification/)
- [Model Context Protocol](https://modelcontextprotocol.io)

## License

MIT
