# Launcher candidate qualification

`release.yml` accepts `launcher-qualification: true` for `storytold/craft-launcher`. Other apps
keep the default `false`, so this adds no builds, runner usage or new gate to their releases.

The opt-in reusable job depends on the existing macOS, Windows and Linux packaging jobs. It
downloads their immutable artifacts from the same run and executes the launcher's
`packaging/e2e/runner.py` on seven native architecture targets. It does not rebuild binaries or
receive signing secrets. Linux gets a virtual desktop and real package tools. The Windows ARM
job runs on Windows ARM, while x86 runs as a 32-bit process on the x64 image.

Draft creation requires this job's explicit `success`; skipped, cancelled or failed qualification
cannot pass. Before creating, editing, deleting or uploading any GitHub release asset,
`actions/draft-release` downloads the reports and runs the exact source commit's
`packaging/e2e/verify_release.py`. That validator checks the required platform/operation set,
source SHA, workflow run, version, cleanup/restart evidence and artifact hashes. The generated
`launcher-qualification.json` joins the candidate's release assets/checksums.

The companion `launcher-qualification-contract.yml` is a cheap PR gate for the launcher's
Python evidence-contract tests. The ordinary launcher Rust jobs already compile/test its native
qualification module, so no extra full PR matrix is added. Full qualification runs only when
the existing candidate/release workflow is invoked. No scheduled workflow is added.

Land the shared opt-in interfaces before enabling them in the launcher caller. Keep the launcher
integration branch separate from its verifier hotfix. Validation of these local changes does not
mean hosted candidate execution has passed. `tests/test_launcher_qualification.py` checks the
workflow contract; the launcher README and generated evidence enumerate remaining UI,
format/architecture, self-update-discovery and disabled-Gatekeeper coverage boundaries.

This gates this workflow's draft creation; it does not alter repository protections or prevent
a maintainer manually publishing a release outside the workflow.
