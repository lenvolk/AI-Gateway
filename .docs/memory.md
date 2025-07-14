# AI Gateway Project Analysis

## Architecture Overview
- **Azure API Management-centric**: All labs route through APIM as the gateway layer
- **Modular Bicep Infrastructure**: Reusable modules in `/modules/` for APIM, cognitive services, monitoring
- **Lab-based Experimentation**: Each lab (`/labs/`) demonstrates specific APIM+OpenAI patterns
- **Jupyter Notebook Workflow**: Each lab has a notebook with step-by-step deployment and testing

## Key Patterns Discovered
1. **Standard Lab Structure**: main.bicep + policy.xml + notebook.ipynb + README.MD + cleanup
2. **Shared Utilities**: `/shared/utils.py` and `/shared/apimtools.py` for common operations
3. **Policy Templates**: XML policies with placeholder replacement via Bicep
4. **Resource Naming**: Consistent pattern with unique suffixes based on subscription ID
5. **Multi-endpoint Load Balancing**: Backend pools for resilience and scale

## Development Workflow
1. Define parameters in notebook variables
2. Generate Bicep parameters JSON
3. Deploy infrastructure via `az deployment group create`
4. Test endpoints through APIM with subscription keys
5. Clean up resources via shared utilities

## Critical Dependencies
- Azure CLI authentication required
- Python 3.12+ with specific packages (see requirements.txt)
- VS Code with Jupyter extension
- Azure subscription with RBAC permissions
