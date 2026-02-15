# Validate All Input

## Applies To

- All application types
- All architectures (web, mobile, APIs, desktop, services)

## Summary

Validate all untrusted input using **allowlist (positive) validation** before use.
Treat all external data as untrusted, including data from users, APIs, files, queues,
headers, cookies, databases, caches, and third-party systems.

Input validation reduces the attack surface for injection, data corruption, business
logic abuse, and denial-of-service conditions.

## Why This Matters

Proper validation helps prevent or reduce:

- SQL/NoSQL/LDAP/XML/command injection
- Cross-site scripting (XSS) and content injection
- Path traversal and file handling abuse
- Integer overflows/underflows and unsafe memory behavior
- Regex denial-of-service (ReDoS)
- Logic abuse through malformed, out-of-range, or unexpected values

Validation is not a replacement for output encoding, parameterized queries, authorization,
or secure session handling. It is one essential control in a layered defense strategy.

## Core Principles

1. **Default deny.** Reject input unless it matches explicitly defined valid rules.
2. **Validate at trust boundaries.** Validate as data enters each boundary (API edge, service boundary, message consumer, persistence boundary).
3. **Use type-safe parsing first.** Parse input to strong types early (int, date, UUID, enum) rather than validating raw strings everywhere.
4. **Canonicalize before validating when needed.** Normalize encoding, Unicode form, path format, and case where relevant.
5. **Validate for context and intent.** The same field may need different rules in different workflows.
6. **Fail safely.** Return generic errors to clients; log detailed diagnostics server-side.
7. **Centralize validation logic.** Use shared validators/schemas to avoid drift and duplicate bugs.

## What to Validate

For every input field, define:

- **Required/optional** state
- **Type** (string, int, decimal, bool, enum, date/time, UUID, object, array)
- **Length/size bounds** (min/max length, max payload size, max collection size)
- **Character constraints** (allowlist of acceptable characters)
- **Format** (strict pattern, schema, or parser)
- **Range** (numeric/date limits)
- **Domain/business rules** (state transitions, ownership constraints, uniqueness, cross-field dependencies)

## Implementation Guidance

1. **Inventory all input sources.**
   Build and maintain a list of every external data source:
   request bodies, query params, path params, headers, cookies, uploaded files,
   webhooks, queue messages, imported files, and partner integrations.

2. **Define formal contracts.**
   Use request/response schemas where possible (OpenAPI/JSON Schema/Protobuf/XSD).
   Keep these contracts versioned and reviewed.

3. **Create reusable validators.**
   Implement shared validation libraries for common primitives:
   email, phone, country code, language code, IDs, date ranges, and filenames.

4. **Validate structure first, business rules second.**
   First validate shape/type/format; then enforce business constraints.

5. **Use safe parsers, not ad-hoc regex for everything.**
   Prefer built-in parsers for dates, URLs, IPs, UUIDs, and numbers.
   Use regex only for fields where it is appropriate and bounded.

6. **Bound expensive operations.**
   Limit regex complexity and input lengths before regex matching.
   Set payload size limits and parser limits to reduce DoS risk.

7. **Sanitize only when required by context.**
   Validation decides what is acceptable.
   Output encoding protects render contexts (HTML/JS/CSS/URL).
   Do not conflate the two.

8. **Handle invalid input consistently.**
   Return clear client-safe errors (e.g., `400 Bad Request`, validation code, field name).
   Do not expose stack traces, SQL errors, filesystem paths, or internal object details.

9. **Log invalid attempts securely.**
   Log enough detail for investigation (timestamp, route, field, rule violated, source IP/user/session).
   Do not log secrets, tokens, raw credentials, or sensitive PII.

10. **Test validators continuously.**
   Add unit tests for valid/invalid/boundary cases.
   Include fuzz and property-based tests for parsers and schema validation.

## High-Value Validation Patterns

- **IDs**: strict UUID/int parsing, exact length/format
- **Filenames/paths**: allowlist chars, deny absolute paths, canonicalize and enforce allowed base directory
- **Dates/times**: strict parse, timezone policy, min/max bounds
- **Numbers**: parse, range check, precision constraints
- **Enumerations**: exact set membership
- **Arrays/objects**: max item count, required keys, unknown key policy
- **Free text**: length limits, Unicode normalization, context-aware output encoding later

## Common Anti-Patterns to Avoid

- Blocklist-only filtering (easy to bypass)
- Validating only in the client
- Relying on implicit framework coercion without explicit checks
- Reusing one loose validator for multiple sensitive fields
- Accepting unknown fields silently in security-sensitive endpoints
- Returning verbose internal errors on validation failures

## Example Validation Checklist

- [ ] Every endpoint has documented input contracts
- [ ] Every input field has type, bounds, and format rules
- [ ] Payload/body size limits are enforced
- [ ] Invalid input paths are tested (negative tests)
- [ ] Validation errors are safe and consistent
- [ ] Sensitive values are excluded from logs
- [ ] Validators are centralized and reused

## Related Controls

- Parameterized queries for database access
- Output encoding by render context
- Authentication and authorization checks
- CSRF protection and session hardening
- Secure error handling and logging

## Short Operational Playbook

When adding a new endpoint:

1. Define schema and field constraints.
2. Add parser + validator tests (valid, invalid, boundary).
3. Enforce payload size and collection limits.
4. Return standardized validation error responses.
5. Add metrics/logging for repeated invalid patterns.
6. Include endpoint in security test coverage.
