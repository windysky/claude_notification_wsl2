# PROJECT_LOG.md

---

## Session 2026-02-11 22:30-23:15

**Coding CLI used**: Claude Code CLI

**Phase(s) worked on**:
- Hook improvements (detailed notifications)
- Documentation updates
- Bug fix (duplicate notifications)

**Concrete changes implemented**:
1. Updated Stop.sh to extract last assistant message from transcript (like Codex CLI)
2. Updated Notification.sh to properly extract message field from payload
3. Removed duplicate hooks from ~/.claude/settings.json (user-level)
4. Updated README.md with current hooks and detailed notifications feature
5. Updated docs/HOOKS.md with current hook types and best practices
6. Bumped version to 1.2.0

**Files/modules/functions touched**:
- hooks/Stop.sh - Added transcript parsing, message extraction
- hooks/Notification.sh - Added message field extraction
- ~/.claude/settings.json - Removed hooks (kept project-level only)
- README.md - Updated features, hooks config, version
- docs/HOOKS.md - Complete rewrite for current hooks

**Key technical decisions and rationale**:
- **Transcript reading**: Stop hook now reads transcript_path from payload and extracts last assistant message, providing detailed notifications like Codex CLI's last-assistant-message feature
- **Project-level only**: Hooks should only be configured in project-level settings to avoid duplicates

**Problems encountered and resolutions**:
- **Duplicate notifications**: User reported receiving duplicate notifications. Investigation revealed hooks were configured in both user-level (~/.claude/settings.json) and project-level (.claude/settings.json). Fixed by removing hooks from user-level settings.

**Items explicitly completed**:
- Detailed notifications feature (Codex CLI style)
- Duplicate notification fix
- Documentation update for v1.2.0

**Verification performed**:
- Tested Python extraction logic - correctly extracts last assistant message
- Tested notify.sh directly - notifications sent successfully
- Analyzed logs to confirm duplicate notification cause

**Commits**:
- `ebe4251` feat: Extract last assistant message for detailed notifications
- `fd767ac` docs: Update documentation for v1.2.0 with detailed notifications

---

## Session 2026-02-14 14:00-16:00

**Coding CLI used**: Claude Code CLI

**Phase(s) worked on**:
- Global hook path configuration fix
- Investigation of moai-adk settings management

**Concrete changes implemented**:
1. Fixed setup.sh to use `$HOME` instead of hardcoded `/home/username/` paths for global hooks
2. Updated user's `~/.claude/settings.json` to use `$HOME` paths
3. Investigated moai-adk's `install.sh` and `ensureGlobalSettingsEnv()` function
4. Confirmed moai-adk only cleans up moai-specific hooks, not notification hooks

**Files/modules/functions touched**:
- `setup.sh` - Changed hook path generation to use `$HOME` for portability
- `~/.claude/settings.json` - Updated hook paths to use `$HOME`

**Key technical decisions and rationale**:
- **Global hooks need absolute paths**: Notification hooks must work from ANY project directory, not just from `claude-notification-wsl2`. Using `$HOME` makes paths portable across usernames.
- **moai-adk coexistence**: moai-adk's `ensureGlobalSettingsEnv()` only removes moai-specific hooks (`handle-*.sh`, `post_tool__*.py`), not notification hooks (`Notification.sh`, `Stop.sh`, `PermissionRequest.sh`)
- **Project-level vs Global**: moai-adk uses project-level hooks in `.claude/settings.json`; notification system uses global hooks in `~/.claude/settings.json`

**Problems encountered and resolutions**:
- **"Stop hook not found" error**: User reported `/home/juhur/PROJECTS/1COMPLETED1/agentic-cli-installer/hooks/Stop.sh: not found`. Investigation revealed global hooks used `$CLAUDE_PROJECT_DIR/hooks/Stop.sh` which only existed in the notification project. Fixed by using absolute paths (`$HOME/PROJECTS/.../hooks/Stop.sh`).

**Items explicitly completed**:
- Global hook path configuration fix
- User's global settings updated with `$HOME` paths

**Verification performed**:
- Verified global settings now use `$HOME/PROJECTS/.../hooks/` paths
- Confirmed moai-adk won't remove notification hooks (only removes `handle-*.sh` patterns)

**Commits**:
- `325033b` fix: Use $CLAUDE_PROJECT_DIR instead of hardcoded paths in hooks (superseded)
- `c05a88c` fix: Use absolute path for global hooks instead of $CLAUDE_PROJECT_DIR (superseded)
- `1fac376` fix: Use $HOME instead of hardcoded path in hook commands (final)

---

## Session 2026-02-14 16:30-17:15 (retroactive entry)

**Coding CLI used**: Claude Code CLI

**Phase(s) worked on**:
- Installation-location refactor (v1.2.1 -> v1.2.2)

**Concrete changes implemented**:
1. Moved hook scripts from `$HOME/PROJECTS/.../hooks/` into `~/.claude/hooks/wsl-toast/` for cleaner separation from project directory
2. Updated `scripts/notify.sh` to auto-discover the PowerShell script location (same-dir, parent, grandparent fallback chain) so it works from the installed location
3. Bumped version to 1.2.2

**Files/modules/functions touched**:
- `scripts/notify.sh` - `find_windows_dir` now searches multiple candidate roots
- `~/.claude/settings.json` - hook paths pointed at `$HOME/.claude/hooks/wsl-toast/`
- `~/.claude/hooks/wsl-toast/` - new installation directory

**Items explicitly completed**:
- Installation-location refactor

**Commits**:
- `cbc232b` chore: Bump version to 1.2.1
- `a894295` feat: Install hooks to ~/.claude/hooks/wsl-toast/ for better separation
- `620f49b` fix: Update notify.sh to find PowerShell script from installed location

---

## Session 2026-04-16 21:55-22:15

**Coding CLI used**: Claude Code CLI (v2.1.112, Opus 4.7)

**Phase(s) worked on**:
- v1.3.0 release: silent-by-default toasts, duplicate-toast suppression, terminal-title working spinner

**Concrete changes implemented**:

Phase A - Silent sound:
1. Added `-Silent` and `-Sound` switch parameters to `windows/wsl-toast.ps1`; passes `-Silent` to `New-BurntToastNotification` by default.
2. Added `--silent` / `--sound` CLI flags and `WSL_TOAST_SILENT` env var to `scripts/notify.sh`; default silent.
3. Extended `config.json` schema with new `silent` key; existing legacy `sound_enabled: false` also silences.
4. Flipped user's `~/.wsl-toast/config.json` to `silent: true, sound_enabled: false`.

Phase B - Kill duplicate toast:
5. `hooks/Notification.sh` now exits 0 when `notification_type == "idle_prompt"`. This suppresses the ~10s-after-Stop duplicate toast observed in `~/.wsl-toast/logs/hooks.log` (Stop fires first with the detailed last-message, then CC fires Notification idle_prompt a few seconds later).

Phase C - Terminal-title working spinner:
6. New shared helper `hooks/_spinner.sh` with `spinner_start` / `spinner_stop` functions. Discovers the user's terminal device by walking up the `/proc/<pid>/status` PPid chain until finding a `/dev/pts/*` on fd 0 or fd 1 (hook stdin is a pipe from CC, so `tty` won't work directly).
7. Spinner writes two escape sequences each frame:
   - `OSC 0` - Braille frame + "Claude working..." in window/icon title (visible in Windows Terminal title bar + taskbar text).
   - `OSC 9;4;3;0` - Windows Terminal indeterminate taskbar progress (pulsing yellow bar on the taskbar icon). Harmlessly ignored by other terminals.
8. New `hooks/UserPromptSubmit.sh` hook starts the spinner. Stop.sh and Notification.sh both call `spinner_stop` as the first action after logging.
9. Registered `UserPromptSubmit` in `~/.claude/settings.json`.

Phase D - Docs:
10. Updated `README.md`: new features bullet, v1.3.0 config example, new CLI flags documented, hook config example refreshed, Changelog entry.
11. Updated `docs/HOOKS.md` overview to include UserPromptSubmit + new behaviors.
12. Updated `PROJECT_HANDOFF.md` and appended this log entry.

**Files/modules/functions touched**:
- `windows/wsl-toast.ps1` - `-Silent` / `-Sound` params; `$script:IsSilent`; BurntToast `Silent=$true`
- `scripts/notify.sh` - `SILENT_MODE` default, `--silent`/`--sound` flags, config `silent` + legacy `sound_enabled` parsing, PS arg emission
- `hooks/_spinner.sh` - NEW shared helper
- `hooks/UserPromptSubmit.sh` - NEW spinner starter
- `hooks/Stop.sh` - calls `spinner_stop` before toast
- `hooks/Notification.sh` - calls `spinner_stop`; suppresses `idle_prompt`
- `~/.claude/settings.json` - registered `UserPromptSubmit` hook
- `~/.wsl-toast/config.json` - flipped to `silent: true`
- `~/.claude/hooks/wsl-toast/*` - mirrored all updated scripts
- `README.md`, `docs/HOOKS.md`, `PROJECT_HANDOFF.md`, `PROJECT_LOG.md` - documentation

**Key technical decisions and rationale**:
- **Silent-by-default**: user explicitly requested the ding off; defaulting to silent lets new installs be quiet with no config needed.
- **TTY discovery via PPid walk**: hook stdin is a pipe, so `tty` returns "not a tty". Walking up `/proc/$PPID/status` until we hit CC's controlling terminal is the only reliable way to get a writable `/dev/pts/*`.
- **OSC 9;4 secondary channel**: CC's own TUI may overwrite the title bar between our 150ms frames, so the Windows Terminal taskbar progress indicator (OSC 9;4) is a more reliable visibility signal. Taskbar progress pulses visibly even when the title is being rewritten.
- **`idle_prompt` suppression, not `Stop` removal**: keeping `Stop` preserves the detailed Codex-style last-message feature the user values; dropping the `Notification` branch only for `idle_prompt` keeps permission_prompt + auth_success alerts functional.
- **Background animator fully detached**: `(...) </dev/null >/dev/null 2>&1 &` with trap + disown ensures the spinner process survives hook exit and self-cleans on TERM.

**Problems encountered and resolutions**:
- **`tty` command fails in hook**: CC pipes a JSON payload to the hook's stdin, so `tty` reports "not a tty". Resolved by walking the PPid chain via `/proc/<pid>/fd/0` readlink.
- **Legacy `sound_enabled: true` in user's config**: would have clashed with new silent-by-default. Resolved by (a) making `sound_enabled: false` also silence and (b) auto-migrating user's config to `silent: true, sound_enabled: false`.
- **PowerShell Silent parameter compatibility**: guarded with `$paramNames -contains 'Silent'` so older BurntToast versions without `-Silent` don't error.

**Items explicitly completed**:
- Issue 1 (ding sound off) - silent-by-default with opt-in `--sound`
- Issue 2 (duplicate toasts with delay) - idle_prompt suppressed
- Issue 3 (working spinner) - Braille title-bar animation + OSC 9;4 taskbar progress
- Issue 4 (check native CC features) - confirmed CC 2.1.112 has its own in-UI spinner (`spinnerTipsEnabled`, `spinnerVerbs`) but no native WSL2 toast integration; this project does not block any native feature.

**Verification performed**:
- `notify.sh --title "..." --message "..."` - sent a real silent toast; user confirmed no ding.
- `spinner_start; sleep 4; spinner_stop` - confirmed Braille frames cycle in the parent terminal and Windows Terminal taskbar progress activates/clears correctly.
- `_spinner_find_user_tty` - returned `/dev/pts/3` in a standalone bash subshell, verifying the PPid walk.
- Code review of `Notification.sh` logic path for `idle_prompt` suppression.

**Commits**:
- (pending) v1.3.0 release commit — superseded by the 2026-04-16 22:30 session below, which refined the spinner design before committing.

---

## Session 2026-04-16 22:30-23:15

**Coding CLI used**: Claude Code CLI (v2.1.112, Opus 4.7)

**Phase(s) worked on**:
- Spinner refinement: Braille title loop → flicker-free static title + taskbar pulse
- Multi-session fix: per-tty state files
- Doc sync, gitignore of moai-adk local dirs, v1.3.0 commit + push

**Concrete changes implemented**:

1. User tested v1.3.0 and reported the title flickering between our Braille frames and CC's built-in title ("Claude running"). Root cause: OSC 0 title writes every 150ms repaint the title bar, and CC 2.1.112 has its own `spinnerTipsEnabled` (default true) also writing to the title.

2. Disabled CC's built-in title spinner by adding `"spinnerTipsEnabled": false` to `~/.claude/settings.json`.

3. Experimented with capturing the live title via xterm `\033[21t` query so we could prepend the Braille frame to the user's real shell title. Failed: inside a hook, CC owns the tty's input stream so the terminal's response goes to CC, not our hook. `stty raw` manipulation on the shared tty also risks disturbing CC's I/O. Abandoned.

4. Settled on reconstructing a shell-style title from env vars (`user@host: ~/path`) since that's what PROMPT_COMMAND would print anyway — no blocking reads, always works.

5. User then reported the animated title was still flickering because 150ms OSC 0 rewrites cause a visible repaint on every frame — flicker is inherent to the mechanism. Dropped the title animation loop entirely. Final design:
   - Title is written ONCE (static) on `UserPromptSubmit`
   - Animation comes from Windows Terminal's own indeterminate taskbar progress (OSC 9;4;3), which is system-animated and flicker-free
   - No background animator process; no PID file needed
   - Tradeoff: no Braille spinner in the title — but the user preferred zero flicker over the animation

6. User discovered concurrent CC sessions stepped on each other because state files were single-instance (`~/.wsl-toast/spinner.{pid,tty,title}`). Fixed by keying per tty — state files now named `spinner-dev_pts_N.*`. Added legacy cleanup.

7. Synced all changes to the installed location at `~/.claude/hooks/wsl-toast/_spinner.sh`.

8. Updated README, docs/HOOKS.md, PROJECT_HANDOFF.md to match the final flicker-free design.

9. Extended `.gitignore` to cover moai-adk local directories (`.agency/`, `.claude/agents/`, `.claude/commands/`, `.claude/hooks/`, `.claude/output-styles/`, `.claude/rules/`, `.claude/skills/`, `.moai/`) plus ephemeral `archive/` and `resume_session.txt`.

**Files/modules/functions touched**:
- `hooks/_spinner.sh` — rewrote: removed Braille animation loop, title-capture attempt, and PID-tracking animator; added `_spinner_file_prefix` (per-tty keying), `_spinner_build_title` (shell-style title from env), `_spinner_cleanup_legacy` (pre-v1.3.1 state cleanup)
- `~/.claude/settings.json` — added `"spinnerTipsEnabled": false`
- `README.md`, `docs/HOOKS.md` — replaced "Braille spinner" copy with accurate flicker-free description
- `PROJECT_HANDOFF.md` — updated status rows, moved multi-session/flicker items from Known to Resolved
- `.gitignore` — added moai-adk local dirs + ephemeral files

**Key technical decisions and rationale**:
- **Dropped title animation**: OSC 0 cannot be updated without repainting the title bar. No amount of tuning eliminates the flicker. The Windows Terminal taskbar pulse (OSC 9;4;3) is a cleaner channel because it's animated by the terminal itself, not by our writes.
- **Reconstruct title instead of capturing**: querying the live title from a hook is fundamentally blocked by CC owning the tty's input. Reconstructing from `$USER@$HOSTNAME: $PWD` is essentially what bash's default PROMPT_COMMAND does anyway.
- **Per-tty state keys, not session_id**: the tty is available without parsing the hook's JSON payload and it's inherently correct (the spinner is a terminal-level effect).
- **No `moai-adk` artifacts committed**: the `.agency/`, `.moai/`, `.claude/agents/` etc. dirs are the user's local moai-adk runtime, not part of this project. Gitignored.

**Problems encountered and resolutions**:
- **Flicker (reported by user)**: resolved by removing the animation loop entirely.
- **Multi-session state clobber (reported by user)**: resolved by per-tty keying.
- **Stale docs**: README said "Braille spinner in title bar" — corrected before shipping.
- **CC's own built-in title spinner**: identified `spinnerTipsEnabled` as the switch; now disabled in global settings.

**Items explicitly completed**:
- Flicker-free busy indicator
- Multi-session support
- Documentation sync with actual shipped behavior
- v1.3.0 git commit + push

**Verification performed**:
- User confirmed taskbar pulse is visible and flicker-free in their Windows Terminal
- User confirmed original title (and `spinnerTipsEnabled: false`) prevents CC from overwriting
- Clarified that Windows Terminal's green-asterisk tab indicator is tab-focus-dependent (only shows on background tabs), not a bug in our code

**Commits**:
- `fac8367` feat: v1.3.0 — silent toasts, idle_prompt dedup, flicker-free busy indicator

