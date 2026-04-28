# Deployment Pipeline — NYC311 Data Platform

## Overview

Environment promotion for the NYC311 data platform is managed through a **Fabric-native deployment pipeline** with three stages: Development, Test, and Production. The pipeline is triggered automatically on merge to the `main` branch and requires human review and approval between each stage transition to ensure quality gates are enforced before production changes go live.

---

## Pipeline Stages

### Development

The Development stage is the active authoring environment. It maps to the `NYC311-Fabric` workspace and reflects the current state of the `main` branch in the connected GitHub repository. All feature work is done in isolated feature branches and merged to `main` via pull request — at the point of merge, the Development workspace is considered the validated baseline for promotion.

The Development stage contains the full workspace artifact set: Lakehouse, notebooks, pipeline, semantic model, and reports. Changes deployed here are considered stable enough for functional testing but not yet production-validated.

### Test

The Test stage is promoted to from Development after a mandatory reviewer approval. At least one reviewer must approve the Development → Test deployment before Fabric executes the stage copy.

Test is used for integration validation — confirming that pipeline execution, notebook logic, semantic model refresh, and report rendering all behave correctly after promotion. The Lakehouse in the Test stage shares the same schema as Development, allowing realistic data validation against representative data volumes.

Any issues identified in Test result in fixes being made in a new feature branch, merged to `main`, and re-deployed to Development before re-promotion to Test.

### Production

Promotion from Test to Production follows the same reviewer approval gate. Production is the live environment — it is the stage from which end users and report consumers access data. Deployments to Production are expected to be infrequent and represent fully validated releases.

---

## Reviewer Gates

Reviewer approval is required at two transition points:

- **Development → Test** — confirms that the change set in Development is ready for integration testing
- **Test → Production** — confirms that Test validation has passed and the release is approved for live deployment

Reviewers are assigned within the Fabric deployment pipeline configuration. The approval workflow is completed within the Fabric portal before the deployment proceeds. This gate prevents unreviewed notebook or model changes from reaching Production and provides an audit trail of who approved each release.

---

## What Gets Deployed

Fabric deployment pipelines copy the following item types across stages:

- Fabric pipeline definitions (`NYC311_API_Incremental`)
- Notebook source code (`Ingest_API_Live`, `Transform_Bronze_to_Silver`, `Transform_Silver_to_Gold`)
- Semantic model definition (tables, relationships, measures, storage mode configuration)
- Power BI report definitions

The **Lakehouse** is not duplicated by the deployment pipeline — it is a single shared instance. Data state is not promoted between stages; only item definitions (code, model, reports) move through the pipeline.

---

## Relationship to Git Workflow

The deployment pipeline works in tandem with the Git branching strategy rather than as a replacement for it. Git manages source history, code review, and version control. The Fabric deployment pipeline manages environment promotion and enforces review gates at the infrastructure layer.

The intended flow is:

```
feature branch
      │  PR review + approval
      ▼
    main  ──► triggers Fabric workspace sync (Development)
      │        │
      │        ▼  reviewer approval required
      │      Test
      │        │  reviewer approval required
      │        ▼
      │    Production
```

See [Git Branching Strategy](../../04-devops/docs/git-branching-strategy.md) for the full source control workflow.

---

## Related Documentation

- [Git Branching Strategy](../../04-devops/docs/git-branching-strategy.md)
- [Fabric Environment Setup](./fabric-environment-setup.md)
- [Fabric Pipeline Orchestration](../../04-devops/docs/fabric-pipeline.md)
