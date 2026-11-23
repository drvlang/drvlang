# Working on drvlang

These instructions apply to every drvlang language builder and to their shared tooling. Each repository links here from its root AGENTS.md. Update this file when changing the shared contract, and exercise the change in the affected repositories.

## Build from the native lockfile

- Use dynamic Nix derivations on the public build and test paths. A planner running during the build must produce a derivation from the project's dependency or compilation graph. Consume its output through `builtins.outputOf`. The consumer must be able to evaluate and build with `allow-import-from-derivation = false`. Document the Nix features needed by both client and builder.
- Use floating content-addressed outputs by default for drvlang-generated packages, compiled units, dependency assemblies, checks, and their public forwarding derivations. Keep immutable downloads verified by the native lockfile's hashes. Use stable output names and declare the files and store references a consumer actually reads. For cache publication, use derivation-aware copies that preserve build traces for the supported Nix version.
- Treat the native lockfile and checked-in manifests as the source of dependency versions and integrity data. After a native lockfile update, the same Nix expression and CLI invocation must work without replacing a Nix hash, running a pin generator, or committing a second dependency manifest. Verify downloaded dependencies against the native checksums or immutable source identities. Keep the compiler, package manager, and builder inputs pinned.
- Use the ecosystem's current, fast, reproducible workflow. Choose uv for Python and pnpm for JavaScript. Document the supported native lockfile formats. Prefer Rust for new planners, CLIs, and benchmark harnesses. Go is suitable for tooling that benefits from its native Go APIs.
- Show an explicit source file set in every consumer flake example. Use the checkout as the root of `lib.projectSource` and list the manifests, locks, source, assets, test data, and shared packages that target actually reads. Use `srcForTarget` when programs need different inputs. Keep unrelated commands, documentation, caches, and build products outside their derivations. Verify both package and check derivation paths stay stable after an unrelated edit and change after an input edit.

## Give each language the same entry points

- Provide `drv<language> build`, `test`, and `run`. Accept `-C PROJECT` and an optional positional target. Support output-link selection on builds and pass arguments after `--` to the test runner or built program. Preserve exit status from builds, tests, and programs.
- Provide `lib.mkFlake` and `lib.mkPackage` for a single project, and `lib.mkMonorepoFlake` for a repository with named programs and shared packages. Accept an explicit `commands` name-to-relative-path map or the same map under `commands` in a root `drvlang.json`. A target selected through the CLI and its named flake output must select the same project. Language-native conventions can provide additional discovery.
- Provide `packages`, `checks`, and `apps` for supported systems. Build outputs contain the selected package. Checks run its real tests and fail on a test failure. Apps run the built program with the caller's arguments. Document how to select the executable for a project with several entry points.
- Give a new user a short README explaining when to use the builder, the required Nix settings, and working build, test, run, and monorepo examples. State material platform and lockfile constraints.

## Ground changes in working projects

- Pin a real, publicly available upstream benchmark per language. Identify its revision, the target, and what the benchmark validates. Run the built program or its upstream tests. Use Rust for benchmark orchestration, including when the project being measured uses another language.
- Exercise the public CLI and a downstream flake for a single project and a monorepo with shared code. Verify test failures propagate, target selection works, and a native dependency update needs no extra Nix hash. Check the generated build graph and exercise it with import-from-derivation disabled.
- Check the realized outputs' content addresses. Exercise an input edit that produces identical installed files and a change that affects behavior. Verify downstream reuse for the first case and updated behavior for the second. A changed recipe path alone does not establish that its output changed or its consumers rebuilt.
- Report measurements with their command, workload, trial count, and cache conditions. Keep correctness checks alongside timings.
- Measure CI changes on the consumer path. Exercise a package and its check together, and a repeated job or source edit with dependencies warm. Record which planners, compilers, or tests actually ran. Cacheable build outputs and their required planners should be eligible for substitution with Nix’s normal `always-allow-substitutes = false` setting.
