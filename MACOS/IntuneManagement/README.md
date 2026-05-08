# macOS IntuneManagement directory — not used by Thorp deployment

This directory is preserved upstream for compatibility with the legacy
[IntuneManagement][1] PowerShell tool. **It is not consumed by Thorp
Design's `deploy.py` pipeline.**

`deploy.py` reads `MACOS/NativeImport/` exclusively whenever that directory
contains JSON. The fallback to `MACOS/IntuneManagement/` only triggers
when `NativeImport/` is empty — never the case in practice.

When customising on a Thorp `thorp-macos-*` branch, edit only the
`NativeImport/` copy. The file names and versions in this directory may
diverge from `NativeImport/` after customisation; that's expected — only
`NativeImport/` is authoritative.

[1]: https://github.com/Micke-K/IntuneManagement
