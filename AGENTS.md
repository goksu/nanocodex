# Nanocodex development

- Define observable behavior and failure cases before implementation. Validate
  changes with black-box, end-to-end journeys that a real user could perform:
  invoke the shipped CLI, call the public API over its actual transport, or use
  both when both surfaces matter. Run the real executable/runtime and assert on
  user-visible results, including representative errors, authorization, and
  recovery paths. Use synthetic accounts/data and safe test environments; stub
  only unavoidable external dependencies, not the behavior under test.
- Skip low-level unit tests as the default for both new coverage and routine
  validation. Do not add helper-by-helper or mock-heavy tests to stand in for a
  user journey. Prefer a small set of representative E2E scenarios over a large
  matrix of incidental configurations. If a critical failure truly cannot be
  observed at a public boundary, document that gap and use the narrowest
  realistic integration check rather than silently substituting unit coverage.
- Finish each E2E run with reproducible evidence: the command, inputs, expected
  and observed outcomes, and an inspectable trace, transcript, log, screenshot,
  or recording where relevant. A passing test name alone is not evidence.
- Continuously look for ways to improve the developer experience in CI. Inspect
  the relevant jobs' actual results, duration, failures, and artifacts; favor
  fast, reliable user-journey feedback, actionable failure output, and easy
  reproduction locally. Remove redundant work and flaky setup, but do not hide
  failures, skip required checks, or trade away meaningful E2E coverage merely
  to make CI green. When changing CI, verify the resulting workflow run.
- When a journey or protocol check covers the same failure, remove redundant
  lower-level cases, unused fixtures, test-only APIs, and obsolete runner
  references. Prune mock setup that no surviving scenario uses. Use compiler,
  lint, and package checks for static contracts; do not test source text,
  private layouts, method presence, fixed prompt/UI copy, or a mock's own
  behavior as a proxy for runtime behavior.

- Keep documentation focused on current APIs, architecture, setup, and operations.
  Remove superseded designs, implementation plans, checklists, and review notes
  when the work lands; Git retains their history. Update links and consumers
  when removing documents or support files.
- Publish per-run screenshots, videos, logs, traces, and benchmark results as CI
  artifacts or keep them in ignored `output/`. Do not commit generated evidence;
  retain only intentional fixtures consumed by tests or current documentation.
- Keep only the static assets and fixture files their consumers need. Check
  dynamic filename construction and build manifests before pruning imported
  asset packs; preserve attribution and canonical source artwork.

- `macos/` owns the desktop app and native tiled workspace; `js/desktop-runtime`
  owns its runtime. `apple/NanocodexInbox` targets iPhone and iPad.
- `js/nanocodex` and `js/nanocodex-react` are public contracts. Cover changes
  with relevant contract, type, package, and runtime checks.
- `js/nanocodex-vite` owns the Vite plugin, WASM build, OAuth relay, and
  Cloudflare Vite integration.
- Apps and Workers deploy independently. Shared behavior belongs in a package,
  consumed through its public API.
- Use root `pnpm` scripts and existing Turbo/Portless/Vite/Wrangler tooling.
  Deploy dependencies first and `account` last. Component deploy scripts build
  their dependencies from a clean checkout. See [README.md](README.md) for setup.
- Use synthetic identities and project data in fixtures and examples. Keep real
  account IDs, private project inventories, and one-off personal migration plans
  outside tracked source; pass operational data through private runtime inputs.

- On a shared macOS Hand, invoke Xcode through `scripts/xcodebuild-guard.sh`
  instead of raw `xcodebuild` for `apple/` and `macos/` work. The per-user OS
  lock queues builds across agent sessions until the build actually exits; the
  default `-jobs 3` leaves CPU for interactive use, and UI tests default to one
  nonparallel Simulator destination. Explicit caller flags override those
  defaults. Do not boot duplicate simulators for concurrent UI tests; shut down
  only the simulators used by your run when finished.

## Fork workflow overrides

- Follow the user's personal isolated-worktree requirements. Create a new
  thread-dedicated detached worktree outside the repository before changing
  files. Keep the source checkout read-only unless the user explicitly opts
  out. Eval-loop overrides do not waive these requirements.
- Keep commits focused, chronological, and independently understandable. Never
  mix unrelated cleanup into an iteration commit.
- Preserve unrelated user work. Never commit `.env`, caches, retained jobs,
  build output, or another user's untracked files.

- Use the local Codex checkout before making architecture or behavior claims
  about Codex. Do not browse or invoke OpenAI documentation tooling unless the
  user explicitly asks.

## Frontier eval iteration

- Optimize for wall-clock time from an idea to evidence from the real benchmark
  host. Local compilation ceremony, compatibility work, speculative tests, and
  preserving replaceable experimental processes are subordinate to that loop.
- Run benchmarks on `ubuntu@dev-georgios`. The canonical state directory is
  `/mnt/nanocodex-evals/evals` and the canonical ledger is
  `/mnt/nanocodex-evals/evals/state.sqlite3`. Imports, new worksets, resumed
  runs, coordinator/API reads, and the eval dashboard use that ledger. Use
  `--state-dir /mnt/nanocodex-evals/evals` for every benchmark add, run, resume,
  migration, coordinator/API, and UI operation. Add new profiles and attempts
  to that ledger instead of creating per-run or smoke state databases. Use
  another host or state directory only when the user explicitly requests an
  isolated experiment.
- Deploy a coherent slice immediately and exercise it there. Start from fresh
  `origin/master` plus the focused change being tested unless the user names
  another ref. Build that exact source; if GitHub or DNS is unavailable on the
  host, transfer the exact local source instead of waiting or using an old
  deployment.
- Replacement is component-scoped, not preservation-oriented. Controller/UI
  work replaces the controller/UI and leaves workers and coordinator alone;
  coordinator work replaces the coordinator; worker/runtime or schema work may
  stop the controller and all workers for that benchmark before restarting the
  whole scoped run. Never disturb unrelated profiles or services. When the user
  asks to replace a scoped component on the box, replace every running instance
  of that component instead of preserving stale processes.
- Do not run `cargo test`, broad `cargo check`, Clippy, or full-workspace builds
  during the active edit loop. Make the complete focused change, format it, use
  cheap consumer typechecks when useful, then build once for deployment on
  `dev-georgios`. Run a focused Rust test only for a demonstrated regression or
  when the user explicitly asks. Reserve broad validation for an explicit
  milestone, release gate, or final handoff where its signal justifies the
  compile time.
- Never test neural scheduling policy by asserting prompt text. Build, deploy,
  and exercise orchestration changes against the real coordinator and host.
  Record worker/VM correspondence, task deltas, completions per unit time,
  memory, swap, load, pressure, infrastructure retries, and OOMs.
- High utilization is the goal, not a failure. Judge saturation by productive
  throughput, stale claims, infrastructure retries, OOM behavior, and recovery;
  do not label a host unhealthy merely because CPU, RAM, swap, load, or pressure
  is high. During a normal saturation measurement, never manually shed workers:
  the OS and controller own exhaustion behavior. Scoped deployment and schema
  resets are the explicit exception.
- Treat live waves as telemetry, not blocking work. Continue inspecting real
  evidence, fixing known failures, and preparing the next deployment while a
  wave runs. Wait only when a concurrent mutation would invalidate a specific
  measurement needed for the next decision.
- Treat obsolete services, systemd drop-ins, scratch directories, deployments,
  and other stale host residue as operator cleanup. Inspect their exact scope
  and remove them directly on `dev-georgios`; do not infer a product feature,
  compatibility path, migration, or automatic cleanup requirement merely
  because old operational state exists.
- Use `just run` for a live native smoke, focused trials while iterating, and the
  full configured eval only for milestone or release gates. Never modify a
  benchmark task or verifier to make Nanocodex pass. Inspect exact JSONL,
  trajectories, verifier output, and retained evidence for concrete claims.

## Experimental eval state

- Eval ledgers, coordinator state, retained artifacts, and their schemas are
  mutable development state, not compatibility boundaries.
- Keep SQLite `user_version = 1`; it is a current-format marker, not migration
  history. On every schema change, stop the scoped run, directly mutate the
  canonical database in place to the one new layout, update the single current
  schema definition, and restart. Preserve completed rows only when the direct
  transformation is useful and obvious; otherwise recreate or reseed them.
- Never add old-schema readers, migration ladders, version-specific branches,
  dual writes, fallback runtimes, or compatibility shims unless the user
  explicitly asks. Never return to an older binary because current code rejects
  experimental state.
- Do not make backups or pause iteration to preserve experimental state unless
  the user explicitly requests one. Once the canonical database is migrated,
  delete obsolete schema and migration code immediately.
