# Sentinel IAM Platform  
## 01 — Architecture Overview  
Version: 1.0  
Status: Draft  

---

# 1. Architectural Vision

Sentinel IAM Platform is a centralized, multi-tenant Identity and Access Management (IAM) orchestration layer. It provides enterprise-grade authentication governance and authorization management by integrating with Keycloak as the external Identity Provider (IdP).

The architecture enforces strict separation of responsibilities:

- Keycloak handles authentication, token issuance, session management, and LDAP federation.
- Sentinel IAM orchestrates tenant lifecycle and RBAC modeling.
- Protected microservices validate JWT tokens issued by Keycloak.
- LDAP provides external directory federation per tenant realm.

The system is designed for scalability, isolation, and security-first operation.

---

# 2. System Context

Sentinel IAM operates within an enterprise service ecosystem composed of users, microservices, and external identity infrastructure.

## External Actors

- Enterprise End Users  
- Backend Microservices  
- Enterprise LDAP Directory  
- System Administrators  

Sentinel IAM does not participate directly in user login flows. Authentication is fully delegated to Keycloak.

### High-Level Interaction

Enterprise Users → Keycloak (per-tenant realm)  
Microservices → Validate JWT tokens  
Sentinel IAM → Manages tenants and RBAC  
LDAP → Federated identity source  

---

# 3. High-Level Container Architecture

The platform consists of the following core containers:

- Client Applications  
- Keycloak (Realm per Tenant)  
- Protected Microservices (Resource Servers)  
- Sentinel IAM Service  
- PostgreSQL Database (Sentinel Metadata)  
- LDAP Directory  

### Logical Flow

1. Client applications authenticate via Keycloak.
2. Keycloak issues JWT access tokens.
3. Clients call protected microservices.
4. Microservices validate JWT tokens.
5. Sentinel IAM manages tenant provisioning and RBAC governance.

Sentinel IAM communicates with:

- Keycloak (Admin API)
- PostgreSQL (tenant metadata storage)
- LDAP (via Keycloak federation)

---

# 4. Internal Component Architecture (Sentinel IAM)

Sentinel IAM follows a layered architecture to ensure maintainability and separation of concerns.

## 4.1 Controller Layer

Responsibilities:

- Expose REST APIs for tenant and RBAC management  
- Validate request payloads  
- Enforce administrative access control  
- Return standardized responses  

Example endpoints:

- `POST /tenants`
- `POST /tenants/{tenantId}/users`
- `POST /tenants/{tenantId}/roles`

---

## 4.2 Service Layer

Responsibilities:

- Execute business logic  
- Handle tenant onboarding workflow  
- Manage role hierarchy modeling  
- Coordinate between internal database and Keycloak operations  

---

## 4.3 Keycloak Admin Client Layer

Responsibilities:

- Communicate with Keycloak Admin API  
- Create and configure realms  
- Manage users, roles, and groups  
- Configure LDAP federation  
- Apply realm templates  

This layer abstracts Keycloak integration from the rest of the system.

---

## 4.4 Security Layer

Responsibilities:

- Configure JWT validation  
- Define resource server behavior  
- Enforce method-level security  
- Protect administrative endpoints  

Sentinel IAM itself operates as a secured resource server.

---

## 4.5 Repository Layer

Responsibilities:

- Store tenant metadata  
- Maintain realm mapping  
- Store administrative audit logs  
- Persist system configuration settings  

The repository layer interacts with PostgreSQL.

---

# 5. Multi-Tenancy Model

## 5.1 Strategy: Realm per Tenant

Each tenant is provisioned with a dedicated Keycloak realm.

### Benefits

- Strong isolation between tenants  
- Independent role and group modeling  
- Independent LDAP federation configuration  
- Reduced risk of privilege leakage  
- Clear security boundaries  

Each realm contains:

- Tenant users  
- Tenant roles  
- Tenant groups  
- Tenant-specific identity federation configuration  

---

## 5.2 Tenant Onboarding Flow

1. Administrator calls `POST /tenants`
2. Sentinel creates a new realm in Keycloak
3. Realm template configuration is applied
4. LDAP federation is configured (if enabled)
5. Default roles and groups are created
6. Tenant metadata is stored in Sentinel database

This ensures consistent and repeatable tenant provisioning.

---

# 6. Authentication Architecture

Authentication is fully delegated to Keycloak.

## 6.1 User Authentication Flow (Authorization Code Flow)

1. User accesses application  
2. Application redirects to Keycloak tenant realm  
3. User authenticates  
4. Keycloak issues JWT access token  
5. Application sends token to protected API  
6. Protected API validates token signature  

Sentinel IAM does not process login credentials directly.

---

# 7. Service-to-Service Authentication

## 7.1 Client Credentials Flow

Used for machine-to-machine communication.

1. Service A requests token from Keycloak  
2. Keycloak issues access token  
3. Service A calls Service B with token  
4. Service B validates JWT using public key  

Sentinel IAM standardizes governance but does not issue tokens.

---

# 8. Administrative Operation Flow

## 8.1 Create Tenant

- Sentinel receives tenant creation request  
- Calls Keycloak Admin API  
- Creates new realm  
- Applies realm configuration template  
- Configures LDAP federation  
- Creates default RBAC structure  
- Stores tenant metadata  

---

## 8.2 Create User

- Administrator calls Sentinel API  
- Sentinel calls Keycloak Admin API for tenant realm  
- User is created  
- Roles are assigned  
- Optional group mapping is applied  

---

## 8.3 Create Role

- Sentinel creates role in tenant realm  
- Optionally creates composite roles for hierarchy  
- Updates internal metadata if required  

---

# 9. Data Storage Strategy

Sentinel IAM maintains its own PostgreSQL database.

Stored data includes:

- Tenant registry  
- Realm mapping  
- Administrative audit logs  
- System configuration settings  

Sensitive credentials and passwords remain managed exclusively by Keycloak.

---

# 10. Security Principles

The architecture enforces:

- Stateless authentication model  
- Externalized identity provider  
- Strict tenant isolation  
- Least privilege access control  
- Secure inter-service communication  
- Clear separation between authentication and authorization orchestration  

---

# 11. Deployment Architecture

The platform is containerized using Docker.

Core containers:

- sentinel-iam-service  
- keycloak  
- postgresql  
- ldap  

All services communicate over an internal Docker network.

The architecture is extensible to orchestration platforms such as Kubernetes.

---

# 12. Architectural Characteristics

- Modular and extensible  
- Horizontally scalable  
- Multi-tenant isolated  
- Security-first design  
- Enterprise-ready infrastructure component  

---

End of Document
