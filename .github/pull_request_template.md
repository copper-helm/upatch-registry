## Upatch

- CVE:
- What it blocks or detects:
- Kernel it was tested on:

## Checklist

- [ ] One upatch in this pull request
- [ ] `tools/check` passes
- [ ] `tools/compile <shirudo-checkout>` passes
- [ ] `payloads/<cve-id>/` holds a harmless probe, not an exploit
- [ ] `setup`, when there is one, pins the vulnerable release, checks its download and listens on 127.0.0.1 only; `teardown` removes it
- [ ] `tools/test --setup` passes on a shirudo test machine
- [ ] `evidence/evidence.toml` records that test, with no exploit code
- [ ] The README names the side effects, the affected kernels and the sources
