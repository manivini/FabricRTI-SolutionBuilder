# CI/CD Patterns Reference – Fabric RTI Demos

This document covers three CI/CD mechanisms for Microsoft Fabric workspaces and how to choose between them.

---

## Three CI/CD Mechanisms

### 1. Git Integration (Built-in)

Fabric workspaces natively sync to **GitHub** or **Azure DevOps** repos.

**How it works:**
- Workspace → Settings → Git integration
- Choose provider (GitHub / Azure DevOps) → authenticate → select repo + branch + folder
- **Bidirectional sync**: changes in Fabric commit to Git; changes in Git sync to Fabric

**What syncs:**
| Item Type | Syncs? | Notes |
|---|---|---|
| KQL Database schema | ✅ | Tables, functions, policies |
| Eventstream | ✅ | Configuration and topology |
| Notebooks | ✅ | As Fabric folder format |
| Real-Time Dashboard | ✅ | Layout + queries |
| Data Activator (Reflex) | ✅ | Rules and actions |

**Limitations:**
- Feature branches are challenging — each workspace can only connect to one branch
- No automated merge conflict resolution
- `.ipynb` files at repo root won't match Fabric's internal folder format

---

### 2. Deployment Pipelines (Built-in)

Promote items across **Dev → Test → Prod** stages without Git.

**How it works:**
- Deployment Pipelines → New pipeline → assign workspaces to stages
- Click **Deploy** to promote all (or selected) items to the next stage
- Rules can parameterize connections per stage (e.g., different Eventstream endpoints)

**Best for:**
- Teams that want stage-based promotion without Git complexity
- Demos where you need separate Dev and Prod environments

**Limitations:**
- No rollback (must re-deploy from previous stage)
- Limited to 3 stages (Dev, Test, Prod)

---

### 3. fabric-cicd (Python Package)

Programmatic deployment using the `fabric-cicd` Python package for full automation.

**How it works:**
```bash
pip install fabric-cicd
```

```python
from fabric_cicd import FabricWorkspace, publish_all_items

ws = FabricWorkspace(
    workspace_id="<workspace-guid>",
    repository_directory="./fabric-items",
    item_type_in_scope=["Notebook", "KQLDatabase", "Eventstream"]
)
publish_all_items(ws)
```

**Best for:**
- GitHub Actions / Azure Pipelines automation
- Complex environments with many workspaces
- Teams requiring full programmatic control

---

## Architecture Patterns

### Pattern A: Single Workspace + Git
```
GitHub Repo ←→ Fabric Workspace (Dev)
```
- Simplest setup, good for demos
- One branch, one workspace

### Pattern B: Git + Deployment Pipelines
```
GitHub Repo ←→ Dev Workspace → Deployment Pipeline → Prod Workspace
```
- Git for version control, Pipelines for promotion
- Most common enterprise pattern

### Pattern C: fabric-cicd Automation
```
GitHub Repo → GitHub Actions → fabric-cicd → Dev/Prod Workspaces
```
- Full CI/CD automation
- Best for large-scale or multi-tenant deployments

---

## Decision Matrix

| Factor | Git Integration | Deployment Pipelines | fabric-cicd |
|---|---|---|---|
| Setup complexity | Low | Low | Medium |
| Automation | Manual sync | Manual deploy | Full CI/CD |
| Branching | Limited | N/A | Full |
| Rollback | Git revert + sync | Re-deploy previous | Script-based |
| Best for | Solo dev / demos | Team promotion | Enterprise CI/CD |

---

## Security Considerations

- **Never commit connection strings** to Git — use Fabric environment variables or Key Vault
- **Service principals** are required for fabric-cicd in CI pipelines
- Deployment Pipelines respect **workspace roles** — only Admins can deploy
- Git integration requires **Contributor** role on the workspace

---

## Demo Flow Recommendation

For demos, use **Pattern A** (Single Workspace + Git):
1. Build everything in one workspace
2. Connect to GitHub for version control
3. Show the Git integration UI as part of the demo story
4. Mention Deployment Pipelines and fabric-cicd as enterprise scale-out options

---

## Useful Links

- [Fabric Git Integration Docs](https://learn.microsoft.com/fabric/cicd/git-integration/intro-to-git-integration)
- [Deployment Pipelines Docs](https://learn.microsoft.com/fabric/cicd/deployment-pipelines/intro-to-deployment-pipelines)
- [fabric-cicd Package](https://pypi.org/project/fabric-cicd/)
- [Fabric Toolbox (Community)](https://github.com/microsoft/fabric-toolbox)
