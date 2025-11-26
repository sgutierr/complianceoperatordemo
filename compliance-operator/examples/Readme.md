https://ralvares.github.io/openshift-security-framework/docs/html/compliance-operator-customization.html

## Ejemplos de CustomRule

### Demo 1: Security Checks Básicos
- Audit cluster-admin Access with an Allow-List (aligns to CIS 5.1.1)
- Discover Unapproved (Shadow) Databases
- Verify Allowed Registries Configured

### Demo 2: NetworkPolicy Security Checks
- Detect allow-all NetworkPolicies in labeled namespaces
- Require a deny-all NetworkPolicy in labeled namespaces

### Demo 3: Container and Pod Security Best Practices
- No latest image tags
- Resource limits enforcement
- Non-root containers
- Secret management
- NetworkPolicy for critical namespaces
- Readiness probes
- Host network restrictions
- Privileged mode restrictions
- Pod Security Standards

Ver el README.md en cada directorio para más detalles.