# AI Gateway Development Guide

## Architecture Overview

This repository implements an **AI Gateway pattern** using Azure API Management (APIM) as the central control plane for Azure OpenAI and other AI services. Each lab demonstrates specific capabilities like token limiting, load balancing, semantic caching, and Model Context Protocol (MCP) integration.

### Core Components
- **`/labs/`**: Experimental playgrounds, each with `main.bicep` + `policy.xml` + `.ipynb` notebook
- **`/modules/`**: Reusable Bicep modules for APIM (`/modules/apim/v1/`), cognitive services, monitoring  
- **`/shared/`**: Common utilities (`utils.py`, `apimtools.py`) for deployment and APIM management
- **`/tools/`**: Testing utilities including mock server, tracing, and OpenAI client tools
- **`/workshop/`**: Portal-based workshop experience for trying AI Gateway features

## Lab Structure Pattern

Every lab follows this standardized structure:
```
labs/[lab-name]/
├── main.bicep              # Infrastructure definition with module imports
├── policy.xml              # APIM policy with {placeholders} for Bicep substitution
├── [lab-name].ipynb        # Step-by-step deployment and testing notebook
├── params.json             # Generated Bicep parameters (not in git)
├── clean-up-resources.ipynb # Resource cleanup
├── README.MD               # Lab-specific documentation
└── src/                    # Lab-specific source code (when needed)
```

## Development Workflow

1. **Initialize notebook variables**: Set `deployment_name`, `resource_group_name`, model configs
2. **Generate Bicep parameters** as JSON from notebook variables using Python
3. **Deploy via Azure CLI**: `az deployment group create --template-file main.bicep --parameters @params.json`
4. **Test through APIM** using subscription keys and gateway URLs from deployment outputs
5. **Clean up** using shared utilities: `utils.cleanup_resources(deployment_name, resource_group_name)`

## Key Patterns

### Bicep Module Usage
```bicep
// Standard pattern: import shared modules, apply policy transformations
module apimModule '../../modules/apim/v1/apim.bicep' = { ... }
module openAIModule '../../modules/cognitive-services/v1/openai.bicep' = { ... }

// Policy placeholder replacement
var updatedPolicyXml = replace(loadTextContent('policy.xml'), '{backend-id}', backendId)
```

### APIM Policy Structure
All policies follow this inbound→backend→outbound→on-error structure with managed identity auth:
```xml
<policies>
    <inbound>
        <base />
        <authentication-managed-identity resource="https://cognitiveservices.azure.com" />
        <set-header name="Authorization">...</set-header>
        <set-backend-service backend-id="{backend-id}" />
        <!-- Lab-specific policies here -->
    </inbound>
    <!-- ... -->
</policies>
```

### Resource Naming Convention
- Resource groups: `lab-{deployment-name}` where deployment-name = notebook directory name
- Unique suffixes: `uniqueString(subscription().id, resourceGroup().id)` 
- APIM instance: `apim-{resourceSuffix}`

## Critical Setup Requirements

### Prerequisites Check
```python
# Always start notebooks with this pattern:
import os, sys, json
sys.path.insert(1, '../../shared')  # Add shared utilities
import utils

# Set deployment name from notebook directory
deployment_name = os.path.basename(os.path.dirname(globals()['__vsc_ipynb_file__']))
resource_group_name = f"lab-{deployment_name}"

# Verify Azure CLI auth
output = utils.run("az account show", "Retrieved az account", "Failed to get current az account")
```

### Environment Dependencies
- **Azure CLI**: Must be authenticated (`az login`)
- **Python 3.12+**: With packages from root `requirements.txt`
- **VS Code**: With Jupyter extension for notebook execution
- **Azure Permissions**: Contributor + RBAC Administrator (or Owner) on subscription

### Variable Naming Pattern
All labs follow consistent variable naming:
```python
# Standard deployment variables
deployment_name = os.path.basename(os.path.dirname(globals()['__vsc_ipynb_file__']))
resource_group_name = f"lab-{deployment_name}"
resource_group_location = "uksouth"  # Adjust for your region

# OpenAI model configuration
openai_model_name = "gpt-4o-mini"
openai_model_version = "2024-07-18"
openai_api_version = "2024-10-21"
openai_deployment_name = "gpt-4o-mini"
```

## Testing and Debugging

### Using Shared Tools
```python
from shared.apimtools import APIMClientTool

# Discover and test APIM APIs
apim_client = APIMClientTool(resource_group_name)
apim_client.initialize()
api_path = apim_client.discover_api("/openai")
```

### Mock Server for Testing
Use `/tools/mock-server/` for offline development:
```python
# Customize app.py for specific response patterns
# Supports OpenAI API compatibility for development
```

## Common Gotchas

- **Policy placeholders**: Must match exactly between `policy.xml` and Bicep replacement logic
- **Module paths**: Always use relative paths `../../modules/` from lab directories
- **Resource cleanup**: Use `utils.cleanup_resources()` to avoid orphaned Cognitive Services accounts
- **API versions**: OpenAI API versions change frequently; check lab parameters for current versions
- **Regional availability**: Not all OpenAI models available in all regions; check Azure docs

## Model Context Protocol (MCP) Labs

Recent labs explore MCP integration:
- **Client authorization flow**: OAuth2 with APIM as both client and server
- **Tool integration**: Connecting LLMs to external APIs through MCP protocol
- **Agent workflows**: OpenAI Agents SDK with APIM-controlled backend services

## Python Dependencies

All labs share a common `requirements.txt` at the root with essential packages:
- `azure-mgmt-apimanagement`: For APIM management operations  
- `openai`, `azure-ai-projects`, `azure-ai-inference`: AI model interactions
- `mcp`, `autogen-core`, `semantic-kernel[mcp]`: Model Context Protocol support
- `requests`, `azure-identity`, `azure-cli`: Core Azure operations

## Workshop Experience

The `/workshop/` directory provides a portal-based experience for trying AI Gateway features without deep technical setup. Use this for demos or quick feature exploration before diving into lab implementations.

## Agent-Specific Labs

Several labs focus on agentic AI patterns:
- `openai-agents/`: OpenAI Agents SDK with function calling
- `ai-agent-service/`: Azure AI Agent Service integration  
- `model-context-protocol/`, `mcp-client-authorization/`: MCP tool integration
- `realtime-mcp-agents/`: Real-time audio with MCP tools
- `mcp-a2a-agents/`: Agent-to-agent communication patterns