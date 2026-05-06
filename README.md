# StructuredID Proto Definitions (CE)

Protocol Buffer definitions for StructuredID Community Edition.

## Structure

```
proto/
├── buf.yaml          # Buf configuration
├── buf.gen.yaml      # Code generation config
└── sid/
    └── v1/
        ├── identity.proto  # Profile, credentials, sessions
        ├── auth.proto      # Authentication flows
        └── authz.proto     # Authorization (RBAC, policies)
```

## Services

### IdentityService (`sid/v1/identity.proto`)
Manages profile lifecycle, credentials, and sessions.

**Key entities:**
- `Profile` — user identity (CE: Profile IS the user, no separate User entity)
- `Credential` — authentication factor (OPAQUE, WebAuthn, TOTP, recovery codes)
- `Session` — active user session with scopes

**RPC methods:**
- Profile CRUD: `CreateProfile`, `GetProfile`, `UpdateProfile`, `DeleteProfile`, `ListProfiles`
- Credential management: `AddCredential`, `ListCredentials`, `RevokeCredential`
- Session management: `ListSessions`, `RevokeSession`

### AuthService (`sid/v1/auth.proto`)
Handles authentication flows.

**Protocols:**
- **OPAQUE** (RFC 9807): Zero-knowledge password authentication
  - 2-step registration: `OpaqueRegistrationStart` → `OpaqueRegistrationFinish`
  - 2-step login: `OpaqueLoginStart` → `OpaqueLoginFinish`
- **WebAuthn** (passkeys): FIDO2 authentication
  - 2-step registration: `WebAuthnRegistrationStart` → `WebAuthnRegistrationFinish`
  - 2-step authentication: `WebAuthnAuthenticationStart` → `WebAuthnAuthenticationFinish`
- **OAuth2 / OIDC**: Standard flows
  - `OAuth2Authorize`, `OAuth2Token`, `OAuth2Introspect`, `OAuth2Revoke`
- **Session validation**: `ValidateSession`

### AuthzService (`sid/v1/authz.proto`)
Handles authorization and policy evaluation.

**Key entities:**
- `Role` — named collection of permissions
- `Group` — collection of profiles
- `RoleAssignment` — assigns role to profile/group with optional scope and expiry
- `Policy` — Cedar ABAC policy

**RPC methods:**
- Permission check: `CheckPermission`, `BatchCheckPermission`
- Role management: `CreateRole`, `GetRole`, `UpdateRole`, `DeleteRole`, `ListRoles`
- Role assignment: `AssignRole`, `RevokeRole`, `ListRoleAssignments`
- Group management: `CreateGroup`, `GetGroup`, `UpdateGroup`, `DeleteGroup`, `ListGroups`
- Group membership: `AddToGroup`, `RemoveFromGroup`, `ListGroupMembers`
- Policy management: `CreatePolicy`, `GetPolicy`, `UpdatePolicy`, `DeletePolicy`, `ListPolicies`

## Code Generation

### Prerequisites

Install buf and code generators:

```bash
# Install buf
brew install bufbuild/buf/buf

# Install Rust generators
cargo install protoc-gen-tonic protoc-gen-prost

# Install TypeScript generator
npm install -g @protobuf-ts/plugin

# Install doc generator
go install github.com/pseudomuto/protoc-gen-doc/cmd/protoc-gen-doc@latest
```

### Generate Code

```bash
# From proto/ directory
buf generate

# Generated files:
# - ../sid/crates/sid-proto/src/*.rs (Rust)
# - ../admin-ui/src/generated/*.ts (TypeScript)
# - docs/api.md (Documentation)
```

### Lint & Breaking Change Detection

```bash
# Lint proto files
buf lint

# Check for breaking changes (against main)
buf breaking --against '.git#branch=main'
```

## Versioning

**Current version:** `sid/v1`

**Breaking change policy:**
- `sid/v1/` — NEVER break (semver MAJOR bump required)
- Deprecate old fields, add new fields/RPCs
- Use `optional` for new fields to maintain backward compatibility

## License

Apache 2.0

> Note: Proto definitions are licensed under Apache 2.0 to allow unrestricted integration by third parties. The StructuredID CE server binary is licensed under AGPL-3.0.
