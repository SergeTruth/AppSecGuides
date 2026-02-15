# Cryptographic Storage

## Applies To

- All applications that store sensitive data
- All deployment models (cloud, on-prem, hybrid)
- All data stores (SQL, NoSQL, object storage, file systems, backups, caches)

## Summary

Protect sensitive data at rest using strong, modern cryptography and disciplined key management.
Encryption is effective only when keys are generated, stored, rotated, and audited securely.

Cryptographic storage must be designed as a system, not treated as a single library call.

## Security Objectives

1. Preserve confidentiality of sensitive data at rest.
2. Prevent unauthorized key access and key misuse.
3. Detect tampering where integrity guarantees are required.
4. Limit blast radius via key separation and scoped access.
5. Support operational needs: rotation, recovery, auditing, and incident response.

## Data Classification First

Before encrypting, classify stored data:

- **Restricted**: credentials, auth tokens, private keys, regulated data (PCI/PII/PHI).
- **Sensitive**: business data requiring confidentiality.
- **Internal/Public**: data that may not require encryption at field level.

Use classification to drive:

- What to encrypt
- Which keys to use
- Retention policy
- Access controls
- Monitoring and alerting thresholds

## What to Encrypt

Encrypt sensitive data in all persistent locations:

- Databases (selected fields/columns and/or full-disk depending on risk)
- Backups and snapshots
- Logs that may include sensitive fields
- Caches and queue payloads
- File/object storage
- Search indexes and analytics extracts

Prefer **field-level encryption** for highly sensitive values over relying only on volume/disk encryption.

## Recommended Algorithms and Modes

- Symmetric encryption: **AES-256-GCM** (preferred AEAD)
- Alternate AEAD where appropriate: **ChaCha20-Poly1305**
- Hashing for password verification: **Argon2id** (or bcrypt/scrypt if constrained)
- Key derivation: **HKDF** or standards-based KDFs for specific protocols
- Asymmetric encryption/signing: modern vetted curves/algorithms provided by platform libraries

Avoid:

- Custom crypto implementations
- Deprecated ciphers/modes (DES, RC4, ECB, etc.)
- Homegrown “encryption” or obfuscation

## Key Management Requirements

1. **Use a managed KMS/HSM** where available.
2. **Separate keys from data** (never store raw master keys with encrypted data).
3. **Use envelope encryption**:
   - Data Encryption Keys (DEKs) encrypt data.
   - Key Encryption Keys (KEKs) in KMS/HSM protect DEKs.
4. **Scope keys by purpose and domain** (service/environment/tenant/data class).
5. **Rotate keys regularly** and support re-encryption workflows.
6. **Revoke and replace keys** quickly during suspected compromise.
7. **Enforce least privilege** on key usage permissions.
8. **Audit all key operations** (encrypt/decrypt/generate/rotate/disable/delete).

## Integrity and Authenticity

Use authenticated encryption (AEAD) whenever possible.
If AEAD is not available for a required legacy flow, use an approved Encrypt-then-MAC design.

Protect against:

- Ciphertext tampering
- Replay where relevant
- Context confusion (bind encryption context/associated data such as record ID, tenant ID, and version)

## Implementation Guidance

1. **Define cryptographic boundaries.**
   Identify exactly where plaintext is allowed in memory and where ciphertext must exist.

2. **Create a cryptography service layer.**
   Centralize encrypt/decrypt operations and policy decisions.
   Do not spread ad hoc crypto calls across business code.

3. **Version your ciphertext format.**
   Include algorithm, key ID/reference, nonce/IV, and version metadata to support migration.

4. **Generate unique nonces/IVs correctly.**
   Never reuse nonce/IV with the same key where prohibited by the algorithm.

5. **Minimize plaintext exposure.**
   Decrypt as late as possible, keep decrypted values in memory briefly, and avoid logging plaintext.

6. **Handle failures safely.**
   On decrypt/authentication failure, fail closed and emit sanitized errors.

7. **Secure backups and exports.**
   Ensure backup pipelines preserve encryption and key-access controls.

8. **Test cryptographic behavior.**
   Add tests for key rotation, decrypt compatibility, tamper detection, and corrupted metadata.

## Operational Checklist

- [ ] Sensitive fields are inventoried and documented
- [ ] Strong AEAD algorithm is used
- [ ] Keys are managed by KMS/HSM
- [ ] Envelope encryption is implemented
- [ ] Key access is least-privileged and audited
- [ ] Key rotation and re-encryption procedures are automated/tested
- [ ] Backups/snapshots are encrypted and access-controlled
- [ ] Logs and telemetry do not expose plaintext secrets
- [ ] Incident playbook exists for key compromise

## Common Anti-Patterns

- Encrypting with one static global key for everything
- Storing master keys in application config or source control
- Reusing IVs/nonces incorrectly
- Logging decrypted data for debugging
- Treating disk encryption alone as sufficient for all threat models
- Failing to plan for key rotation and ciphertext migration

## Minimal Incident Response Plan (Crypto Compromise)

1. Disable or restrict affected keys immediately.
2. Identify impacted datasets/services and exposure window.
3. Rotate keys and re-encrypt affected data.
4. Invalidate affected secrets/tokens where applicable.
5. Review access logs and root cause.
6. Harden controls and update runbooks/tests.

## Related Controls

- Secure secret management
- Access control and authorization
- Secure logging and monitoring
- Data retention and minimization
- Backup and disaster recovery controls

