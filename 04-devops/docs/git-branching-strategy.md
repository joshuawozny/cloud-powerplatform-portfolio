# Git Branching Strategy — NYC311 Data Platform

## Overview

The NYC311 data platform uses a **feature branch workflow** managed through GitHub. All development work occurs in isolated feature branches, with changes promoted to the `main` branch via pull request. Merges to `main` trigger the Fabric deployment pipeline, making Git the source of truth for all platform artifact definitions.

Source control is managed locally using **VS Code** with the built-in Git integration for staging, committing, and pushing changes, and the GitHub pull request extension for PR creation and review.

---

## Branch Structure

### `main`

The `main` branch represents the production-validated state of the platform. Direct commits to `main` are not permitted — all changes arrive via pull request from a feature branch. On merge to `main`, the Fabric workspace Git integration syncs the updated definitions to the Development stage of the deployment pipeline.

### Feature Branches

All active development occurs in feature branches created from `main`. Branch naming follows a descriptive convention reflecting the scope of the change:

```
feature/incremental-api-ingestion
feature/gold-aggregation-tables
feature/semantic-model-storage-modes
fix/silver-null-handling
docs/lakehouse-architecture
```

Feature branches are short-lived — they are created for a specific piece of work, reviewed, merged, and deleted. Long-running branches are avoided to minimize merge conflicts and keep the change surface of each PR small and reviewable.

---

## Pull Request Workflow

All merges to `main` require a pull request. The PR process enforces code review before any changes reach the Fabric Development workspace.

**Creating a PR** — once feature branch work is complete and committed, a PR is opened from the feature branch targeting `main`. The PR description summarizes the change, references any relevant issues or requirements, and notes any testing performed locally.

**Review** — at least one reviewer must approve the PR before merge is permitted. Reviewers are expected to examine notebook logic, pipeline configuration changes, and semantic model definition changes for correctness and consistency with platform standards.

**Merge** — on approval, the PR is merged using a squash merge to keep the `main` branch history clean. The feature branch is deleted after merge.

**Post-merge sync** — the Fabric workspace Git integration detects the change on `main` and prompts a workspace update, syncing the latest notebook, pipeline, and model definitions to the Development stage.

---

## VS Code Workflow

Day-to-day development uses VS Code as the Git client. The typical local workflow is:

1. Pull latest `main` to ensure the local baseline is current
2. Create a new feature branch from `main`
3. Make changes to notebooks, pipeline definitions, or model files within the branch
4. Stage and commit changes incrementally with descriptive commit messages
5. Push the feature branch to GitHub
6. Open a pull request from GitHub or the VS Code GitHub Pull Requests extension
7. Address any review feedback with additional commits to the feature branch
8. Merge on approval

VS Code's diff viewer is particularly useful for reviewing changes to serialized Fabric item definitions (JSON/TMDL files) before committing, catching unintended side effects of model or pipeline edits.

---

## What Is Version-Controlled

The Fabric workspace Git integration serializes the following artifact types to the repository:

| Artifact | Serialization Format |
|----------|---------------------|
| Fabric pipeline | JSON activity definitions |
| Notebooks | `.py` source files |
| Semantic model | TMDL (Tabular Model Definition Language) files |
| Power BI reports | Report definition JSON |

The Lakehouse schema (Delta table structure) is defined by the notebook transformation logic and is therefore implicitly version-controlled through notebook source — no separate schema migration files are maintained.

---

## Relationship to Deployment Pipeline

Git branching governs code quality and review. The Fabric deployment pipeline governs environment promotion. They operate as complementary controls:

- Git enforces that no unreviewed code reaches Development
- The deployment pipeline enforces that no unreviewed promotion reaches Test or Production

Neither can be bypassed without circumventing both controls, providing a layered governance model appropriate for a production data platform.

See [Deployment Pipeline](../../01-power-platform/docs/deployment-pipeline.md) for the full promotion workflow.

---

## Related Documentation

- [Deployment Pipeline](../../01-power-platform/docs/deployment-pipeline.md)
- [Fabric Pipeline Orchestration](./fabric-pipeline.md)
- [Fabric Environment Setup](../../01-power-platform/docs/fabric-environment-setup.md)
