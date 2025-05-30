# Auth0 Integration Guide

This guide provides detailed information about integrating Auth0 with ETDI for secure tool authentication and authorization.

## Prerequisites

- An Auth0 account
- ETDI Python SDK installed
- Basic understanding of OAuth 2.0

## Auth0 Setup Details

### Application Configuration

In your Auth0 dashboard:

1. **Application Settings**
   - Application Type: Machine to Machine
   - Token Endpoint Authentication Method: `client_secret_post`
   - Grant Types: Client Credentials
   - Allow Non OIDC Compliant Conformance: Yes

2. **API Configuration**
   ```json
   {
     "identifier": "https://api.etdi-tools.demo.com",
     "signing_alg": "RS256",
     "token_lifetime": 86400,
     "allow_offline_access": true,
     "skip_consent_for_verifiable_first_party_clients": true
   }
   ```

3. **Required Scopes**
   ```json
   [
     {
       "value": "create:resource-servers",
       "description": "Create new tool registrations"
     },
     {
       "value": "read:resource-servers",
       "description": "Read tool information"
     },
     {
       "value": "update:resource-servers",
       "description": "Update existing tools"
     },
     {
       "value": "delete:resource-servers",
       "description": "Remove tool registrations"
     }
   ]
   ```

## Configuring Auth0 for Demo Scripts

### Demo Script Configuration

The ETDI package includes two main demo scripts that showcase different aspects of the security features:
- `run_e2e_demo.py`: Demonstrates end-to-end ETDI features
- `run_real_server_demo.py`: Shows tool poisoning prevention

To get both demos working, you'll need to configure Auth0 correctly:

### 1. Auth0 Application Setup

Create two applications in your Auth0 dashboard:

1. **ETDI Tool Registry Application**
   ```json
   {
     "name": "ETDI Tool Registry",
     "application_type": "machine_to_machine",
     "token_endpoint_auth_method": "client_secret_post",
     "grant_types": ["client_credentials"],
     "allowed_origins": [],
     "client_id": "your_client_id",  // You'll get this after creation
     "client_secret": "your_client_secret"  // You'll get this after creation
   }
   ```

2. **ETDI Demo Server Application**
   ```json
   {
     "name": "ETDI Demo Server",
     "application_type": "machine_to_machine",
     "token_endpoint_auth_method": "client_secret_post",
     "grant_types": ["client_credentials"],
     "allowed_origins": [],
   }
   ```

### 2. Auth0 API Configuration

Create an API with these settings:

```json
{
  "name": "ETDI Tool Registry API",
  "identifier": "https://api.etdi-tools.demo.com",
  "signing_alg": "RS256",
  "token_lifetime": 86400,
  "scopes": {
    "create:resource-servers": "Create new tool registrations",
    "read:resource-servers": "Read tool information",
    "update:resource-servers": "Update existing tools",
    "delete:resource-servers": "Remove tool registrations",
    "execute:tools": "Execute registered tools"
  }
}
```

### 3. Environment Variables

Set these environment variables for both demo scripts:

```bash
# For run_e2e_demo.py
export AUTH0_DOMAIN="your-tenant.auth0.com"
export AUTH0_CLIENT_ID="your-tool-registry-client-id"
export AUTH0_CLIENT_SECRET="your-tool-registry-client-secret"
export AUTH0_AUDIENCE="https://api.etdi-tools.demo.com"

# For run_real_server_demo.py (additional vars)
export ETDI_AUTH0_DOMAIN="your-tenant.auth0.com"
export ETDI_DEMO_CLIENT_ID="your-demo-server-client-id"
export ETDI_DEMO_CLIENT_SECRET="your-demo-server-client-secret"
```

### 4. Code Changes

#### For run_e2e_demo.py

Update the OAuth configuration in the demo_tool_provider_sdk() function:

```python
oauth_config = OAuthConfig(
    provider="auth0",
    client_id=os.getenv("AUTH0_CLIENT_ID"),
    client_secret=os.getenv("AUTH0_CLIENT_SECRET"),
    domain=os.getenv("AUTH0_DOMAIN"),
    audience=os.getenv("AUTH0_AUDIENCE"),
    scopes=["create:resource-servers", "read:resource-servers"]  # Updated scopes
)
```

#### For run_real_server_demo.py

In the legitimate_etdi_server.py:

```python
oauth_config = OAuthConfig(
    provider="auth0",
    client_id=os.getenv("ETDI_DEMO_CLIENT_ID"),
    client_secret=os.getenv("ETDI_DEMO_CLIENT_SECRET"),
    domain=os.getenv("ETDI_AUTH0_DOMAIN"),
    audience=os.getenv("AUTH0_AUDIENCE"),
    scopes=["execute:tools", "read:resource-servers"]
)
```

### 5. Permissions Setup

1. Go to your Auth0 dashboard > APIs > ETDI Tool Registry API
2. Click on "Permissions" tab
3. Add all required scopes:
   ```
   create:resource-servers
   read:resource-servers
   update:resource-servers
   delete:resource-servers
   execute:tools
   ```

4. Go to "Machine to Machine Applications"
5. Authorize both applications and grant appropriate scopes:
   - Tool Registry App: All scopes
   - Demo Server App: `execute:tools`, `read:resource-servers`

### 6. Testing the Configuration

1. Test the e2e demo:
```bash
# First test
python examples/etdi/run_e2e_demo.py

# If you see OAuth errors, verify your environment variables:
echo $AUTH0_DOMAIN
echo $AUTH0_CLIENT_ID
echo $AUTH0_CLIENT_SECRET
echo $AUTH0_AUDIENCE
```

2. Test the real server demo:
```bash
# First test
python examples/etdi/tool_poisoning_demo/run_real_server_demo.py

# If you see OAuth errors, verify your environment variables:
echo $ETDI_AUTH0_DOMAIN
echo $ETDI_DEMO_CLIENT_ID
echo $ETDI_DEMO_CLIENT_SECRET
```

### Common Issues

1. **401 Unauthorized in run_e2e_demo.py**
   - Verify AUTH0_CLIENT_SECRET is correct
   - Check if scopes are properly granted in Auth0 dashboard
   - Ensure audience matches API identifier exactly

2. **403 Forbidden in run_real_server_demo.py**
   - Check if ETDI_DEMO_CLIENT_ID has proper scopes
   - Verify ETDI_AUTH0_DOMAIN is correct
   - Ensure API permissions are properly set up

3. **Token Validation Errors**
   - Verify signing algorithm is RS256
   - Check if audience matches in both config and Auth0
   - Ensure domain is correct and includes ".auth0.com"

## Implementation Examples

### Basic Auth0 Provider

```python
from mcp.etdi.oauth import OAuthConfig, Auth0Provider

# Create OAuth configuration
oauth_config = OAuthConfig(
    provider="auth0",
    client_id=os.getenv("AUTH0_CLIENT_ID"),
    client_secret=os.getenv("AUTH0_CLIENT_SECRET"),
    domain=os.getenv("AUTH0_DOMAIN"),
    audience=os.getenv("AUTH0_AUDIENCE"),
    scopes=["read:resource-servers"]
)

# Initialize provider
auth0_provider = Auth0Provider(oauth_config)
await auth0_provider.initialize()
```

### Secure Server with Auth0

```python
from mcp.etdi import SecureServer
from mcp.etdi.types import SecurityLevel

# Create server with Auth0
server = SecureServer(
    name="auth0-secure-server",
    security_level=SecurityLevel.HIGH,
    oauth_provider=auth0_provider,
    enable_request_signing=True
)

# Register protected tool
@server.tool(
    "protected_tool",
    required_scopes=["read:resource-servers"],
    require_auth=True
)
async def protected_tool(data: str) -> str:
    return f"Protected data: {data}"
```

### Client with Auth0 Authentication

```python
from mcp.etdi.client import ETDIClient
from mcp.etdi.types import ETDIClientConfig

# Create client config
client_config = ETDIClientConfig(
    security_level=SecurityLevel.HIGH,
    oauth_config=oauth_config,
    verification_cache_ttl=300
)

# Initialize client
client = ETDIClient(client_config)
await client.initialize()

# Call protected tool
result = await client.call_tool(
    "protected_tool",
    {"data": "test"},
    required_scopes=["read:resource-servers"]
)
```

## Security Best Practices

1. **Token Management**
   - Store tokens securely
   - Implement token refresh logic
   - Handle token revocation

2. **Scope Management**
   - Use principle of least privilege
   - Regularly audit granted scopes
   - Implement scope validation

3. **Error Handling**
   ```python
   try:
       token = await auth0_provider.get_token(
           tool_id="secure-tool",
           permissions=["read:resource-servers"]
       )
   except Auth0Error as e:
       if e.error == "invalid_client":
           # Handle invalid credentials
           logger.error("Invalid Auth0 credentials")
       elif e.error == "insufficient_scope":
           # Handle missing permissions
           logger.error("Insufficient permissions")
   ```

## Monitoring and Debugging

### Enable Debug Logging

```python
import logging
logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger("mcp.etdi.oauth.auth0")
```

### Monitor Auth0 Events

```python
from mcp.etdi.events import EventType, get_event_emitter

emitter = get_event_emitter()
emitter.on(EventType.AUTH_SUCCESS, lambda e: print(f"Auth success: {e}"))
emitter.on(EventType.AUTH_FAILURE, lambda e: print(f"Auth failure: {e}"))
```

## Common Issues and Solutions

1. **Token Acquisition Failures**
   - Verify client credentials
   - Check API audience
   - Confirm scope configuration

2. **Invalid Scope Errors**
   - Review API permissions
   - Check scope formatting
   - Verify scope assignments

3. **Rate Limiting**
   - Implement token caching
   - Use exponential backoff
   - Monitor rate limits

## Testing Auth0 Integration

```python
import pytest
from mcp.etdi.oauth import Auth0Provider

@pytest.mark.asyncio
async def test_auth0_integration():
    # Initialize provider
    provider = Auth0Provider(oauth_config)
    await provider.initialize()
    
    # Test token acquisition
    token = await provider.get_token(
        tool_id="test-tool",
        permissions=["read:resource-servers"]
    )
    assert token.access_token
    assert "read:resource-servers" in token.scopes
    
    # Test token validation
    is_valid = await provider.validate_token(token.access_token)
    assert is_valid
    
    await provider.cleanup()
```

## Additional Resources

- [Auth0 Documentation](https://auth0.com/docs)
- [OAuth 2.0 Specification](https://oauth.net/2/)
- [ETDI Security Guide](../../security-features.md)
- [Example Implementation](../../../examples/etdi/oauth_providers/auth0_example.py) 