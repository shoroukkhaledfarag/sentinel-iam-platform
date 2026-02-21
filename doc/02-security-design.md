# Sentinel IAM Platform  
## 02 — Security Design Document  
Version: 1.0  
Status: Draft  

---

# 1. Security Architecture Overview

Sentinel IAM Platform follows a security-first architecture by delegating authentication responsibilities to Keycloak while maintaining centralized governance and authorization orchestration.

The system enforces:

- Stateless authentication using JWT
- Externalized Identity Provider (IdP)
- Strict tenant isolation (Realm per Tenant)
- Role-Based Access Control (RBAC)
- Least privilege enforcement
- Secure inter-service communication

Authentication and authorization concerns are clearly separated to reduce attack surface and complexity.

---

# 2. Authentication Model

## 2.1 Identity Provider

Authentication is fully delegated to Keycloak.

Responsibilities of Keycloak:

- User login and credential validation
- Token issuance (JWT access tokens)
- Session management
- Single Sign-On (SSO)
- LDAP federation

Sentinel IAM does not store or process user passwords.

---

## 2.2 Token Strategy

The platform uses JWT (JSON Web Tokens) for stateless authentication.

### Access Tokens

- Digitally signed using asymmetric cryptography (RS256)
- Short-lived
- Contain tenant-specific realm information
- Include role claims

### Refresh Tokens

- Managed by Keycloak
- Used to obtain new access tokens
- Not processed directly by Sentinel IAM

---

## 2.3 Supported OAuth2 Flows

### Authorization Code Flow

Used for user-based authentication:

1. User is redirected to Keycloak.
2. User authenticates.
3. Keycloak issues authorization code.
4. Code is exchanged for JWT access token.
5. Token is sent to protected APIs.

### Client Credentials Flow

Used for service-to-service communication:

1. Service authenticates using client ID and secret.
2. Keycloak issues access token.
3. Service calls another protected service with JWT.

Sentinel IAM standardizes governance but does not implement OAuth server logic.

---

# 3. Authorization Model

## 3.1 Role-Based Access Control (RBAC)

Sentinel IAM enforces Role-Based Access Control at the application layer.

Each tenant realm maintains:

- Tenant-specific roles
- Composite roles (role hierarchy)
- Group-based role assignments

Authorization enforcement occurs via:

- Method-level security annotations
- JWT role claim validation
- Least privilege principles

---

## 3.2 Tenant Isolation

Isolation is achieved through:

- Realm per tenant architecture
- Separate user stores per realm
- Separate role definitions per realm
- Independent LDAP federation configuration

This prevents cross-tenant privilege leakage.

---

# 4. Administrative Access Security

Administrative APIs exposed by Sentinel IAM are protected via:

- JWT validation
- Role-based restrictions (e.g., PLATFORM_ADMIN)
- Method-level security enforcement
- Strict input validation

Only authorized administrative roles may:

- Create tenants
- Create users
- Assign roles
- Configure federation

---

# 5. Keycloak Integration Security

## 5.1 Admin API Authentication

Sentinel IAM communicates with Keycloak Admin API using:

- Secure service account credentials
- Restricted admin-level roles
- Internal network communication

Admin credentials are:

- Stored securely via environment variables
- Never hardcoded
- Not exposed via API responses

---

## 5.2 Realm Template Governance

Tenant realm creation follows a predefined template:

- Default password policies
- Brute force protection enabled
- Required email verification (optional)
- Secure token lifespan configuration

This ensures consistent security posture across tenants.

---

# 6. LDAP Federation Security

LDAP federation is configured per tenant realm.

Security measures:

- Secure LDAP (LDAPS) recommended
- Bind credentials stored securely in Keycloak
- Controlled group-to-role mapping
- Minimal attribute exposure

Sentinel IAM does not directly authenticate against LDAP.

---

# 7. Token Validation Strategy

Protected services validate JWT tokens using:

- Public key verification
- Signature validation
- Expiration checks
- Audience validation
- Issuer validation

No session state is stored in Sentinel IAM.

This ensures horizontal scalability and stateless architecture.

---

# 8. Threat Model

## 8.1 Identified Threats

- Token theft
- Replay attacks
- Privilege escalation
- Cross-tenant access attempts
- Credential leakage
- Misconfigured realm policies

---

## 8.2 Mitigation Strategies

### Token Theft

- Short token expiration
- HTTPS enforced
- Secure storage in client applications

### Replay Attacks

- Short-lived tokens
- TLS enforcement

### Privilege Escalation

- Strict RBAC
- Composite role governance
- Least privilege enforcement

### Cross-Tenant Access

- Realm-based isolation
- Tenant-scoped authorization checks

### Credential Leakage

- Environment-based secret storage
- No sensitive logging
- Limited admin API exposure

---

# 9. Secure Configuration Practices

- No hardcoded secrets
- Environment-specific configuration
- Secure logging (no PII exposure)
- Container network isolation
- Separate database credentials per environment

---

# 10. Audit and Monitoring (Planned)

Sentinel IAM database stores:

- Tenant creation events
- Administrative role changes
- User provisioning operations

Future enhancements may include:

- Centralized audit streaming
- SIEM integration
- Alerting mechanisms

---

# 11. Security Principles Summary

The platform follows:

- Defense in depth
- Separation of concerns
- Least privilege
- Stateless authentication
- Tenant isolation by design
- Externalized identity provider model

---

End of Document
