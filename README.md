# infra-sage

A production-ready Model Context Protocol (MCP) server for Terraform Infrastructure as Code assistance.

## Features

- **5 powerful tools** for Terraform infrastructure management
- Full TypeScript strict mode implementation
- Execute Terraform CLI commands safely
- Module generation with best practices
- Drift detection and state management
- Production-ready error handling

## Tools

### generate_module
Generate complete Terraform module templates with provider config, variables, outputs, and documentation.

### validate_config
Validate Terraform configuration files using `terraform validate` with JSON output.

### plan_changes
Preview infrastructure changes with `terraform plan`, supporting var-files and targeted resources.

### list_resources
List all resources in Terraform state with optional filtering by type or name.

### check_drift
Detect configuration drift between Terraform state and actual infrastructure using refresh-only plans.

## Installation

```bash
npm install
npm run build
```

## Configuration

Optionally set your Terraform working directory:

```bash
export TERRAFORM_DIR="/path/to/terraform/project"
```

If not set, the current working directory will be used.

## Usage with Claude Desktop

Add to your `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "infra-sage": {
      "command": "node",
      "args": ["/path/to/infra-sage/build/index.js"],
      "env": {
        "TERRAFORM_DIR": "/path/to/terraform/project"
      }
    }
  }
}
```

## Example Usage

```typescript
// Generate a module
{
  "tool": "generate_module",
  "arguments": {
    "moduleName": "vpc",
    "provider": "aws",
    "variables": [
      {
        "name": "cidr_block",
        "type": "string",
        "description": "VPC CIDR block",
        "default": "\"10.0.0.0/16\""
      },
      {
        "name": "enable_dns",
        "type": "bool",
        "description": "Enable DNS support",
        "default": "true"
      }
    ],
    "outputs": [
      {
        "name": "vpc_id",
        "description": "VPC identifier",
        "value": "aws_vpc.main.id"
      }
    ]
  }
}

// Validate configuration
{
  "tool": "validate_config",
  "arguments": {
    "directory": "./terraform"
  }
}

// Plan changes
{
  "tool": "plan_changes",
  "arguments": {
    "directory": "./terraform",
    "varFile": "prod.tfvars"
  }
}

// List resources
{
  "tool": "list_resources",
  "arguments": {
    "filter": "aws_instance"
  }
}

// Check drift
{
  "tool": "check_drift",
  "arguments": {
    "detailed": true
  }
}
```

## Features

- **Multi-Provider Support**: AWS, Azure, GCP, Kubernetes
- **Module Generation**: Complete module scaffolding with documentation
- **Validation**: JSON-formatted validation output
- **Plan Analysis**: Extract and summarize planned changes
- **State Management**: List and filter resources in state
- **Drift Detection**: Identify infrastructure drift with detailed reporting

## Requirements

- Node.js 18+
- Terraform CLI installed and in PATH
- Initialized Terraform workspace (for state operations)

## Security Notes

- Never commits or modifies infrastructure automatically
- Read-only operations except for module generation
- All destructive operations require manual confirmation via Terraform
- Supports workspace isolation

## License

MIT
