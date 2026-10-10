# EPIC 05.1 — Validation API Candidate Audit

## Status
**AUDIT COMPLETE — DO NOT EXTRACT THE GENERIC VALIDATOR API YET.** Source searched on the post-ECS migration baseline `d3b81484f42d084cae150b5b60fc64185cfb70c2`; the Logging slice in PR #20 is independently implemented, Unity-tested and merged to `dev` as `ec967a2944ba83db69655d4415d416f49a2ce122`.

## Source evidence
- `Assets/_MythHunter/Code/Utils/Validation/IValidator.cs` declares `IValidator<T>.Validate(T)`, returning `ValidationResult`.
- `Assets/_MythHunter/Code/Utils/Validation/Validator.cs` implements `Validator<T>`, a rule-builder based on `Func<T, ValidationResult>` and `Func<T, bool>`; it aggregates errors and returns immediately on a critical rule result.
- `ValidationResult` is declared in the same `Validator.cs` file. Its public surface is `IsValid`, `IsCritical`, mutable `List<string> Errors`, and static `Success`, `Error`, `Critical` factories.
- Repository code search for `IValidator` finds the interface declaration and `DIValidator.cs`; the latter validates scene objects for DI wiring and does not use the generic `IValidator<T>` contract.
- Search for `new Validator`, `Validator<`, `ValidationResult.Error` and `ValidationResult.Critical` found only the generic validator implementation itself, not production construction/call sites elsewhere.
- Other observed validation APIs are separate Game Layer APIs: `DependencyValidator.ValidateAllDependencies()` returns its own `List<ValidationIssue>`; `StaticData.Validate()` and `PreloadSceneConfig.ValidateConfig()` have different semantics. They are not callers proving demand for `IValidator<T>`.

## Decision
Do not extract `IValidator<T>` or create a Framework Validation assembly in this iteration. Current evidence does not demonstrate real cross-module or even external production use of the generic builder. Moving only the interface would also leave its coupled `ValidationResult` contract behind, and copying the mutable result/critical short-circuit semantics into Framework would institutionalize an unvalidated API without consumers.

Keep the existing files untouched for compatibility. If a second genuine module later needs generic validation, reopen this decision with concrete caller requirements and design a cohesive contract/result API (error representation, immutability, critical/fatal semantics, rule execution order, exception behavior) with tests and an adapter plan before source migration.

## Next roadmap item
Proceed to **EPIC 05.2 — Dependency Injection audit/design**. First map concrete consumers and lifetimes for `IDIContainer`, `IDIInstaller`, scopes, lazy dependencies, lifecycle/disposal, and resolution failures. Do not extract DI wholesale or mix logger cleanup into the DI task.
