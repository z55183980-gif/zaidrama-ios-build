# ZaiDrama iOS build runner

This repository hosts manual Apple-platform builds for ZaiDrama. The iOS workflow fetches `ios/` and `frontend/public/brand/`
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

## macOS desktop

Run `macos.yml` on this repository’s `main` and supply a lowercase, full `source_sha` from the private ZaiDrama repository. The same protected `appstore` environment supplies the read-only source deploy key and the diagnostics recipient’s public certificate. QA accepts reviewed private branch commits. Both modes run the macOS XCTest suites, build an Intel and Apple silicon application, verify the ZIP and open the packaged workspace through LaunchServices.

- `qa` builds with an ad hoc signature and does not load signing credentials.
- `release` requires the input SHA to match the current private `main`. It additionally loads `MACOS_DEVELOPER_ID_P12_BASE64` and `MACOS_DEVELOPER_ID_P12_PASSWORD` from `appstore` and uses the team’s existing App Store Connect key for notarization. The final archive must pass stapler, Gatekeeper and clean launch checks.

```sh
gh workflow run macos.yml -R z55183980-gif/zaidrama-ios-build --ref main -f source_sha="$SOURCE_SHA" -f mode=qa
```

After a successful run, download the `mac-<mode>-<sha>` artifact. Use the private key corresponding to the environment’s `DIAGNOSTICS_PUBLIC_CERT`. The decrypted tar contains the application ZIP, its SHA-256 checksum, the source SHA and, for a release, the notarization receipt. Failed builds provide a separate encrypted gzip-compressed tar of diagnostics. The private key stays with its holder.

```sh
gh run download "$RUN_ID" -R z55183980-gif/zaidrama-ios-build -n "mac-$MODE-$SOURCE_SHA" -D result
openssl smime -decrypt -inform DER -binary -in result/mac-package.enc -inkey "$RECIPIENT_KEY" -out result/mac-package.tar
tar -xf result/mac-package.tar -C result
(cd result && shasum -a 256 -c ./*.zip.sha256)
```
