<!--
================================================================================
SYNC IMPACT REPORT
================================================================================
Version change: 0.0.0 → 1.0.0 (MAJOR - initial constitution adoption)

Modified principles: N/A (initial creation)

Added sections:
  - Core Principles (3): Privacy-First, Test-First, Simplicity
  - Technical Constraints (Security, Performance, Cost Efficiency)
  - Development Workflow
  - Governance

Removed sections: N/A

Templates requiring updates:
  ✅ plan-template.md - Constitution Check section compatible
  ✅ spec-template.md - Requirements structure compatible
  ✅ tasks-template.md - Phase structure compatible

Follow-up TODOs: None
================================================================================
-->

# AI Photo Cleaner Constitution

## Core Principles

### I. Privacy-First & Local-First

All photo processing MUST occur locally on the user's machine by default. User data
(photos, metadata, face embeddings) MUST NOT be transmitted to external services
without explicit user consent.

**Non-negotiable rules:**
- Photos MUST be processed using local inference models (ONNX runtime)
- OAuth tokens MUST be stored securely in local SQLite with encryption
- No telemetry or analytics that transmits user photo data externally
- Cloud features (Phase 2) MUST be opt-in, never default
- Face embeddings MUST remain local; cloud sync requires explicit user action

**Rationale:** Users trust this tool with their personal photo libraries. Privacy
violations would destroy that trust and potentially expose sensitive personal moments.

### II. Test-First Development (NON-NEGOTIABLE)

Test-Driven Development is mandatory for all feature implementation. Tests MUST be
written and verified to fail before implementation code is written.

**Non-negotiable rules:**
- Red-Green-Refactor cycle strictly enforced
- Tests MUST be written first → User approved → Tests fail → Then implement
- Contract tests required for all API endpoints
- Integration tests required for Google Photos API interactions
- Unit tests required for ML inference pipelines (blur detection, face clustering)
- No PR may be merged with failing tests

**Rationale:** Photo management operations are destructive (deletion, moves). Bugs
could result in permanent loss of irreplaceable memories. TDD prevents regressions.

### III. Simplicity & YAGNI

Start with the simplest solution that works. Avoid over-engineering and premature
abstraction. Only build what is explicitly needed for the current phase.

**Non-negotiable rules:**
- Phase 1 local-only features MUST NOT include cloud infrastructure code
- No feature flags for hypothetical future requirements
- Three similar lines of code is better than a premature abstraction
- Maximum 3 projects/modules until complexity is explicitly justified
- Avoid backwards-compatibility shims; if something is unused, delete it
- Configuration MUST use sensible defaults; advanced settings are optional

**Rationale:** Complexity creates bugs, slows development, and makes maintenance
harder. The PRD already defines a phased approach—respect those boundaries.

## Technical Constraints

### Security Standards

- OAuth2 tokens MUST use secure storage (encrypted SQLite or system keychain)
- Refresh tokens MUST NOT be logged or exposed in error messages
- All user inputs MUST be validated before use in API calls or database queries
- Google Photos API credentials MUST be stored in environment variables, never in code
- SQLite database files MUST have restricted file permissions (600)

### Performance Targets

- Photo scanning MUST use async/concurrent requests (batch Google API calls)
- ML inference MUST use ONNX runtime for cross-platform performance
- Duplicate detection MUST use perceptual hashing with O(n log n) comparison
- UI MUST handle 500+ photos in batch view without per-photo interaction
- Database queries MUST be indexed for common access patterns

### Cost Efficiency

- Local inference MUST be preferred over cloud APIs to minimize ongoing costs
- When cloud inference is required, MUST evaluate low-cost providers (OpenRouter, etc.)
- Google Photos API usage MUST respect quota limits through batching and caching
- ML models MUST be <200MB total; larger models require explicit justification
- Development infrastructure MUST use free tiers where possible (SQLite over hosted DB)

## Development Workflow

### Code Review Requirements

- All PRs MUST pass automated tests before review
- PRs MUST include test coverage for new functionality
- Security-sensitive changes (OAuth, token handling) require explicit review focus
- Breaking API changes require migration documentation

### Quality Gates

- Linting and formatting MUST pass (configured tooling enforced)
- Type checking MUST pass (Python type hints, TypeScript strict mode)
- Test coverage MUST NOT decrease for core modules (inference, API, storage)
- No TODO comments without linked issue tracking

## Governance

This constitution supersedes all other development practices for the AI Photo Cleaner
project. All contributors MUST adhere to these principles.

**Amendment Procedure:**
1. Proposed amendments MUST be documented with rationale
2. Amendments MUST be reviewed and approved before implementation
3. Version number MUST be incremented according to semantic versioning:
   - MAJOR: Backward incompatible principle removals or redefinitions
   - MINOR: New principle/section added or materially expanded
   - PATCH: Clarifications, wording, non-semantic refinements
4. All dependent templates MUST be updated to reflect amendments

**Compliance Review:**
- PRs MUST verify compliance with Core Principles before merge
- Complexity additions MUST be justified against Simplicity principle
- Security-sensitive features MUST be reviewed against Security Standards

**Runtime Guidance:**
- Implementation details that may change frequently SHOULD be documented in
  feature-specific docs rather than this constitution
- This constitution defines principles; the PRD defines features

**Version**: 1.0.0 | **Ratified**: 2025-12-31 | **Last Amended**: 2025-12-31
