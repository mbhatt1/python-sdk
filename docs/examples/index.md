# ETDI Examples

This section contains examples and guides for using ETDI in various scenarios.

## Core Examples

- [Basic ETDI Usage](etdi/basic_usage.md): Get started with ETDI's core features
- [E2E Demo](etdi/run_e2e_demo.md): Complete end-to-end demonstration
- [Tool Poisoning Demo](etdi/tool_poisoning_demo.md): Security attack prevention

## OAuth Integration

- [OAuth Providers](etdi/oauth_providers/index.md): Integrate various OAuth providers
  - [Auth0 Integration](etdi/oauth_providers/auth0.md): Complete Auth0 setup guide

## FastMCP Integration

- [FastMCP Overview](../fastmcp/index.md): Using ETDI with FastMCP servers

## Example Categories

### Security Examples
- Tool poisoning prevention
- OAuth authentication
- Request signing
- Call chain verification

### Integration Examples
- FastMCP server integration
- Custom OAuth providers
- Event system usage
- Tool discovery

### Development Examples
- Basic tool registration
- Custom provider implementation
- Security policy configuration
- Event handling

## Running the Examples

Most examples can be run directly from the examples directory:

```bash
# Basic usage example
python examples/etdi/basic_usage.py

# Complete E2E demo
python examples/etdi/run_e2e_demo.py

# Tool poisoning prevention demo
python examples/etdi/tool_poisoning_demo/run_real_server_demo.py
```

## Example Requirements

Make sure you have:

1. Python 3.9 or higher
2. ETDI package installed
3. Required environment variables set
4. OAuth provider credentials (if using OAuth examples)

## Contributing Examples

Want to contribute a new example? Follow these steps:

1. Create your example in the appropriate directory
2. Add documentation in the docs/examples directory
3. Update this index
4. Submit a pull request

See [Contributing Guide](../../CONTRIBUTING.md) for more details. 