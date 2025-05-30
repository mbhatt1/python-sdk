# Getting Started with ETDI

For a conceptual overview of ETDI and its security model, see [ETDI Concepts](etdi-concepts.md).

This guide will help you set up the Enhanced Tool Definition Interface (ETDI) security framework and create your first secure AI tool server.

## Prerequisites

- Python 3.11 or higher
- Git
- A text editor or IDE

## Installation

### 1. Clone the Repository

```bash
git clone https://github.com/python-sdk-etdi/python-sdk-etdi.git
cd python-sdk-etdi
```

### 2. Set Up Virtual Environment

```bash
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
```

### 3. Install Dependencies

```bash
pip install -e .
```

## Quick Start Example

Create your first secure server:

```python
# secure_server_example.py
import asyncio
from mcp.etdi import SecureServer, ToolProvider
from mcp.etdi.types import SecurityLevel

async def main():
    # Create secure server with high security
    server = SecureServer(
        name="my-secure-server",
        security_level=SecurityLevel.HIGH,
        enable_tool_verification=True
    )
    
    # Register a secure tool
    @server.tool("get_weather")
    async def get_weather(location: str) -> dict:
        """Get weather for a location with security verification."""
        # Tool implementation here
        return {"location": location, "temperature": "72°F"}
    
    # Start the server
    await server.start()

if __name__ == "__main__":
    asyncio.run(main())
```

## Setting Up Auth0 Authentication

ETDI supports OAuth2 authentication through Auth0. Follow these steps to set up Auth0:

### 1. Create an Auth0 Account and Application

1. Sign up for an Auth0 account at [auth0.com](https://auth0.com)
2. Create a new Application:
   - Go to Applications > Create Application
   - Name it (e.g., "ETDI Tool Registry")
   - Select "Machine to Machine" as the application type
   - Click Create

### 2. Configure Auth0 API

1. Go to Applications > APIs
2. Create a new API:
   - Name: "ETDI Tool Registry API"
   - Identifier (audience): `https://api.etdi-tools.demo.com` (or your preferred URL)
   - Signing Algorithm: RS256

3. Define API Scopes:
   - Go to the Permissions tab
   - Add these scopes:
     - `create:resource-servers`: Create new tool registrations
     - `read:resource-servers`: Read tool information
     - `update:resource-servers`: Update existing tools
     - `delete:resource-servers`: Remove tool registrations

### 3. Configure Environment Variables

Set up your environment variables:

```bash
# Auth0 Configuration
export AUTH0_DOMAIN="your-tenant.auth0.com"
export AUTH0_CLIENT_ID="your-client-id"
export AUTH0_CLIENT_SECRET="your-client-secret"
export AUTH0_AUDIENCE="https://api.etdi-tools.demo.com"
```

### 4. Initialize Auth0 in Your Code

```python
from mcp.etdi.oauth import OAuthConfig, Auth0Provider
from mcp.etdi.types import SecurityLevel

# Create OAuth configuration
oauth_config = OAuthConfig(
    provider="auth0",
    client_id=os.getenv("AUTH0_CLIENT_ID"),
    client_secret=os.getenv("AUTH0_CLIENT_SECRET"),
    domain=os.getenv("AUTH0_DOMAIN"),
    audience=os.getenv("AUTH0_AUDIENCE"),
    scopes=["create:resource-servers", "read:resource-servers"]
)

# Initialize Auth0 provider
auth0_provider = Auth0Provider(oauth_config)

# Create secure server with Auth0
server = SecureServer(
    name="my-secure-server",
    security_level=SecurityLevel.HIGH,
    oauth_provider=auth0_provider
)
```

### 5. Test Auth0 Integration

```python
# test_auth0.py
import asyncio
from mcp.etdi.oauth import OAuthConfig, Auth0Provider

async def test_auth0():
    oauth_config = OAuthConfig(
        provider="auth0",
        client_id=os.getenv("AUTH0_CLIENT_ID"),
        client_secret=os.getenv("AUTH0_CLIENT_SECRET"),
        domain=os.getenv("AUTH0_DOMAIN"),
        audience=os.getenv("AUTH0_AUDIENCE"),
        scopes=["read:resource-servers"]
    )
    
    auth0_provider = Auth0Provider(oauth_config)
    await auth0_provider.initialize()
    
    # Test token acquisition
    token = await auth0_provider.get_token(
        tool_id="test-tool",
        permissions=["read:resource-servers"]
    )
    print("✅ Successfully acquired OAuth token")
    print(f"   Token type: Bearer")
    print(f"   Scopes: {', '.join(token.scopes)}")
    
    await auth0_provider.cleanup()

if __name__ == "__main__":
    asyncio.run(test_auth0())
```

### Troubleshooting Auth0

Common issues and solutions:

1. **401 Unauthorized**:
   - Check if your client secret is correct
   - Verify the audience matches your API identifier
   - Ensure the application has the required scopes granted

2. **403 Forbidden**:
   - Check if the required scopes are enabled in your Auth0 API
   - Verify the application has been granted access to the API

3. **Invalid Grant Type**:
   - Ensure "Client Credentials" grant type is enabled in your Auth0 application settings

For more details on Auth0 integration, see the [Auth0 Integration Guide](examples/etdi/oauth_providers/auth0.md).

## Enabling Request Signing

Request signing ensures that every tool invocation and API request is cryptographically signed and verifiable, protecting against tampering and impersonation. ETDI supports RSA and ECDSA algorithms, with automatic key management.

Request signing is non-breaking and can be enabled incrementally—existing tools continue to work without modification.

### Minimal Example

```python
from mcp.etdi import SecureServer

server = SecureServer(
    name="my-secure-server",
    enable_request_signing=True,  # Enable request signing for all tools
)

@server.tool("secure_tool", etdi_require_request_signing=True)
async def secure_tool(data: str) -> str:
    return f"Signed and secure: {data}"
```

For a full end-to-end example, see [Request Signing Example](../examples/etdi/request_signing_example.py).

## Security Configuration

Configure security levels and policies:

```python
from mcp.etdi.types import SecurityPolicy, SecurityLevel

policy = SecurityPolicy(
    security_level=SecurityLevel.HIGH,
    require_tool_signatures=True,
    enable_call_chain_validation=True,
    max_call_depth=10,
    audit_all_calls=True
)

server = SecureServer(security_policy=policy)
```

## Next Steps

- [Authentication Setup](security-features.md): Configure OAuth and enterprise SSO
- [Tool Poisoning Prevention](attack-prevention.md): Protect against malicious tools
- [Examples](examples/index.md): Explore real-world examples and demos
- [Request Signing Example](examples/etdi/request_signing_example.py): See how to implement and use request signing

## Verification

Test your setup:

```bash
python examples/etdi/verify_implementation.py
```

This script will verify that ETDI is properly installed and configured.

## End-to-End ETDI Security Workflow

Follow these steps for a complete, secure ETDI deployment:

1. **Start a Secure Server**
   - Use the Quick Start or Security Configuration examples above to launch a server with ETDI security features enabled.
   - Optionally, enable request signing for all tools (see 'Enabling Request Signing' above).

2. **Run a Secure Client**
   - Use the ETDI client to discover, verify, and approve tools.
   - Example: See `examples/etdi/basic_usage.py` for a minimal client workflow.

3. **Invoke Tools Securely**
   - Invoke tools from the client. If request signing is enabled, all invocations will be cryptographically signed and verified.
   - Example: See `examples/etdi/request_signing_example.py` for client-side signing.

4. **Check Security and Audit Logs**
   - Review server and client output for verification status, approval, and audit logs.
   - Example: See `examples/etdi/verify_implementation.py` to verify your setup.

This workflow ensures that your tools are protected against tampering, impersonation, and unauthorized access, leveraging all core ETDI security features.