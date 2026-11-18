# ZaiDrama iOS build runner

This repository contains only a manual macOS build workflow. Application source
is fetched from the private ZaiDrama repository using a read-only deploy key.
Source code, IPAs, signing material and plain build logs are never published here.
Failed build logs are encrypted to the owner’s diagnostics certificate and expire
after one day. The protected `appstore` environment permits only `main`.

Dispatch `ios.yml` with a full `source_sha`. Leave `upload_testflight` false for
client tests and signed device archive validation; set it true to
upload after those checks. The workflow compiles for physical iOS devices during
archiving and does not build a simulator app. Validation-only archives are
deleted after the run.
Real-device behavior and visual acceptance require human testing in TestFlight.

## Native Swift app (`ios-native.yml`)

The Swift rewrite lives under `ios/` in the private source and is built by
`ios-native.yml`, modeled on the Zai native workflow. Dispatch it on `main` with
a full `source_sha` (any branch commit of the private source):

- `upload_testflight` off, `ui_preview` off: generate and check the Xcode
  project, compile for the simulator and run the XCTest suite.
- `upload_testflight` on: the above, then sign, archive and upload.
- `ui_preview` on: sign and upload at once, without tests. Not a release candidate.

It uses the same `appstore` environment secrets and bundle identifier as
`ios.yml`. Native build numbers start at `100.0.1`, above the React Native
builds (`<run>.<attempt>`), so both can upload to the same app while the native
app reaches parity. The marketing version defaults to `1.0.0` and can be set
with the `IOS_NATIVE_MARKETING_VERSION` variable. Failure logs are encrypted to
`DIAGNOSTICS_PUBLIC_CERT` and kept for one day, as for `ios.yml`.
