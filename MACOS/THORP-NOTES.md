# Thorp fork notes (macOS)

This file is specific to the Thorp Design fork. It is kept separate from `README.md` and `CHANGELOG.md` so that merges from upstream do not conflict with it.

## Policy tracking

* Policy GUIDs (OIBIDs) live only in `MACOS/PolicyManifest.json`. Upstream also writes the OIBID into each policy's `description` field. We do not. Changing a description changes the policy, which forces a version bump and a redeploy of every macOS policy across the fleet, and we decided that was not worth it.
* Each manifest entry's `name` is the policy file name without `.json`. A policy's identity is that name with the trailing ` - vX.Y` or ` - vX.Y.Z` removed. When a policy is bumped, update `name` and keep the same `oibId`.
* `previousVersions` is empty for every entry. Upstream uses it for the GUIDs a policy had in earlier OIB releases, and none were issued before this manifest; the version trail is in git history.
* The manifest lists the 16 policies in `NativeImport/` plus the 3 compliance policies from `IntuneManagement/CompliancePolicies/` (`NativeImport/` has no compliance policies).
* `Scripts/Update-OIBManifest.ps1` cannot validate this manifest. It reads `IntuneManagement/` only, and it requires an OIBID in every description. Do not run it in Update mode against MacOS: it would rewrite descriptions.

## Things not to break

* The Passcode policy (`MacOS - OIB - SC - Device Security - Passcode`) has no `D` or `U` token in its name. Its All Devices assignment, with the macOS filter, was made by hand in Intune. Never rename its stem.
* `IntuneManagement/` holds legacy copies. They are not deployed and some are stale (older versions, and settings that have since changed in `NativeImport/`). `NativeImport/` is authoritative. See `IntuneManagement/README.md`.

## Deliberate exceptions to CIS and mSCP

### Major macOS upgrades deferred 90 days

`Updates - D - Update Configuration` sets `MajorPeriodInDays = 90`, the most Intune allows. CIS 1.6 asks for 30 days or fewer. We keep 90 on purpose.

* We do not take major upgrades on Apple's schedule. The custom `macOS - Software Update Target` policy, driven by `.deployment/macos-target-major` in intune-policies, enforces the major and point release we choose, by a deadline we set. Enforcement overrides the deferral, so a major we have approved still installs on time.
* The deferral only stops users upgrading to a new major by themselves before we have tested it. macOS 27 removed Rosetta and libiodbc on upgrade, which broke Vectorworks and needed packaged fixes first. Users moving early would have lost working software.
* Minor and security updates are not deferred, and Background Security Improvements install automatically, so the intent of the benchmark (patches arrive quickly) is still met.

Review this if the update target stops being maintained: without it, the deferral would hold every Mac 90 days behind a major release.

## Fork changelog

### 2026-10-01

* Device Security - Restrictions v2.0.2: removed the 17 keys that macOS 27 deprecated.
* Device Security - Restrictions v2.0.3: erase content and settings, and modification of printer sharing and Bluetooth sharing, are now blocked.
* Microsoft Edge - Security v2.0.2: removed the obsolete SSLVersionMin, and moved ProactiveAuthEnabled (obsolete, still false) to its replacement ProactiveAuthWorkflowEnabled.
* Microsoft OneDrive - Known Folder Move v2.0.1: removed the deprecated OpenAtLogin setting.
* Device Security - Passcode v2.0.1: failed-attempt lockout reset changed from 0 to 15 minutes, so the 10-attempt lockout now works.
* Updates - Update Configuration v2.0.2: users can no longer roll back Background Security Improvements.
* Added `PolicyManifest.json` (this file's companion), with no change to any policy.
