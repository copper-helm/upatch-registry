# upatch-registry

Source packages of [shirudo](https://github.com/shirudo-dev/shirudo) upatches: small signed BPF programs that block or detect a vulnerable path, in the Linux kernel or in an application, until a machine gets the fix.

Each CVE has one package directory, `upatches/<cve-id>/`. You build it, sign it with your own shirudo key and publish it to your own OCI registry. Your machines then pull it like any other upatch.

## Layout

```
upatches/
  <cve-id>/                 one package per CVE, such as cve-2026-99350
    Beefile                 the package: name, license, upatches, tree entries, the CVE they fix
    <cve-id>-<what>.bpf.c   a BPF program, such as cve-2026-99350-rxrpc-block.bpf.c
    README.md               the vulnerability, what the upatch blocks, side effects, sources
    evidence/               evidence.toml: how it was tested and what it showed, plus small logs
payloads/
  <cve-id>/                 the same CVE id
    payload                 a harmless probe of the step the upatch guards
    setup, teardown         install and remove the vulnerable software, when the test needs it
    README.md               what the probe does, what it needs, what each result means
tools/
  check                     checks every entry's names, Beefile, license lines and evidence
  compile                   compiles every entry with clang 23 and the flags shirudo uses
  build                     builds and signs packages with your shirudo key
  test                      runs each payload before, while and after its upatch is loaded
```

## Use a upatch

You need a shirudo operator host and the signing key your machines trust. Read the entry's README first: a mitigation can switch off a kernel feature on every machine that loads it.

1. Build and sign the package. It lands in `build/packages/<cve-id>-1.0.0.bee`.

    ```
    tools/build <shirudo-checkout> <keys-dir> 1.0.0 upatches/<cve-id>
    ```

2. Publish it and add it to your upatch tree. Every machine that follows the tree loads it within about a minute.

    ```
    shirudo bpm push build/packages/<cve-id>-1.0.0.bee oci://<registry>/<namespace>/<cve-id> --keys-dir <keys-dir>
    shirudo bpm tree append build/packages/<cve-id>-1.0.0.bee oci://<registry>/<namespace>/<tree> --tree-id <tree-id> --keys-dir <keys-dir>
    ```

shirudo's [publishing guide](https://docs.shirudo.dev/upatches/publish/publishing.html) covers the details.

## Check an entry

```
tools/check
tools/compile <shirudo-checkout>
```

`tools/compile` needs clang 23. CI runs both on every pull request.

## Test an entry on a machine

Build the package, copy the registry and the package to a shirudo test machine, and run as root:

```
tools/test --setup build/packages/<cve-id>-1.0.0.bee
```

It runs the entry's `setup` when it has one, then its payload, loads the upatch, runs the payload again, unloads the upatch and runs it a third time. The upatch passes when the payload was blocked while it was loaded. Set `SHIRUDO` when the trusted runtime binary isn't `shirudo` on the path, and the payload's own settings as its README says.

## Contributing, security and license

- [CONTRIBUTING.md](CONTRIBUTING.md) says how to add a upatch.
- [SECURITY.md](SECURITY.md) says how to report a upatch that fails to protect a machine or harms one.
- Upatch sources are licensed GPL-2.0 OR BSD-2-Clause and everything else BSD-2-Clause. Each file carries its SPDX identifier, and the license texts are in [LICENSES](LICENSES).
