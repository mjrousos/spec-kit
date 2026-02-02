---
name: specflow-upgrading
description: Upgrading SpecFlow dependencies to Reqnroll while upgrading from .NET Framework to .NET Core.
---

# Upgrading SpecFlow Dependencies

SpecFlow is a popular Behavior-Driven Development (BDD) framework for .NET applications. When upgrading .NET Framework applications that use SpecFlow to .NET Core, it is recommended to transition from SpecFlow to Reqnroll, a modern BDD framework that supports .NET Core.

> Source of truth: Reqnroll's official migration guide and docs. ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))

---

## 0 - Overview & Strategy

There are **two supported paths**:

1. **Compatibility Package path** (fastest, few/no code changes) – swap NuGets, keep `TechTalk.SpecFlow` usings via a shim package. ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))
2. **Full Namespace path** (cleanest) – swap NuGets and **replace namespaces** (`TechTalk.SpecFlow` → `Reqnroll`), plus a couple of symbol renames. ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))

> Reqnroll is based on **SpecFlow v4**; migrating from **SpecFlow v3** may surface v3→v4 breaking changes (Cucumber Expressions behavior, removed "call step from step", etc.). ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))

This plan lets an AI automation tool choose either path per repo/solution and includes verifications, fallbacks, and CI notes.

### Decision Matrix – Automating Path Selection

Use this matrix to programmatically decide between the **Compatibility Package path** and the **Full Namespace path**. Score each criterion; choose Compatibility if it clearly dominates for speed/low risk, otherwise prefer Full Namespace for long-term cleanliness.

| Criterion | Low Complexity / Favor Full Namespace (score +1 Full) | High Complexity / Favor Compatibility (score +1 Compat) | Notes / Automation Signal |
|-----------|--------------------------------------------------------|----------------------------------------------------------|---------------------------|
| SpecFlow package count | ≤3 (core + test adapter only) | >3 (multiple plugins, Actions, LivingDoc) | Count `SpecFlow.*` references in `.csproj` |
| Custom DI (BoDi) usage | None detected | Heavy `IObjectContainer` usages | Grep `using BoDi;` / `IObjectContainer` |
| Output helper usage | Not used | Multiple `ISpecFlowOutputHelper` injections | Grep symbol |
| SpecFlow.Actions usage | None | Present (e.g., Selenium, Playwright) | Map to Reqnroll compatibility Actions |
| Feature file count | <50 | ≥50 | Count `*.feature` files |
| Namespaces in code | Centralized step project only | Spread across many projects | Number of projects containing `TechTalk.SpecFlow` |
| Time-to-upgrade constraint (input flag) | Flexible | Urgent | External user input to tool |
| Need quick rollback | Not critical | Critical (must revert fast) | If true, lean Compatibility |
| CI pipeline stability | Green & simple | Flaky / complex filters | If unstable, reduce change surface |
| Version gap (SpecFlow v3→v4 delta risk) | Already on v4 | On v3 with many legacy patterns | Detect removed patterns (e.g., step re-invocation) |
| LivingDoc dependency | Not needed | Required in reports | If required, Compatibility eases stopgap |
| Multirepo consistency goal | Desire unified approach | Heterogeneous / pilot phase | Early pilot favors Compatibility |

#### Scoring Logic (Pseudo-code)

```
fullScore = sum(criteria favoring Full)
compatScore = sum(criteria favoring Compatibility)

if compatScore - fullScore >= 3:
    path = "Compatibility"
elif fullScore - compatScore >= 2:
    path = "FullNamespace"
else:
    # Tie-breakers
    if timeConstraint == "urgent": path = "Compatibility"
    elif longTermMaintainabilityPriority: path = "FullNamespace"
    else: path = "Compatibility"  # default conservative
```

#### JSON Decision Artifact Example

```json
{
  "project": "BowlingGame.Specs",
  "metrics": {
    "specFlowPackageCount": 3,
    "actionsPackages": [],
    "boDiUsages": 0,
    "outputHelperUsages": 0,
    "featureFileCount": 1,
    "techTalkNamespaceProjects": 1,
    "specFlowVersionMajor": 3,
    "livingDocRequired": false,
    "ciStable": true,
    "timeConstraint": "normal",
    "rollbackCritical": false,
    "longTermMaintainabilityPriority": true
  },
  "scoring": {
    "fullScore": 6,
    "compatScore": 1
  },
  "selectedPath": "FullNamespace",
  "rationale": [
    "Few packages and minimal framework surface",
    "No BoDi or output helper complexity",
    "Maintainability prioritized"
  ]
}
```

Automation should persist this artifact before performing transformations.

---

## 1 - Preconditions & Discovery (AI Tasks)

* **Detect test framework** per project: MSTest / NUnit / xUnit.

  * Look for `MSTest.TestFramework`, `nunit`, `xunit` refs in `.csproj`.
* **Detect SpecFlow usage**: any `SpecFlow.*` packages or `TechTalk.SpecFlow` in code.
* **Detect config**: `specflow.json` or legacy `App.config` SpecFlow section.
* **Detect `SpecFlow.Actions.*`** packages; note equivalent Reqnroll Actions shims. ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))
* **Detect BoDi usages** (`using BoDi;`, `IObjectContainer`). ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))
* **Detect ISpecFlowOutputHelper** usage (will be `IReqnrollOutputHelper` after full migration). ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))
* **Detect Scenario Outlines** if MSTest is used (row-test behavior nuance). ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))
* **Detect SpecFlow LivingDoc** usage (doc generation expectations). ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))

**Exit criteria:** Inventory JSON artifact listing projects, packages, config files, metrics above.

---

## 2 - Package Changes (both paths)

For each SpecFlow test project:

* **Remove**: all `SpecFlow.*` & `CucumberExpressions.SpecFlow.*` packages. ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))
* **Add** (pick 1 per framework):

  * `Reqnroll.MsTest` **or** `Reqnroll.NUnit` **or** `Reqnroll.xUnit` (plus test SDK packages already in use). ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))
* **If Compatibility path**: also **add** `Reqnroll.SpecFlowCompatibility`.

  * If any `SpecFlow.Actions.*` were present, add the matching `Reqnroll.SpecFlowCompatibility.Actions.*` (e.g., Selenium). ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))

> The Reqnroll MSBuild generator replaces SpecFlow's generator and handles `*.feature` → `*.cs` generation. ([docs.reqnroll.net PDF](https://docs.reqnroll.net/_/downloads/en/latest/pdf/))

**Acceptance:** Project builds; feature-code-behind files generated.

---

## 3 - Namespaces & Symbols (Full Namespace path only)

* **Replace all** `TechTalk.SpecFlow` → `Reqnroll` (match case & whole word). ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))
* **BoDi** DI container namespace: `BoDi` → `Reqnroll.BoDi`. ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))
* **Output helper**: `ISpecFlowOutputHelper` → `IReqnrollOutputHelper`. ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))
* **Assist helpers**: main extensions live under `Reqnroll` now (using may simplify). ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))
* **Table alias**: `DataTable` alias exists; `Table` still works (no change required). ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))

**Acceptance:** Project compiles without `TechTalk.SpecFlow` or `BoDi` namespace errors.

---

## 4 - Configuration Migration

* Reqnroll uses **`reqnroll.json`**; it also honors **`specflow.json`** unchanged. Quick path: keep `specflow.json` now; rename later. ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))
* **Full migration**: rename `specflow.json` → `reqnroll.json` and update schema to
  `https://schemas.reqnroll.net/reqnroll-config-latest.json`.

  * `stepAssemblies` → `bindingAssemblies` (rename recommended).
  * `bindingCulture/name` → `language/binding`. ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))
* **If legacy `App.config`** was used, port settings into `reqnroll.json` per configuration docs. ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))

**Acceptance:** Running tests show configuration applied (e.g., language/binding, bindingAssemblies).

---

## 5 - Breaking-Change Checks (v3→v4 lineage)

* **Cucumber Expressions detection**

  * If regex steps get misdetected as Cucumber Expressions, **force regex** with `^ $` in step attributes, or update to valid Cucumber Expressions, or adjust global detection per guide.
  * Bulk-fix regex markers with the documented regex replace, when needed. ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))
* **Removed "call step from step" API** (removed in v4; not available in Reqnroll). Refactor to helper methods. ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))

**Acceptance:** No 'matching step definition found' errors from mis-detected expressions; no usage of removed step-calling pattern remains.

---

## 6 - MSTest Scenario Outlines (Row Tests)

* Reqnroll generates **data-driven row tests** for Scenario Outlines with MSTest. Some tooling (e.g., Azure Pipelines `VSTest` with distributed execution or filtering) may need adjustments; if incompatible, **set** `generator/allowRowTests=false` in `reqnroll.json` to revert to SpecFlow-like behavior. ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))

**Acceptance:** Scenario Outline examples execute as intended in local + CI runs; filtering/reporting behaves as required.

---

## 7 - IDE & Plugins

* **Visual Studio extension**: Reqnroll VSIX handles both SpecFlow & Reqnroll projects and .NET 8. Install/verify. ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))
* **Plugins**: Replace SpecFlow-managed integrations with `Reqnroll.*` equivalents (e.g., Autofac). ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))
* **Living Documentation**: SpecFlow+ LivingDoc was closed-source; see the **workaround** thread for using the SpecFlow LivingDoc CLI with Reqnroll until a replacement is shipped. ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))

---

## 8 - CI/CD Updates

* Ensure build agents restore Reqnroll packages and run the same test runners (dotnet test / vstest).
* For MSTest + Scenario Outlines, validate **filtering** and **distributed execution**; toggle `allowRowTests` if needed. ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))
* If LivingDoc reports are required, implement the current workaround process referenced above. ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))

**Acceptance:** Green pipeline across branches; reports generated as before (or an agreed replacement).

---

## 9 - Rollout Plan

1. **Pilot** one solution using **Compatibility Package path** (fast validation).
2. If clean, choose **Full Namespace path** to remove shims and future-proof.
3. Roll to remaining solutions; keep a flag to **fall back** to compatibility package if any blockers surface.

---

## 10 - Automated Transform Steps (suggested implementation)

> Pseudocode for the AI tool; apply per `*.csproj` in the solution.

1. **Swap NuGets**

  * Remove packages: `SpecFlow*`, `CucumberExpressions.SpecFlow.*`.
  * Add `Reqnroll.{MsTest|NUnit|xUnit}` (match current framework).
  * If path = Compatibility: add `Reqnroll.SpecFlowCompatibility`; also add `Reqnroll.SpecFlowCompatibility.Actions.*` for each `SpecFlow.Actions.*` found. ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))

2. **Namespace & symbol refactors** (Full Namespace path):

  * Replace `TechTalk.SpecFlow` → `Reqnroll`.
  * Replace `using BoDi;` → `using Reqnroll.BoDi;`.
  * Replace `ISpecFlowOutputHelper` → `IReqnrollOutputHelper`. ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))

3. **Config**

   * If `specflow.json` exists:

    * Option A (quick): keep as-is (Reqnroll-compatible).
    * Option B (full): rename to `reqnroll.json`, add `$schema`, rename keys (`stepAssemblies` → `bindingAssemblies`, `bindingCulture/name` → `language/binding`). ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))
   * If `App.config` contains SpecFlow section: map to `reqnroll.json`.

4. **Cucumber Expressions hardening (if regex-first project)**

  * Bulk add `^...$` to `[Given|When|Then]("...")` attributes using the documented find/replace. ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))
  * Or set project policy to generate regex skeletons via:

    ```json
    {
      "trace": { "stepDefinitionSkeletonStyle": "RegexAttribute" }
    }
    ```

5. **MSTest Scenario Outline choice** (if MSTest):

  * If CI tooling incompatible with row tests, set:

    ```json
    { "generator": { "allowRowTests": false } }
    ```

6. **Build & Test**

   * `dotnet restore && dotnet build && dotnet test -v n`.

7. **Report & Gate**

   * Emit a summary (changed packages, files touched, config diffs, failing specs, suggested next actions).

---

## 11 - Acceptance Checklist (per project)

* [ ] Swapped to `Reqnroll.*` test package; **no** `SpecFlow.*` packages remain. ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))
* [ ] Builds succeed; `.feature` files generate code-behind via Reqnroll MSBuild generator. ([docs.reqnroll.net PDF](https://docs.reqnroll.net/_/downloads/en/latest/pdf/))
* [ ] Tests run & pass locally; **Scenario Outlines** run as desired (row tests on/off per policy). ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))
* [ ] Config validated (`reqnroll.json` or compatible `specflow.json`); schema added if renamed. ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))
* [ ] Namespace & symbol refactors complete (if Full path): `TechTalk.SpecFlow` → `Reqnroll`, `BoDi` → `Reqnroll.BoDi`, `ISpecFlowOutputHelper` → `IReqnrollOutputHelper`. ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))
* [ ] Any Cucumber Expression mis-detections resolved (markers or expression fixes). ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))
* [ ] CI green; filtering/distribution verified; LivingDoc strategy agreed/worked around. ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))

---

## 12 - Notes & References

* **Official migration guide** (packages, namespaces, config, breaking changes, MSTest Outline behavior, Actions shims, VSIX, LivingDoc note). ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))
* **Guides index** (context & related topics). ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/index.html))
* **Reqnroll docs PDF** (MSBuild generator, packages). ([docs.reqnroll.net PDF](https://docs.reqnroll.net/_/downloads/en/latest/pdf/))
* **Background/why + reassurance of effort**. ([Reqnroll](https://reqnroll.net/news/2024/02/from-specflow-to-reqnroll-why-and-how/))

---

### Appendix A   Example `.csproj` fragment (MSTest + Compatibility path)

```xml
<ItemGroup>
  <PackageReference Include="Microsoft.NET.Test.Sdk" Version="17.8.0" />
  <PackageReference Include="MSTest.TestAdapter" Version="3.2.0" />
  <PackageReference Include="MSTest.TestFramework" Version="3.2.0" />

  <PackageReference Include="Reqnroll.MsTest" Version="2.*" />
  <PackageReference Include="Reqnroll.SpecFlowCompatibility" Version="2.*" />
  <!-- If previously used SpecFlow.Actions.Selenium -->
  <PackageReference Include="Reqnroll.SpecFlowCompatibility.Actions.Selenium" Version="2.*" />
</ItemGroup>
```

(Structure per guide example.) ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))

---

### Appendix B – Bulk "force regex" replace (if needed)

* **Find (regex):** `\[(Given|When|Then)\((@?)"(.*?)"\)\]`
* **Replace:** `[$1($2"^$3$")]`
  Adds `^...$` to all step attributes. ([docs.reqnroll.net](https://docs.reqnroll.net/latest/guides/migrating-from-specflow.html))

---

### Appendix C – `reqnroll.json` schema & options (snippets)

```json
{
  "$schema": "https://schemas.reqnroll.net/reqnroll-config-latest.json",
  "language": { "feature": "en-US" },
  "bindingAssemblies": [{ "assembly": "ExternalStepDefs" }],
  "generator": { "allowRowTests": true },
  "trace": { "stepDefinitionSkeletonStyle": "RegexAttribute" }
}
```
