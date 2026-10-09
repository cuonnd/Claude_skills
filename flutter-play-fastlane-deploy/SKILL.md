---
name: flutter-play-fastlane-deploy
description: Build a Flutter AAB and upload it to Google Play with fastlane supply (internal / beta / production, promote, changelogs, listing). Use when asked to bump a version, build an AAB, push an app to the Play Store, add release notes, or when `fastlane` is "command not found".
---
# Flutter → Google Play with fastlane

## Before building
1. Version lives in `pubspec.yaml` (`version: 1.0.1+8`, number after `+` = Play version code). Check the last version on Play first; only bump if the current one was already uploaded (Play rejects a reused version code).
2. Run the project's checks (`flutter analyze`, `flutter test`), then `flutter build appbundle --release` → `build/app/outputs/bundle/release/app-release.aab`.
3. If `flutter pub get` fails with "requires Flutter SDK version >=X", the dependency is newer than the installed Flutter: downgrade that package (as `pub` suggests) instead of upgrading Flutter.

## fastlane "command not found"
fastlane installed via `gem` is not on PATH. Binary is at `/opt/homebrew/lib/ruby/gems/3.3.0/bin/fastlane`. Either call it by full path or add to `~/.zshrc`:
`export PATH="/opt/homebrew/lib/ruby/gems/3.3.0/bin:$PATH"`

## Running lanes
- Run from the `android/` directory and use the platform prefix: `fastlane android <lane>`.
- Auth: service-account JSON via env `GOOGLE_PLAY_JSON_KEY_PATH=/path/key.json` (or `GOOGLE_PLAY_JSON_KEY_B64`). Keep the JSON out of git.
- Typical lanes: `internal`, `beta`, `production rollout:1.0`, `promote_to_production version_code:N rollout:1.0`, `listing_text`, `listing_all`. `skip_build:true` reuses an existing AAB.
- Play only accepts staged rollout below 100%; at 100% the release status is `completed`.
- Uploading the same version code twice fails ("Version code N has already been used"). If a build is already on internal, promote it instead of re-uploading.
- Production upload is an outward-facing action: confirm track and rollout % with the user. In auto-permission mode the command may be blocked; then give the user the exact command to run themselves.

## Release notes (changelogs)
- Play does not require them, but add one short line per locale.
- Keep committed sources in `android/fastlane/changelogs/<locale>/<version_code>.txt` (max 500 chars). `fastlane/metadata/` is usually generated + gitignored, so a lane helper should copy changelogs into `metadata/android/<locale>/changelogs/` before `supply` and set `skip_upload_changelogs: false`.
- Don't put implementation details (IDs, accounts) in the notes; "Bug fixes and stability improvements." is fine.
- Promote lanes reuse the release's existing notes; no changelog upload needed.

## Known Play errors
- Advertising ID declaration mismatch: the app manifest declares `AD_ID` permission but the Play Console declaration says no. Fix the declaration in Play Console (App content → Advertising ID).
- Package name in `Appfile`/`Fastfile` must match the `applicationId` in `android/app/build.gradle`.
