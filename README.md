# upatch-registry

Source packages of [shirudo](https://github.com/shirudo-dev/shirudo) upatches: small signed BPF programs that block or detect a vulnerable path, in the Linux kernel or in an application, until a machine gets the fix.

Each entry is a package directory. You build it, sign it with your own shirudo key and publish it to your own OCI registry. Your machines then pull it like any other upatch.

## Upatches

| Upatch | Fixes | What it does |
| --- | --- | --- |
| [cve-2026-99350-rxrpc-block](upatches/kernel-cve/cve-2026-99350-rxrpc-block) | CVE-2026-99350 | Refuses new RxRPC sockets |
| [cve-2025-24813-tomcat-block](upatches/app-cve/cve-2025-24813-tomcat-block) | CVE-2025-24813 | Refuses the Apache Tomcat partial PUT session file write and load |

## Layout

```
upatches/
  kernel-cve/          mitigations and detectors for Linux kernel CVEs
  app-cve/             mitigations for CVEs in applications, enforced at kernel hooks
    <name>/
      Beefile          the package: name, license, upatch, tree entry, the CVE it fixes
      <name>.bpf.c     the BPF program
      README.md        the vulnerability, what the upatch blocks, side effects, sources
tools/
  check                checks every entry's names, Beefile and license lines
  compile              compiles every entry with clang 23 and the flags shirudo uses
  build                builds and signs packages with your shirudo key
```

## Use a upatch

You need a shirudo operator host and the signing key your machines trust. Read the entry's README first: a mitigation can switch off a kernel feature on every machine that loads it.

1. Build and sign the package. It lands in `build/packages/<name>-1.0.0.bee`.

    ```
    tools/build <shirudo-checkout> <keys-dir> 1.0.0 upatches/kernel-cve/<name>
    ```

2. Publish it and add it to your upatch tree. Every machine that follows the tree loads it within about a minute.

    ```
    shirudo bpm push build/packages/<name>-1.0.0.bee oci://<registry>/<namespace>/<name> --keys-dir <keys-dir>
    shirudo bpm tree append build/packages/<name>-1.0.0.bee oci://<registry>/<namespace>/<tree> --tree-id <tree-id> --keys-dir <keys-dir>
    ```

shirudo's [publishing guide](https://docs.shirudo.dev/upatches/publish/publishing.html) covers the details.

## Check an entry

```
tools/check
tools/compile <shirudo-checkout>
```

`tools/compile` needs clang 23. CI runs both on every pull request.

## Contributing, security and license

- [CONTRIBUTING.md](CONTRIBUTING.md) says how to add a upatch.
- [SECURITY.md](SECURITY.md) says how to report a upatch that fails to protect a machine or harms one.
- Upatch sources are licensed GPL-2.0 OR BSD-2-Clause and everything else BSD-2-Clause. Each file carries its SPDX identifier, and the license texts are in [LICENSES](LICENSES).
