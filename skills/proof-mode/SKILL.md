# Skill: Proof Mode Artifact Emission

When implementing Proof Mode:
- Emit the four required artifacts with deterministic formatting.
- Validate against zod schemas.
- No secrets; no raw diff hunks; use sha256 hashes.
- Add or update a golden fixture under /examples to snapshot outputs.
