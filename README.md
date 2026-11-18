# ZaiDrama iOS build runner

This repository contains only a manual macOS build workflow for the native Swift
ZaiDrama app. Application source (`ios/` and `frontend/public/brand/`) is fetched
from the private ZaiDrama repository using a read-only deploy key. Source code,
IPAs, signing material and plain build logs are never published here. The
public log reports stage results and, when validation, compilation or tests
fail, only the error locations and messages (`path:line:col: error: message`,
paths relative to the source root; no source lines). Full logs are saved as an
artifact encrypted to `DIAGNOSTICS_PUBLIC_CERT` and kept for one day. The
protected `appstore` environment permits only `main`.

Dispatch `ios.yml` on `main` with a full 40-character `source_sha` (any branch
commit of the private source):

- `upload_testflight` off, `ui_preview` off: regenerate and check the Xcode
  project, compile for the simulator and run the XCTest suite.
- `upload_testflight` on: the above, then sign, archive and upload to TestFlight.
- `ui_preview` on: sign and upload at once, without tests. For checking how
  screens look; not a release candidate.

Build numbers start at `100.x.y`, above the retired Expo / React Native builds
(`<run>.<attempt>`, last `26.1`) that used the same bundle identifier. The
marketing version defaults to `1.0.0` and can be set with the
`IOS_NATIVE_MARKETING_VERSION` variable. Uploaded builds are added to the
TestFlight groups named in `TESTFLIGHT_GROUPS` (empty means every group).
Real-device behavior and visual acceptance require human testing in TestFlight.
