# PROJECT_HANDOFF.md

## 1. Project Overview

**Windows Notification Framework for Claude Code CLI on WSL2**

A notification framework that enables Windows toast notifications from WSL2 when using Claude Code CLI. Supports multi-language (EN, KO, JA, ZH) with detailed notifications similar to Codex CLI. v1.3.0 adds silent-by-default toasts, duplicate-notification suppression, and a terminal-title working spinner.

**Last updated**: 2026-04-17 14:10
**Version**: 1.3.1
**Last coding CLI used**: Claude Code CLI (v2.1.112)

## 2. Current State

| Component | Status | Notes |
|-----------|--------|-------|
| Core notification system | Completed | PowerShell BurntToast + WSL2 bridge |
| Multi-language templates | Completed | EN, KO, JA, ZH |
| Hook scripts (Stop, Notification, PermissionRequest) | Completed | Extracts last assistant message |
| Detailed notifications (Codex CLI style) | Completed | Reads transcript for last message |
| Global hook installation | Completed | Uses `$HOME` for portable paths |
| Silent-by-default toasts (v1.3.0) | Completed | `-Silent` to BurntToast; `--sound` to opt back in |
| Duplicate-toast suppression (v1.3.0) | Completed | `Notification.sh` exits on `idle_prompt` |
| Flicker-free busy indicator (v1.3.0) | Completed | Static shell-style title + OSC 9;4;3 taskbar pulse; per-tty state for concurrent sessions. Requires `"spinnerTipsEnabled": false` in `~/.claude/settings.json`. |
| Documentation | Completed | README.md, docs/HOOKS.md updated for v1.3.0 |
| Test coverage | Completed | 92% coverage, 63/63 tests passing (pre-v1.3.0; spinner/silent paths not yet covered) |

## 3. Execution Plan Status

| Phase | Status | Last Updated | Notes |
|-------|--------|--------------|-------|
| Core framework | Completed | 2026-01-12 | PowerShell + Bash bridge |
| Hook integration | Completed | 2026-02-11 | Stop, Notification, PermissionRequest |
| Detailed notifications | Completed | 2026-02-11 | Extracts last assistant message from transcript |
| Documentation v1.2.0 | Completed | 2026-02-11 23:15 | README and HOOKS.md updated |
| Global hook path fix | Completed | 2026-02-14 16:00 | Uses `$HOME` for portability |
| Installed-location refactor (v1.2.2) | Completed | 2026-02-14 17:10 | Hooks live in `~/.claude/hooks/wsl-toast/` |
| Silent + dedup + spinner (v1.3.0) | Completed | 2026-04-16 22:15 | Phases A-D executed in one session |
| Spinner refinement to flicker-free + per-tty | Completed | 2026-04-16 23:15 | Dropped Braille title loop (OSC 0 repaints caused flicker); kept OSC 9;4;3 taskbar pulse. State files keyed per-tty for concurrent sessions. Requires CC `spinnerTipsEnabled: false`. |
| Installer parity with v1.3.0 (v1.3.1) | Completed | 2026-04-17 14:10 | `setup.sh` now copies `_spinner.sh` + `UserPromptSubmit.sh`, registers UserPromptSubmit hook, writes `spinnerTipsEnabled: false`, defaults `config.json` to `silent: true`. |

## 4. Outstanding Work

No active work items. v1.3.1 is feature-complete.

Nice-to-have (not scheduled):
- Unit tests covering the new `--silent`/`--sound` branches in `notify.sh`
- Bats test for `_spinner.sh` PID lifecycle
- Session-id keyed spinner state (to support concurrent CC sessions in split panes)

## 5. Risks, Open Questions, and Assumptions

| Item | Status | Date Opened | Notes |
|------|--------|-------------|-------|
| Concurrent CC sessions share `~/.wsl-toast/spinner.pid` | Resolved 2026-04-16 | 2026-04-16 | State files now keyed per-tty (`spinner-dev_pts_N.*`) |
| Title bar overwritten by CC's own TUI spinner | Resolved 2026-04-16 | 2026-04-16 | User sets `"spinnerTipsEnabled": false` in `~/.claude/settings.json` |
| OSC 0 title animation causes visible flicker | Resolved 2026-04-16 | 2026-04-16 | Dropped the 150ms Braille loop; title is now static, animation comes from Windows Terminal's own taskbar pulse |

## 6. Verification Status

| Item | Method | Result | Date |
|------|--------|--------|------|
| Hook execution | Manual test | Pass | 2026-02-11 |
| Message extraction | Python script test | Pass | 2026-02-11 |
| Notification display | Manual test | Pass | 2026-02-11 |
| Duplicate fix (config-level) | Log analysis | Pass | 2026-02-11 |
| Global hook path fix | Settings verification | Pass | 2026-02-14 |
| Silent toast (v1.3.0) | Manual `notify.sh` call (no ding) | Pass | 2026-04-16 |
| Spinner start/stop (v1.3.0) | Standalone bash test | Pass | 2026-04-16 |
| `idle_prompt` suppression (v1.3.0) | Code review + log-path inspection | Pass | 2026-04-16 |

## 7. Restart Instructions

**Starting point**: v1.3.1 is complete. The next time Claude Code launches, the new hooks in `~/.claude/settings.json` will take effect automatically (no restart of CC required for existing sessions to pick up; new sessions only).

**Recommended next actions**:
1. Use Claude Code in a normal session; watch for the spinner in the Windows Terminal title bar / taskbar, and confirm toasts arrive silently without the ~10s idle_prompt duplicate.
2. If CC's own TUI overwrites the title-bar text so the Braille frames aren't visible, rely on the taskbar pulsing progress (OSC 9;4;3) as the primary indicator.
3. Consider adding automated tests for the new paths if this project is promoted out of personal use.

**Last updated**: 2026-04-17 14:10
