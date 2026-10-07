# StructuredID Proto Definitions

Protocol Buffer definitions of the StructuredID identity provider API. Every
RPC that is reachable over REST carries a `google.api.http` annotation; RPCs
without one are internal.

## Structure

```
proto/
├── buf.yaml            # Buf module and lint configuration
├── buf.gen.yaml        # TypeScript code generation (protobuf-ts)
├── google/api/         # HTTP annotation definitions
└── sid/v1/
    ├── account/        # Self-service account management
    ├── admin/          # Administration (settings, security, enrollment, keys, ...)
    ├── attestation/    # Device attestation
    ├── authn/          # Authentication: OPAQUE, WebAuthn, OAuth 2.0 / OIDC, flows,
    │                   # issuer endpoints, password history
    ├── authz/          # Authorization: RBAC + ABAC (Cedar), forward auth, governance
    ├── common/         # Error reasons, pagination, shared types
    ├── events/         # CloudEvents stream
    ├── federation/     # Upstream identity providers
    ├── identity/       # Profiles, principals, credentials, sessions
    ├── ids/            # Validated identifier messages and their conformance corpus
    ├── machine/        # Machine users and personal access tokens
    ├── projects/       # Projects, applications, OAuth clients, protected resources
    ├── scim/           # SCIM provisioning
    └── test/           # Test-only service
```

## Protocols

- **OPAQUE** (RFC 9807): password authentication without the server learning
  the password, with a zero-knowledge proof of the password policy.
- **WebAuthn** (passkeys).
- **OAuth 2.0 / OpenID Connect**: authorization code with PKCE, client
  credentials, device authorization, token exchange, introspection,
  revocation, dynamic client registration, RP-initiated logout.
- **SCIM 2.0** provisioning.

## Errors

Every refusal is a `google.rpc.Status` carrying a `google.rpc.ErrorInfo` with
domain `"structured.id"` and a reason from `sid/v1/common/errors.proto`. A
product built on this API that defines refusals of its own adds its own reason
enum under its own domain and never redefines these. Clients branch on the
code and the `(domain, reason)` pair, never on the message text.

## Code Generation

Rust code is generated at build time by the server (tonic + prost).
TypeScript clients are generated with buf and protobuf-ts into `gen/ts`;
generated code is never committed, and an application generates its own
clients from these definitions in its build:

```bash
buf generate
```

### Conformance Corpus

`sid/v1/ids/ids.corpus.json` is the language-neutral contract of the
identifier messages: for every message and case, the identifier bytes, the
binary encoding, the standard ProtoJSON and whether the checked conversion
accepts it or why it refuses. Every implementation of these identifiers runs
the same file; a message added to `ids.proto` comes with its cases.

### Lint & Breaking Change Detection

```bash
buf lint
buf breaking --against '.git#branch=main'
```

## Versioning

Package `sid.v1`. Removed fields and values are kept as `reserved`; a change
that breaks the wire contract is marked as breaking in its commit.

## License

Apache 2.0. The proto definitions are licensed under Apache 2.0 to allow
unrestricted integration by third parties; the StructuredID server is licensed
under AGPL-3.0.
