# AI Gateway Development Guide

## Architecture Overview

This repository implements an **AI Gateway pattern** using Azure API Management (APIM) as the central control plane for Azure OpenAI and other AI services. Each lab demonstrates specific capabilities like token limiting, load balancing, and semantic caching.

### Core Components
- **`/labs/`**: Experimental playgrounds, each with `main.bicep` + `policy.xml` + `.ipynb` notebook
- **`/modules/`**: Reusable Bicep modules for APIM (`/modules/apim/v1/`), cognitive services, monitoring
- **`/shared/`**: Common utilities (`utils.py`, `apimtools.py`) for deployment and APIM management
- **`/tools/`**: Testing utilities including mock server and tracing tools

## Lab Structure Pattern

Every lab follows this standardized structure:
```
labs/[lab-name]/
├── main.bicep              # Infrastructure definition with module imports
├── policy.xml              # APIM policy with {placeholders} for Bicep substitution
├── [lab-name].ipynb        # Step-by-step deployment and testing notebook
├── params.json             # Generated Bicep parameters (not in git)
├── clean-up-resources.ipynb # Resource cleanup
└── README.MD               # Lab-specific documentation
```

## Development Workflow

1. **Initialize notebook variables** with resource names, locations, and model configs
2. **Generate Bicep parameters** as JSON from notebook variables
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

# Verify Azure CLI auth
output = utils.run("az account show", "Retrieved az account", "Failed to get current az account")
```

### Environment Dependencies
- **Azure CLI**: Must be authenticated (`az login`)
- **Python 3.12+**: With packages from root `requirements.txt`
- **VS Code**: With Jupyter extension for notebook execution
- **Azure Permissions**: Contributor + RBAC Administrator (or Owner) on subscription

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