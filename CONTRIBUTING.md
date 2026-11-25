# Contributing

Each pull request adds or changes one upatch.

## Add a upatch

1. Make the directory `upatches/<cve-id>/`, named by the CVE id in lowercase, such as `upatches/cve-2026-12345/`. One directory holds every upatch for that CVE, whether it is in the Linux kernel or in an application.
2. Name each upatch after the CVE: the CVE id, then a hyphen and what it does, such as `cve-2026-12345-foo-block`, in lowercase letters, digits and hyphens. The source file's base name and the tree id use the upatch's name. The package uses the CVE id.
3. Add these files:
    - `Beefile`: `[package]` with `name = "<cve-id>"`, `description` and `license = "GPL-2.0 OR BSD-2-Clause"`, then for each upatch an `[[upatches]]` with `source = "<upatch>.bpf.c"` and an `[upatches.tree]` with `id = "<upatch>"` and `fixes = ["CVE-YYYY-NNNNN"]`.
    - `<upatch>.bpf.c`: the BPF program.
    - `README.md`: the sections listed below.
    - `evidence/evidence.toml`: how you tested it, described below.
4. Make the directory `payloads/<cve-id>/`, with the same CVE id, and add `payload` and `README.md`, described below.
5. Run `tools/check` and `tools/compile <shirudo-checkout>`.
6. Build the package and run `tools/test` on a shirudo test machine, which runs the payload before, while and after the upatch is loaded. Record the result in `evidence/`.

## Record the evidence

`evidence/evidence.toml` records the test that step 6 asks for. upatch.dev shows it on the CVE's page, and `tools/check` refuses an entry without it.

```toml
schema = 1
summary = "One or two sentences: what the test showed, and any limit of the test."

[environment]
date = 2026-11-24                 # a TOML date
kernel = "7.4-shirudo-ci-20261123"
architecture = "x86_64"           # x86_64, aarch64 or riscv64
target = "What was attacked: the software, its version and the setting that makes it vulnerable"
machine = "EC2 t3.large"          # optional
distribution = "Ubuntu 26.04"     # optional
shirudo = "154afeea"              # optional, the shirudo commit

[matrix]
columns = ["Exploit step blocked", "Normal use works"]   # 1 to 8 checks
rows = [
  { state = "before", values = ["no", "yes"] },
  { state = "loaded", values = ["yes", "yes"] },          # required
  { state = "after", values = ["no", "yes"] },
]

[[media]]                         # optional
kind = "video"                    # video or image
url = "https://..."               # an https URL or a /media/ path on upatch.dev
caption = "What the recording shows."

[[logs]]                          # optional, a .log or .txt file in evidence/, at most 64 KiB
file = "test-matrix.log"
caption = "What the log shows."
```

Only `evidence.toml` and the logs it lists may sit in `evidence/`. Media is linked, never committed. The payload lives in `payloads/<cve-id>/`, and exploit code stays out of the registry.

## Write the program

- Write CO-RE C against `vmlinux.h`. Never use a hard-coded struct offset: it signs fine and the verifier refuses it on another kernel.
- Use only the [attach kinds](https://docs.shirudo.dev/upatches/write/attach-kinds.html) shirudo supports.
- Refuse an operation in an `lsm` program by returning `-EPERM`, never with `fmod_ret`.
- Test the cheapest condition first and return early. A upatch runs on every event of its hook, on every machine that loads it.
- Start the file with `// SPDX-License-Identifier: GPL-2.0 OR BSD-2-Clause` and declare `char _license[] SEC("license") = "Dual BSD/GPL";`.
- Never add exploit code, or anything that triggers the bug.

## Write the payload

A payload is a probe, not an exploit. It takes the first step that the upatch guards, with harmless input, and reports whether it got through. It never corrupts memory, runs code it brought, raises privileges or leaves anything behind.

- `payload` is an executable that starts with a `#!` line, such as `#!/usr/bin/env python3`, and is at most 64 KiB.
- It prints one line and exits with `0` when it reached the guarded step, `10` when it was blocked, and `20` when the machine lacks the vulnerable feature. Any other status is an error.
- It reads its settings, such as a server's address, from environment variables.
- `README.md` names the CVE, says what the payload does and what it needs, and has a `## Results` section with the four results.
- The directory holds only these two files.

`tools/test <cve-id>-<version>.bee` runs as root on a shirudo test machine. For each package it runs the payload, loads the package's upatches, runs it again, unloads them and runs it a third time. It passes when the payload was blocked while the upatches were loaded, and the runs before and after give the same result.

## Write the README

Use these sections:

- **The vulnerability**: one paragraph that names the CVE.
- **What the upatch does**: what it blocks or detects, and at which hook.
- **Side effects**: what stops working while it is loaded.
- **Affected kernels**: the affected versions and the fixed ones.
- **Preemption model and cost**: the model the code is written for and what each run costs.
- **Tested on**: the kernel you loaded it on and how you showed it works.
- **Sources**: the CVE record, the fix commit and the announcement.

## Choose the machines

`[upatches.tree.selector]` limits which machines load the upatch: kernel version, kernel build ID, architecture, kernel config and labels. Leave it out to reach every machine. A machine that can't read its kernel config never matches a `kconfig` rule. The [Beefile reference](https://docs.shirudo.dev/reference/formats/beefile.html) lists the keys.
