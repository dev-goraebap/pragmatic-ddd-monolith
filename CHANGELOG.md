# Changelog

All notable changes to this skill are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.2.0] - 2026-06-01

### Changed
- **Realigned into a structure-first skill (FSD-analog).** Dropped the 19-command (verb) router model in favor of a self-sufficient SKILL.md whose core is the three-layer one-directional reference graph (`app → module → shared`).
- Rewrote `SKILL.md` as a complete structural decision guide: layer graph, one-line placement heuristic, capability-first `app/<resource>` slices (channels as in-slice fragments), aggregate-organized `module/<bc>`, the three `shared` scopes, variance-based DIP, shallow CQRS, folder→test mapping, and anti-patterns.
- Rewrote `README.md` with the framework-neutral philosophy (the "why": why not pure layered/hexagonal/full-CQRS, variance-based reclassification, the orchestration trade-off).

### Added
- Five consolidated, knowledge-organized references replacing the 19 stubs: `dependency-rules.md`, `app-layer.md`, `module-layer.md`, `migration.md`, `project-convention.md`. Framework notes (NestJS / Spring Boot / ASP.NET Core) preserved per file.

### Removed
- The 19 per-command reference files (teach, document, shape, craft, model, boundaries, critique, audit, refactor, distill, harden, extract, transaction, query, repository, error-model, test-strategy, policy, specification).
- Error taxonomy and test-writing tactics are no longer dictated by the skill — delegated to per-project convention (`project-convention.md` / PROJECT.md). Only the structural test mapping is retained.

## [0.1.0]

### Added
- Initial skill skeleton with router-style SKILL.md and 19 reference command stubs.
- Framework-neutral body with NestJS / Spring Boot / ASP.NET Core mapping notes per reference.
- Apache-2.0 license, npm-installable package metadata.
