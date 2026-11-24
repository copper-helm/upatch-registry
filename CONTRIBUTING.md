# Contributing

Each pull request adds or changes one upatch.

## Add a upatch

1. Pick the category. A Linux kernel CVE goes in `upatches/kernel-cve/`.
2. Name the entry: lowercase letters, digits and hyphens, at most 30 characters. A kernel-cve entry starts with its CVE id, such as `cve-2026-12345-foo-block`. The directory, the package, the source file's base name and the tree id all use this name.
3. Add three files:
    - `Beefile`: `[package]` with `name`, `description` and `license = "GPL-2.0 OR BSD-2-Clause"`, one `[[upatches]]` with `source = "<name>.bpf.c"`, and `[upatches.tree]` with `id = "<name>"` and `fixes = ["CVE-YYYY-NNNNN"]`.
    - `<name>.bpf.c`: the BPF program.
    - `README.md`: the sections listed below.
4. Run `tools/check` and `tools/compile <shirudo-checkout>`.
5. Load the upatch on a shirudo test machine, show that it does what its README says, and unload it.

## Write the program

- Write CO-RE C against `vmlinux.h`. Never use a hard-coded struct offset: it signs fine and the verifier refuses it on another kernel.
- Use only the [attach kinds](https://docs.shirudo.dev/upatches/write/attach-kinds.html) shirudo supports.
- Refuse an operation in an `lsm` program by returning `-EPERM`, never with `fmod_ret`.
- Test the cheapest condition first and return early. A upatch runs on every event of its hook, on every machine that loads it.
- Start the file with `// SPDX-License-Identifier: GPL-2.0 OR BSD-2-Clause` and declare `char _license[] SEC("license") = "Dual BSD/GPL";`.
- Never add exploit code, or anything that triggers the bug.

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
