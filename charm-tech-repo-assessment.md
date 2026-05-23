<!--
  ~ Copyright 2026 Canonical Ltd.
  ~ See LICENSE file for licensing details.
-->

# Charm Tech assessment of `copilot-collections`

## Executive summary

`copilot-collections` is a useful repository for the Charm Tech team **as both a consumer and a contributor**, and the team should treat it as the canonical home for shared Copilot/agent assets going forward.

- **As a consumer:** Charm Tech already publishes four collections from `groups/charm-tech/`, but is currently under-subscribing relative to what is available. Several core skills — particularly the documentation review pipeline (`documentation-review`, `documentation-verify`, `documentation-style`, `documentation-structure`, `documentation-diataxis`, `documentation-build`), the `migrate-harness-tests-to-state-transition-test` skill, and the `copilot-toolkit` meta-skills — directly address Charm Tech's day-to-day work on `ops`, `ops-scenario`, charm libraries, and the operator documentation. Adopting them is low-risk and high-value.
- **As a contributor:** Charm Tech owns a large body of domain knowledge that is not yet captured anywhere in this repo: charmcraft, Pebble, Jubilant, Concierge, integration testing, Scenario authoring (the inverse of the existing migration skill), charm-library conventions, and operator-repo-specific style. Several of these are obvious skill or instruction candidates and would benefit Platform Engineering, Starcraft, and Telemetry too.

**Recommendation:** **Stay in `copilot-collections` and invest in it.** Specifically:

1. Expand `charm-tech-docs` to depend on the `common-documentation` skill bundle (it already does — verify consumers actually pull skills) and add the `migrate-harness-tests-to-state-transition-test` skill to `charm-tech-python-library`.
2. Add a new `charm-tech-toolkit` collection wrapping `copilot-toolkit` so Charm Tech contributors creating new agents/skills/instructions use the meta-tools consistently.
3. Contribute Charm Tech-owned skills back to this repo: `write-state-transition-test`, `charm-library-author`, `pebble-workload`, `juju-integration-test` (Jubilant), and a `charmcraft-yaml` path-instruction set. See [Contribution opportunities](#contribution-opportunities).
4. Treat `canonical/skills-playground` (if/when it exists) as an experimental staging area at most; this repo's `collections.yaml` + sync workflow is the right delivery mechanism for production assets — see [skills-playground comparison](#skills-playground-comparison).

---

## Charm Tech's current footprint

The team's manifest lives at `groups/charm-tech/collections.yaml` and defines four collections. Inheritance is shown in parentheses.

| Collection | Includes | Direct items | Notes |
|---|---|---|---|
| `charm-tech-core` | `core`, `common-go` | – | Brings the two `core` instructions (`instructions.instructions.md`, `core-directive.instructions.md`) and both Go assets (`go-reviewer.agent.md`, `go-style-guide.md`). Implicitly assumes every Charm Tech repo is or could be Go-touching — accurate, because of Pebble, Concierge, and `ops-tracing`. |
| `charm-tech-docs` | `charm-tech-core`, `common-documentation` | `assets/instructions/documentation/documentation-rtd.instructions.md` | Pulls the full doc skill suite (`documentation-style/structure/diataxis/verify/build/review`) from `common-documentation` plus the generic `documentation.instructions.md`. Adds the RTD-specific path instruction on top. |
| `charm-tech-python-library` | `charm-tech-core`, `common-python` | `groups/charm-tech/instructions/python-library.instructions.md` | Adds the Charm-Tech-specific python-library instructions (Ops/Pebble/Scenario context, STYLE.md pointer, Google docstrings, Diátaxis pointers, conventional commits). `common-python` currently only adds `code-commenting.instructions.md`. |
| `charm-tech-make` | `charm-tech-core` | `groups/charm-tech/agents/doc-agent.md`, `groups/charm-tech/agents/lint-agent.md` | Two persona-style agents wired to a `make`-based workflow (`make html`, `make lint`, `make fix`, `make format`, `make spelling`, …). The `dest` paths target `.github/instructions/agents/`, which is **inconsistent with the convention** used elsewhere (`.github/agents/…`) — see [Risks & caveats](#risks--caveats). |

**Charm Tech-owned assets (full list):**

- `groups/charm-tech/agents/doc-agent.md` — Diátaxis-aware technical writer persona; assumes `docs/` Makefile targets.
- `groups/charm-tech/agents/lint-agent.md` — Ruff/Pyright/`make lint`/`make fix` persona.
- `groups/charm-tech/instructions/python-library.instructions.md` — Repository-wide instructions covering `ops`, `ops-scenario`, `ops-tracing`, Pebble, Jubilant, Concierge; sets backwards-compatibility expectations, import style (`import ops`, not `from ops import …`), Google-style docstrings, Diátaxis pointers, and conventional commit rules.

What Charm Tech inherits from shared collections, for reference:

- From `core`: `assets/instructions/common/instructions.instructions.md` (meta-guidelines for writing instruction files), `assets/instructions/common/core-directive.instructions.md` (the "primacy of user directives / surgical edits" base prompt).
- From `common-go`: `assets/agents/go-reviewer.agent.md` (a 9-stage code-review agent with a mandatory documentation-verification pass) plus the substantial `assets/agents/go-style-guide.md`.
- From `common-python`: just `assets/instructions/python/code-commenting.instructions.md` — useful but narrow.
- From `common-documentation` (only via `charm-tech-docs`): `assets/instructions/documentation/documentation.instructions.md` plus the six documentation skills.

---

## Recommended assets to adopt

These are concrete assets already present in `copilot-collections` that Charm Tech does not currently pull in but should.

| Asset | Type | Where it lives | Why it fits Charm Tech |
|---|---|---|---|
| `migrate-harness-tests-to-state-transition-test` | Skill | `skills/migrate-harness-tests-to-state-transition-test/` | Charm Tech owns Harness deprecation in `ops.testing`. The skill is literally about migrating away from a `canonical/operator`-shipped API. Add to `charm-tech-python-library` items. |
| `retrospective-artifacts` | Skill | `skills/retrospective-artifacts/` | Captures incident/troubleshooting retros under `.retrospectives/` with GitHub/Jira/Mattermost integration. Useful for `ops` regressions, Pebble incidents, Juju compatibility breaks. Add to `charm-tech-core` or as opt-in via a `charm-tech-retro` collection. |
| `generate-agent` / `generate-agent-skills` / `generate-path-instructions` / `generate-prompt` / `generate-repo-instructions` | Skills | `skills/generate-*` | Charm Tech is positioned to *write* a lot of new assets (see [Contribution opportunities](#contribution-opportunities)). Surfacing the meta-tools via a `charm-tech-toolkit` collection keeps contributors using the scaffolding and validation scripts instead of free-handing markdown. |
| `copilot-asset-architect.agent.md` | Agent | `assets/agents/copilot-asset-architect.agent.md` | Persona that routes asset-creation requests to the right generator skill. Pairs with the toolkit collection above. |
| `documentation-review` and its sub-skills | Skills | `skills/documentation-*` | Charm Tech *does* pull these via `charm-tech-docs`, but only docs-focused repos consume `charm-tech-docs`. Consider including (or cross-listing) `documentation-style` and `documentation-verify` in `charm-tech-python-library` too — `ops` ships substantial Sphinx docs from the same repo as the library. |
| `documentation-not-rtd.instructions.md` | Instruction | `assets/instructions/documentation/documentation-not-rtd.instructions.md` | For Charm Tech repos that ship Markdown docs without RTD (e.g. some library READMEs, Concierge). Currently used by Platform Engineering. |
| `documentation-release-notes.instructions.md` | Instruction | `assets/instructions/documentation/documentation-release-notes.instructions.md` | The `canonical/release-notes-automation` workflow is used (or planned) across the operator ecosystem. Add to `charm-tech-docs` for any repo that produces release-notes artifacts. |
| `go-reviewer.agent.md` + `go-style-guide.md` | Agent + reference | `assets/agents/` | Already inherited via `common-go`, but Charm Tech should *verify* that Pebble, Concierge, and `ops-tracing` repos are actually subscribed to `charm-tech-core` (they may pre-date the collection). |

**Suggested manifest edits** (illustrative — do not apply as part of this assessment):

```yaml
# groups/charm-tech/collections.yaml

charm-tech-python-library:
  includes:
    - charm-tech-core
    - common-python
  items:
    - src: instructions/python-library.instructions.md
      dest: .github/instructions/python-library.instructions.md
    - src: /skills/migrate-harness-tests-to-state-transition-test
      dest: .github/skills/migrate-harness-tests-to-state-transition-test/

charm-tech-toolkit:
  description: "Charm Tech contributor toolkit (asset generators)"
  includes:
    - charm-tech-core
    - copilot-toolkit
```

---

## Contribution opportunities

These are concrete additions Charm Tech could make to `copilot-collections`. Each one fills a real gap in the current asset set and is genuinely team-owned knowledge — not derivable from the shared `common-*` collections.

1. **`write-state-transition-test` skill.** The mirror image of `migrate-harness-tests-to-state-transition-test`. Greenfield charm tests are not migrations; they need guidance on shaping `testing.State`, choosing event boundaries, using `ctx.run(ctx.on.<event>(...))`, and asserting on `state_out`. Reuse the recipes already under `skills/migrate-harness-tests-to-state-transition-test/references/state-transition-recipes.md`.

2. **`charm-library-author` skill.** Conventions for files in `lib/charms/<owner>/<version>/<lib>.py` from the *publisher* side: `LIBID`, `LIBAPI`, `LIBPATCH` semantics, backwards-compatibility rules, deprecation patterns. Complements Platform Engineering's `lib-updates.instructions.md` at `groups/platform-engineering/instructions/lib-updates.instructions.md`, which handles the *consumer* side.

3. **`pebble-workload` path-instructions** (`applyTo: '**/src/**/*.py'` or similar). Idiomatic `ops.Container` usage: `push`, `pull`, `add_layer`, `replan`, notice handling, file ownership/permissions, working with custom services. Pebble-specific patterns are notoriously easy to get wrong and Charm Tech maintains the canonical examples.

4. **`juju-integration-test` (Jubilant) skill.** How to structure `tests/integration/` with Jubilant: model lifecycle, deploy/relate/wait patterns, multi-controller tests, secret handling, debug bundles. Jubilant is Charm Tech-owned and there is currently no integration-test guidance anywhere in the repo.

5. **`charmcraft-yaml` path-instructions** (`applyTo: 'charmcraft.yaml'`). Schema-aware rules: `bases` vs `platforms`, parts, resources, config option types, action declarations, container declarations for Kubernetes charms. Today every team writing a charm has to figure this out independently.

6. **`ops-scenario-author` agent.** A persistent persona for writing Scenario tests against a charm, distinct from the migration skill (which is a one-shot workflow). Could pair with `write-state-transition-test`.

7. **`charm-tech-style.instructions.md`.** Today `groups/charm-tech/instructions/python-library.instructions.md` points users at `https://github.com/canonical/operator/blob/main/STYLE.md` over HTTP. Mirror or extract the operative rules into a path-instruction file so Copilot doesn't have to fetch a remote URL.

8. **`concierge-config` instruction** (lower priority). For any repo writing `concierge.yaml`-style configs.

9. **Hook a Charm Tech reviewer agent** analogous to `go-reviewer.agent.md` but for Python charms: focus on the import rules from `python-library.instructions.md`, type-hint completeness (`pyright`), Scenario test coverage of new events, and docstring conformance to Sphinx. Could share the 9-stage workflow scaffolding.

10. **Bring the `make`-based agents (`doc-agent`, `lint-agent`) up to the asset-spec.** They are written as system prompts but live under `groups/charm-tech/agents/` and are placed at `.github/instructions/agents/…` rather than `.github/agents/…`. Either convert them to proper `*.agent.md` frontmatter and fix the `dest`, or split each into a path-instruction file + a smaller agent persona.

---

## skills-playground comparison

**Status:** I could not access `https://github.com/canonical/skills-playground`. WebFetch returns HTTP 404 (consistent with the repo being private, deleted, or never existing under that exact name), the GitHub MCP scope in this environment is restricted to `tonyandrewmeyer/copilot-collections` so I could not query the GitHub API for it, and a web search did not surface a `canonical/skills-playground` repository. The closest public Canonical match is `canonical/open-documentation-academy`, which is unrelated.

**Reasoned comparison.** Treating the name at face value:

| Dimension | `copilot-collections` (this repo) | `skills-playground` (assumed, by name) |
|---|---|---|
| Intent | Production distribution toolkit. Versioned via Git tags, consumed by `.github/.copilot-collections.yaml` + `local_sync.sh` + a reusable GitHub Actions workflow (`auto_update_collections.yaml`). | Experimentation / prototyping space, by convention of the "-playground" suffix. |
| Coupling to consumers | Tight: 50+ repos can subscribe and auto-update. Breaking the `collections.yaml` schema breaks downstream syncs. | Loose: no contract with consumers; assets are typically copied by hand or used as references. |
| Lifecycle | Reviewed PRs, CODEOWNERS, markdown linting (`make lint-md`, `.pymarkdown.json`), tagged releases. | Lower bar; useful for trying ideas before they're standardised. |
| Right home for Charm Tech assets | Yes for anything stable and team-wide. | Maybe for one-off experiments that aren't ready to commit to the `collections.yaml` schema. |

**Recommendation.** Until evidence of a real `canonical/skills-playground` surfaces, Charm Tech should treat `copilot-collections` as the single source of truth and skip a playground hop. If a playground repo exists internally, the right pattern is: prototype there, then promote to `copilot-collections` once the asset has frontmatter, a destination path, and at least one consumer ready to subscribe. Anything currently living only in a playground should be enumerated and graduated; nothing in `copilot-collections` should be downgraded into a playground.

---

## Risks & caveats

1. **Sync mechanism is the single point of failure.** Consumers depend on `scripts/local_sync.sh` and `.github/workflows/auto_update_collections.yaml`. A breaking change to `collections.yaml` schema (e.g. nested `includes`, the `src: /assets/...` root-path convention) silently breaks every subscriber. Pin a `version:` in `.github/.copilot-collections.yaml` for Charm Tech-owned repos.

2. **Naming collisions.** `README.md` warns: collection names are global; prefix with the group name. Charm Tech follows this (`charm-tech-*`). If new collections are added (e.g. `charm-tech-toolkit`), keep the prefix discipline. Note that asset *file* names (e.g. `documentation-rtd.instructions.md`) are not group-prefixed, and `dest` paths can collide if two groups both pull the same file under different `dest`s. The current `groups/charm-tech/collections.yaml` `charm-tech-make` entries write to `.github/instructions/agents/` which is **inconsistent with the convention** (`.github/agents/`) — this is a latent bug worth fixing in a separate PR.

3. **Maintenance burden.** Each new skill/agent contributed here becomes a long-term ownership commitment. Charm Tech should appoint a CODEOWNER for `groups/charm-tech/**` and for any core skills it adopts (e.g. `skills/migrate-harness-tests-to-state-transition-test`). Currently `CODEOWNERS` exists at the repo root but Charm Tech should verify it covers their paths.

4. **Version pinning vs. drift.** The auto-update workflow runs Mondays at 09:00 UTC by default. Repos that pin `version: v1.0.0` will not pick up new Charm Tech changes; repos that pin `@main` will pick them up immediately, including regressions. Recommend pinning to release tags and cutting releases on a known cadence after Charm Tech changes land.

5. **`yq` snap vs. apt dependency.** `README.md` explicitly warns that `local_sync.sh` requires the snap version of `yq`. Self-hosted CI runners not provisioning snap will fail silently or with confusing errors. This affects any Charm Tech repo that uses non-standard runners (e.g. some Pebble CI lanes).

6. **Path-resolution gotcha.** Per `README.md`: `src: instructions/x.md` resolves relative to `groups/<team>/`, but `src: /assets/...` resolves from the repo root. `groups/charm-tech/collections.yaml` already mixes both styles. New contributions must follow the convention or the sync will produce empty files.

7. **Skills are directories, not files.** The README calls this out: skill `dest` paths must end with `/`. Easy to get wrong in PR review — worth wiring into `scripts/validate_collections.sh` if not already.

8. **Asset frontmatter drift.** The `copilot-asset-architect.agent.md` agent enforces strict frontmatter rules and mandates using the generator skills. Manually authored Charm Tech assets (the two `agents/*.md` files) don't follow the architect's spec exactly. If Charm Tech adopts the `copilot-toolkit`, regenerate these via `generate-agent` and replace the hand-rolled versions.

9. **External URLs in assets.** `python-library.instructions.md` links to `STYLE.md` on `canonical/operator@main`. That target can move or be renamed without breaking this repo's lint. Prefer vendoring style content (Contribution opportunity #7).

10. **`canonical/skills-playground` uncertainty.** As noted, the playground repo was not reachable in this environment. Before treating it as obsolete, a maintainer with broader access should confirm whether it exists and, if so, audit it for assets that should be promoted here.
