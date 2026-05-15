---
id: US-PLAT-001
epic: EPIC-01
sprint: S1
fr: []
priority: P0
estimate: M
status: Ready
owner: Mobile Lead
---

# US-PLAT-001 — Bootstrap Flutter project + folder layout

## User story

**As** a new engineer joining the team
**I want** a runnable Flutter project on first checkout
**so that** I can ship value within hours, not days.

## Acceptance criteria

1. `flutter create` baseline plus the layer-first structure documented in `architecture/tech-stack.md`:
   ```
   lib/
     @core/
     @share/
     application/
     plugins/
     resource/
     screen/
     widgets/
     main.dart
   ```
2. `flutter run` succeeds on a fresh checkout for both iOS (simulator + device) and Android (emulator + device).
3. README at repo root documents how to run, lint, test, and which Dart and Flutter versions are pinned.
4. `.tool-versions` (asdf) or `fvm` config locks the Flutter version.
5. `analysis_options.yaml` extends `flutter_lints` plus team rules; `flutter analyze` returns zero warnings on `main`.
6. iOS and Android bundle IDs for `dev`, `staging`, `prod` flavours decided and configured.

## Tasks

### Mobile
- [ ] Create project under chosen path with org id from sponsor.
- [ ] Add layer-first folder skeleton with placeholder `README.md` per layer.
- [ ] Configure `pubspec.yaml` baseline deps (no business deps yet).
- [ ] Configure flavours (`dev`, `staging`, `prod`).

### DevOps
- [ ] Pin Flutter version via `fvm` and document.
- [ ] Add `EditorConfig` and `.gitattributes`.

### Testing
- [ ] One smoke widget test that pumps `app/widget`.

## Dependencies

- DEC-009 (orchestration) does not block this story; CI lands later in US-PLAT-005.

## FR mapping

None (foundation).

## Test cases

Implicitly covered by US-PLAT-005 CI build verification.

## Definition of Done

- Fresh checkout to first `flutter run` ≤ 30 minutes including Pod install.
- `flutter analyze` clean.
- README updated.
