# MuxPod Fork — Patches & Learnings

This file tracks fork-specific modifications, device-verified findings, and platform gotchas
that upstream may not be aware of. Maintained on `fork/kargnas` branch only.

Upstream: [moezakura/mux-pod](https://github.com/moezakura/mux-pod)

---

## Patches

### fix(input): Samsung IME composing + modifier keys (PR #30)

- **File**: `lib/widgets/special_keys_bar.dart`
- **Symptom**: CTRL + letter (e.g. Ctrl+C) in DirectInput mode sends literal repeated text ("cccccc") instead of `C-c`
- **Root cause**: Samsung Keyboard (and some Android IMEs) treat English letter input as composing. The `composing=false` event may never arrive while the user keeps typing. The existing modifier check was in the post-composing path and therefore unreachable.
- **Fix**: Intercept the first composing character (`length == 1`) immediately when a modifier (CTRL/ALT) is active, with ASCII letter guard to avoid intercepting Korean (ㅊ) or other non-ASCII composing.
- **Also**: Extended CTRL-only handler to support ALT modifier (`M-` prefix) in DirectInput mode.
- **Tested on**: Galaxy Z Fold (SM-F966N), Android 16 (API 36), Samsung Keyboard

---

## Platform Learnings

### Samsung IME (Android)

- **English composing**: Samsung Keyboard treats English letter input as composing (`composing=true`), unlike Gboard which commits English letters directly. Characters accumulate ("c" → "cc" → "cccccc") while composing remains active.
- **composing=false timing**: The `composing=false` event may arrive late (~700ms after last composing event) or not at all until the user taps elsewhere. Do NOT rely on post-composing processing for time-sensitive operations like modifier key combos.
- **Implication for Option A vs B**: Post-composing fixes (Option B — normalize accumulated text after `composing=false`) are unreliable on Samsung. In-composing interception (Option A — act on first char when `length == 1`) is the proven approach.

### Android Work Profile

- Apps installed in the Work Profile (업무용 프로필, user ID 10) are isolated from VPN running in the Personal Profile (user ID 0). Tailscale VPN connections won't work for Work Profile apps.
- `adb install --user 0` forces installation to the Personal Profile.
- `adb shell pm list users` to list profiles, `adb shell ps -A | grep mux_pod` to check which user is running the app (`u0_` = personal, `u10_` = work).

### Flutter Build & Install

- This project uses `mise` for Flutter version management (`.mise.toml`). Run `mise trust` then `mise install` on first setup.
- Debug vs Release APK have different signing keys. Installing debug over a release (store) build requires uninstalling first (`INSTALL_FAILED_VERSION_DOWNGRADE`), which wipes app data.
- After reinstalling the app, toggle Tailscale VPN off/on to refresh the VPN routing table for the new app installation.

### DirectInput Mode Architecture

- `SpecialKeysBar` widget manages modifier state (`_ctrlPressed`, `_altPressed`, `_shiftPressed`) internally.
- DirectInput uses a sentinel character (`\u200B` zero-width space) for Backspace detection on iOS/iPadOS.
- `_onDirectInputChanged()` is the central handler with two main paths:
  1. **Composing path** (`_isComposing = true`): records text, returns early. Samsung IME hits this for English.
  2. **Post-composing path** (`_isComposing = false`): iOS dedup check → modifier check → send key.
- Hardware keyboard (Bluetooth/USB) goes through `_handleKeyEvent()`, a completely separate path.
- The `Input...` modal dialog (`_showInputDialog`) is a separate widget with NO access to modifier state. Modifier keys only work in DirectInput mode.
