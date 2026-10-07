# BreakingBuild

Hands-on labs for C++ build and release engineering: **source → build → test → package → release**,
in isolated and air-gapped environments.

Each lab starts from something **broken** — a build that rebuilds too much, a binary that works in CI
and dies on the device, a pipeline that can't run offline. The job is to diagnose it, fix it, and
prove the fix with `make test`.

## Contents

Reading refers to chapters of [*Software Engineering at Google*](#reading).

- **1xxx Make** — how incremental builds work · _Reading: ch. 18_
  - `1010-MakeIncrementalBuilds` — editing a header doesn't trigger a rebuild; rules, prerequisites, `-MMD -MP` depfiles
  - `1020-MakeParallelRaces` — passes with `-j1`, fails randomly with `-j8`: a missing prerequisite
- **2xxx CMake + Ninja** — the standard C++ build · _Reading: ch. 18_
  - `2010-CMakeTargetsAndPresets` — include paths leak through a missing `PRIVATE`; "works on my machine" is a flag difference
  - `2020-NinjaAndCompilerCache` — unnecessary rebuilds (`ninja -t explain`); low ccache hit rate from absolute paths
  - `2030-CMakeInstallAndExport` — `find_package(Vision)` fails from the install tree; exported config, version file
- **3xxx Dependencies** — Conan, FetchContent, cross-compiling, toolchains, Python wheels · _Reading: ch. 21, 22, 15_
  - `3010-FetchContentPitfalls` — an unpinned `GIT_TAG` breaks the build overnight
  - `3020-ConanLockfiles` — two machines resolve different revisions; a diamond dependency with conflicting versions; profiles, CMakeDeps, lockfiles, overrides
  - `3030-CrossCompileArm64` — build/host profiles mixed up; `GLIBCXX_3.4.32 not found` on the target; toolchain files, sysroots, ABI
  - `3040-PythonWheels` — pybind11 wheel with the wrong platform tags; unpinned tooling; scikit-build-core, manylinux, hash-pinned locks
  - `3050-ToolchainUpgrade` — GCC 12 → 14: new warnings break `-Werror`, ABI changes; roll it out across the codebase and retire the old toolchain image
- **4xxx Containers** — edge images · _Reading: ch. 25 (optional)_
  - `4010-EdgeImage` — a 1.5 GB image with a missing `.so` at run time; multi-stage, multi-arch, digests, SBOM, signing
- **5xxx Artifact management** — Artifactory · _Reading: ch. 24, 15_
  - `5010-ArtifactoryRepoLayout` — generic, Conan, Docker and PyPI repos; local vs remote vs virtual
  - `5020-VersioningAndPromotion` — a non-unique version gets overwritten; build info, dev → staging → release without rebuilding
  - `5030-RetentionAndCleanup` — storage is full; a cleanup policy that never deletes releases
- **6xxx CI/CD** — branching, pipelines, runners, tests, gates · _Reading: ch. 16, 23, 24, 11, 14, 20_
  - `6005-BranchingAndReleaseTrains` — a hotfix lands on trunk but not the release; release branches, protected tags, cherry-picks; shallow clones, LFS vs Artifactory for large data
  - `6010-GitHubActionsMatrix` — a compiler × build-type matrix whose cache key is wrong
  - `6020-GitLabSelfHostedRunner` — leftover runner state leaks between jobs
  - `6030-JenkinsPipeline` — declarative pipeline, agents, stash/unstash, shared library
  - `6040-TestSizesAndFlakes` — a timing-dependent test: detect, quarantine, fix; small/medium/large tests, hermetic tests
  - `6050-PresubmitGates` — broken code merges because nothing blocks it; clang-tidy, sanitizers and format checks as required checks
- **7xxx Hermetic and reproducible builds** — Bazel, Nix · _Reading: ch. 18_
  - `7010-ReproducibleBinaries` — `__DATE__`, build paths and archive order make builds differ; `SOURCE_DATE_EPOCH`, diffoscope
  - `7020-BazelHermeticToolchain` — the build quietly uses the host's gcc; BUILD files, visibility, registered toolchains
  - `7030-BazelRemoteCache` — cache misses from non-hermetic actions
  - `7040-NixFlake` — pin the whole toolchain with `flake.lock`; package the library as a derivation
- **8xxx Linux and debugging**
  - `8010-LinkerAndLoader` — works in CI, fails on the device; rpath, `ldd`, `readelf`, `LD_LIBRARY_PATH`
  - `8020-FieldCrashDebugging` — a crash from the field; split debug info, symbol store, core dumps
- **9xxx Air-gap and release** — capstone · _Reading: ch. 24_
  - `9010-AirgapMirror` — collect every input (Git repos with submodules and LFS, images, Conan, PyPI, apt, toolchains) into a signed bundle
  - `9020-AirgapBuild` — build with `--network=none`; use `strace` to find every hidden network call
  - `9030-EdgeDelivery` — offline update bundle for the device, with verification and rollback
  - `9090-EndToEndRelease` — source → build → test → package → promote → deliver

## Layout

```
BreakingBuild/
├── README.md
├── Makefile            # help, test (runs every lab's test), env-shell, infra-up/down
├── env/                # Linux toolbox image the labs run in
├── infra/              # shared services for 5xxx+: Artifactory JCR, registry, Gitea + runner, Jenkins
└── NNNN-LabName/
    ├── README.md       # the task
    ├── BREAK.md        # what's broken: symptoms, how to diagnose
    ├── GOTCHAS.md      # things learned along the way
    ├── Makefile        # run / test / clean
    └── ...             # the lab's starting state
```

Every lab is self-contained and uses the same `Makefile` interface, whatever tool it studies:

```sh
make run     # do the thing: cmake, bazel, nix build, docker build, ...
make test    # verify the goal: output, hash equality, offline success, promotion state
make clean
```

## Getting started

The labs run on Linux. On Windows, use the `env/` container (or WSL2):

```sh
make env-shell          # drop into the toolbox container with the repo mounted
make -C 1010-MakeIncrementalBuilds test
```

Labs from 5xxx onwards need the shared services:

```sh
make infra-up
make infra-down
```

## Reading

Titus Winters, Tom Manshreck, Hyrum Wright, *Software Engineering at Google* (O'Reilly, 2020).
Chapters relevant to build and release engineering:

- **ch. 1** What Is Software Engineering? — Hyrum's Law; the distributed builds example
- **ch. 11** Testing Overview, **ch. 14** Larger Testing — test sizes, flakiness, deployment configuration testing
- **ch. 15** Deprecation — retiring toolchains, artifacts and versions
- **ch. 16** Version Control and Branch Management — source of truth, release branches, the one-version rule
- **ch. 18** Build Systems and Build Philosophy — task-based vs artifact-based builds, 1:1:1 rule, visibility, distributed builds
- **ch. 20** Static Analysis — presubmits, compiler integration
- **ch. 21** Dependency Management — diamond dependencies, semver and its limits, minimum version selection
- **ch. 22** Large-Scale Changes — changing every caller at once (e.g. a toolchain upgrade)
- **ch. 23** Continuous Integration — fast feedback, presubmit vs post-submit, hermetic testing
- **ch. 24** Continuous Delivery — release trains, flag-guarded features, "no binary is perfect"
- **ch. 25** Compute as a Service — containers and scheduling (optional)
