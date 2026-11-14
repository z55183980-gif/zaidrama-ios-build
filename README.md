# ZaiDrama iOS build runner

This repository contains only a manual macOS build workflow. Application source
is fetched from the private ZaiDrama repository using a read-only deploy key.
Source code, IPAs, signing material and plain build logs are never published here.
Failed build logs are encrypted to the owner’s diagnostics certificate and expire
after one day. The protected `appstore` environment permits only `main`.

Dispatch `ios.yml` with a full `source_sha`. Leave `upload_testflight` false for
compilation and client tests; set it true to sign and upload after those checks.
Real-device behavior and visual acceptance require human testing in TestFlight.
