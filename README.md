# Infra Sage

A Model Context Protocol (MCP) server for Terraform infrastructure-as-code management. Generate modules, validate configurations, plan changes, and detect drift.

## Overview

Infra Sage brings AI assistance to your Terraform workflows. Instead of manually writing boilerplate or running CLI commands, you can use natural language to:

- Generate complete Terraform module scaffolding
- Validate your configurations before applying
- Preview infrastructure changes
- List and search resources in state
- Detect configuration drift

### Why Use Infra Sage?

**Traditional workflow:**
```bash
mkdir -p modules/vpc
touch modules/vpc/{main,variables,outputs}.tf
# Manually write provider blocks, variable definitions, etc.
terraform validate
terraform plan
```

**With Infra Sage:**
```
"Generate an AWS VPC module with variables for region and cidr_block"
"Validate my Terraform configuration"
"Show me what changes will be applied"
```

## Features

- **Module Generation** - Create complete Terraform modules with best practices baked in
- **Configuration Validation** - Syntax and semantic validation without leaving your editor
- **Change Planning** - Preview infrastructure changes before applying
- **Resource Listing** - Query resources in your Terraform state
- **Drift Detection** - Compare actual infrastructure against your configuration

## Installation

```bash
# Clone the repository
git clone https://github.com/consigcody94/infra-sage.git
cd infra-sage

# Install dependencies
npm install

# Build the project
npm run build
```

### Prerequisites

For full functionality, you'll need Terraform installed:

```bash
# macOS
brew install terraform

# Ubuntu/Debian
sudo apt-get install terraform

# Or download from https://www.terraform.io/downloads
```

## Configuration

### Claude Desktop Integration

Add to your `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "infra-sage": {
      "command": "node",
      "args": ["/absolute/path/to/infra-sage/build/index.js"]
    }
  }
}
```

**Config file locations:**
- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%APPDATA%\Claude\claude_desktop_config.json`
- Linux: `~/.config/Claude/claude_desktop_config.json`

### Working Directory

By default, Infra Sage operates in the current working directory. Most tools accept a `directory` parameter to specify a different location.

## Tools Reference

### generate_module

Generate a complete Terraform module with best-practice structure.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `moduleName` | string | Yes | Name of the module (e.g., `vpc`, `eks-cluster`) |
| `provider` | string | Yes | Cloud provider: `aws`, `azure`, `gcp`, or `kubernetes` |
| `variables` | array | No | Input variables for the module |
| `outputs` | array | No | Output values from the module |

**Variable object structure:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Variable name |
| `type` | string | Yes | Terraform type (`string`, `number`, `bool`, `list(string)`, etc.) |
| `description` | string | No | Variable description |
| `default` | string | No | Default value |

**Output object structure:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Output name |
| `value` | string | Yes | Output value expression |
| `description` | string | No | Output description |

**Example:**

```json
{
  "moduleName": "vpc",
  "provider": "aws",
  "variables": [
    {
      "name": "vpc_cidr",
      "type": "string",
      "description": "CIDR block for the VPC",
      "default": "10.0.0.0/16"
    },
    {
      "name": "environment",
      "type": "string",
      "description": "Environment name"
    },
    {
      "name": "availability_zones",
      "type": "list(string)",
      "description": "List of AZs to use"
    }
  ],
  "outputs": [
    {
      "name": "vpc_id",
      "value": "aws_vpc.main.id",
      "description": "ID of the created VPC"
    },
    {
      "name": "subnet_ids",
      "value": "aws_subnet.private[*].id",
      "description": "List of private subnet IDs"
    }
  ]
}
```

**Generated files:**
- `main.tf` - Main resource definitions with provider block
- `variables.tf` - Input variable declarations
- `outputs.tf` - Output value definitions
- `README.md` - Module documentation

### validate_config

Validate Terraform configuration files for syntax and semantic errors.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `directory` | string | No | Path to Terraform files (default: current directory) |

**Example:**

```json
{
  "directory": "./infrastructure/production"
}
```

**Response includes:**
- Validation status (success/failure)
- List of errors with file locations
- Warnings and recommendations

### plan_changes

Run `terraform plan` to preview infrastructure changes.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `directory` | string | No | Path to Terraform files |
| `varFile` | string | No | Path to `.tfvars` file |
| `target` | string | No | Target specific resource (e.g., `aws_instance.web`) |

**Example:**

```json
{
  "directory": "./infrastructure/staging",
  "varFile": "./environments/staging.tfvars",
  "target": "module.database"
}
```

**Response includes:**
- Resources to be added
- Resources to be changed
- Resources to be destroyed
- Full plan output

### list_resources

List all resources in Terraform state.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `directory` | string | No | Path to Terraform files |
| `filter` | string | No | Filter by resource type or name pattern |

**Example - List all resources:**

```json
{
  "directory": "./infrastructure"
}
```

**Example - Filter by type:**

```json
{
  "directory": "./infrastructure",
  "filter": "aws_instance"
}
```

**Example - Filter by name pattern:**

```json
{
  "filter": "database"
}
```

### check_drift

Detect configuration drift between state and actual infrastructure.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `directory` | string | No | Path to Terraform files |
| `detailed` | boolean | No | Show detailed drift information (default: false) |

**Example:**

```json
{
  "directory": "./infrastructure/production",
  "detailed": true
}
```

**Response includes:**
- Resources with drift detected
- Specific attribute changes
- Recommendations for remediation

## Workflow Examples

### Creating a New Module

1. **Generate the module:**
   ```
   generate_module with moduleName: "api-gateway", provider: "aws", variables: [...]
   ```

2. **Review generated files** in `./modules/api-gateway/`

3. **Customize the main.tf** with your specific resource configurations

4. **Validate the configuration:**
   ```
   validate_config with directory: "./modules/api-gateway"
   ```

### Planning Infrastructure Changes

1. **Check current state:**
   ```
   list_resources with directory: "./infrastructure"
   ```

2. **Preview changes:**
   ```
   plan_changes with directory: "./infrastructure", varFile: "./prod.tfvars"
   ```

3. **Check for drift before applying:**
   ```
   check_drift with directory: "./infrastructure", detailed: true
   ```

### Investigating State

1. **List all resources:**
   ```
   list_resources
   ```

2. **Find specific resource types:**
   ```
   list_resources with filter: "aws_lambda"
   ```

3. **Search by name:**
   ```
   list_resources with filter: "production"
   ```

## Provider-Specific Templates

### AWS Module Template

Generated AWS modules include:
- AWS provider configuration
- Default tags support
- Region variable
- Common AWS resource patterns

### Azure Module Template

Generated Azure modules include:
- AzureRM provider configuration
- Resource group integration
- Location variable
- Naming conventions

### GCP Module Template

Generated GCP modules include:
- Google provider configuration
- Project ID integration
- Region/zone variables
- Labels support

### Kubernetes Module Template

Generated Kubernetes modules include:
- Kubernetes provider configuration
- Namespace support
- Labels and annotations
- Resource quotas

## Security Notes

- **Read-only operations** - Never commits or modifies infrastructure automatically
- **No auto-apply** - All destructive operations require manual `terraform apply`
- **Workspace isolation** - Supports multiple workspaces
- **No credential storage** - Uses your existing Terraform/cloud provider credentials

## Requirements

- Node.js 18 or higher
- Terraform CLI (for plan, validate, and state operations)
- Initialized Terraform workspace (run `terraform init` first)

## Troubleshooting

### "terraform not found"

Ensure Terraform is installed and in your PATH:
```bash
which terraform
terraform version
```

### Validation/Plan fails

1. Run `terraform init` in the target directory first
2. Ensure all required providers are configured
3. Check for missing variable values

### Module generation path issues

The module is created relative to the server's working directory. Use absolute paths or ensure you're running from the expected location.

### State commands fail

1. Ensure you have a valid `terraform.tfstate` or remote backend configured
2. Run `terraform init` to initialize the backend
3. Check credentials for remote backends (S3, GCS, Azure Blob)

## Best Practices

1. **Always validate before planning** - Catch syntax errors early
2. **Use variable files** - Keep environment configs separate
3. **Check for drift regularly** - Especially before major changes
4. **Use modules** - Generated modules follow Terraform best practices
5. **Review plans carefully** - AI assistance doesn't replace human review

## License

MIT License - see [LICENSE](LICENSE) file for details.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Author

consigcody94
