# OAuth Provider Integration Examples

ETDI supports various OAuth providers for secure authentication and authorization. This section contains detailed guides and examples for integrating different OAuth providers with your ETDI implementation.

## Available Providers

### [Auth0](auth0.md)
Complete guide for integrating Auth0 with ETDI, including:
- Application setup and configuration
- API and scope management
- Demo script configuration
- Troubleshooting and best practices

## Common OAuth Concepts

All OAuth provider integrations in ETDI share some common concepts:

1. **Provider Configuration**
   ```python
   from mcp.etdi.oauth import OAuthConfig
   
   oauth_config = OAuthConfig(
       provider="provider_name",
       client_id="your_client_id",
       client_secret="your_client_secret",
       domain="your_domain",
       audience="your_api_audience",
       scopes=["required", "scopes"]
   )
   ```

2. **Token Management**
   - Secure token storage
   - Token refresh handling
   - Token validation

3. **Scope Management**
   - Permission definition
   - Scope validation
   - Access control

4. **Error Handling**
   - Authentication failures
   - Authorization errors
   - Token validation issues

## Best Practices

1. **Security**
   - Use environment variables for sensitive data
   - Implement proper error handling
   - Follow OAuth 2.0 best practices
   - Regular security audits

2. **Configuration**
   - Validate all OAuth settings
   - Test with minimal scopes first
   - Monitor token usage and expiration
   - Implement proper logging

3. **Integration**
   - Start with basic authentication
   - Add authorization gradually
   - Test thoroughly in staging
   - Monitor in production

## Example Usage

Basic OAuth provider setup:

```python
from mcp.etdi.oauth import OAuthProvider
from mcp.etdi.types import OAuthConfig

# Create configuration
config = OAuthConfig(
    provider="your_provider",
    client_id=os.getenv("OAUTH_CLIENT_ID"),
    client_secret=os.getenv("OAUTH_CLIENT_SECRET"),
    domain=os.getenv("OAUTH_DOMAIN"),
    audience=os.getenv("OAUTH_AUDIENCE"),
    scopes=["read", "write"]
)

# Initialize provider
provider = OAuthProvider(config)
await provider.initialize()

# Use in secure server
server = SecureServer(
    name="secure-server",
    oauth_provider=provider,
    security_level=SecurityLevel.HIGH
)
```

## Adding New Providers

To add support for a new OAuth provider:

1. Create provider class:
   ```python
   from mcp.etdi.oauth import BaseOAuthProvider
   
   class CustomProvider(BaseOAuthProvider):
       async def get_token(self, *args, **kwargs):
           # Implement token acquisition
           pass
           
       async def validate_token(self, token: str) -> bool:
           # Implement token validation
           pass
   ```

2. Register provider:
   ```python
   from mcp.etdi.oauth import register_provider
   
   register_provider("custom", CustomProvider)
   ```

3. Create documentation:
   - Add provider guide (like auth0.md)
   - Update this index
   - Add examples and tests

## See Also

- [Security Features](../../../security-features.md)
- [Getting Started](../../../getting-started.md)
- [ETDI Concepts](../../../etdi-concepts.md) 