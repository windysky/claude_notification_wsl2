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

---

## Session 2026-04-17 13:30-14:15

**Coding CLI used**: Claude Code CLI (v2.1.112, Opus 4.7)

**Phase(s) worked on**:
- v1.3.1 installer fix — `setup.sh` parity with v1.3.0 runtime

**Context**:
User pulled the v1.3.0 work on a second machine and ran `./setup.sh`. Spinner still wasn't visible. Investigation showed `setup.sh` had never been updated for v1.3.0: it was stuck at v1.2.2, copying only the three original hooks, not offering the `UserPromptSubmit` hook, not touching `spinnerTipsEnabled`, and rewriting `~/.wsl-toast/config.json` back to the pre-v1.3.0 `sound_enabled: true` default (silently re-enabling the Windows ding). The v1.3.0 release commit (`fac8367`) updated runtime code + docs but forgot the installer.

**Concrete changes implemented**:
1. `setup.sh` header: bumped `# Version: 1.2.2` → `1.3.1`
2. `create_default_config()`: new `~/.wsl-toast/config.json` template now writes `silent: true, sound_enabled: false` (matches v1.3.0 runtime default, prevents regression on re-install)
3. `install_hook_scripts()`: copy list extended with `UserPromptSubmit.sh` and `_spinner.sh`
4. `configure_claude_hooks()`: added `UserPromptSubmit` prompt (default Y), its timeout prompt, and its env var passthrough
5. Python block inside `configure_claude_hooks()`: added registration branch for `UserPromptSubmit` hook, and when that branch fires it also sets `settings["spinnerTipsEnabled"] = False` at the top level
6. README.md: bumped "Version" line to 1.3.1 (2026-04-17); added 1.3.1 changelog entry describing installer fix
7. PROJECT_HANDOFF.md: version → 1.3.1, last-updated → 2026-04-17 14:10, added execution-plan row for the installer fix, updated restart-instructions prose
8. PROJECT_LOG.md: this entry

**Files/modules/functions touched**:
- `setup.sh` — header, `create_default_config`, `install_hook_scripts`, `configure_claude_hooks` (both bash-side prompts and embedded Python block)
- `README.md` — Version line, Changelog (new 1.3.1 entry prepended)
- `PROJECT_HANDOFF.md` — version, last-updated, execution plan table, Outstanding Work wording, Restart Instructions
- `PROJECT_LOG.md` — appended this session

**Key technical decisions and rationale**:
- **`spinnerTipsEnabled: false` tied to UserPromptSubmit opt-in**, not a separate global toggle: the setting only matters when the spinner hook is active; applying it unconditionally would surprise users who decline the spinner.
- **Legacy `sound_enabled: false` kept in the new default**: `scripts/notify.sh` still respects the legacy key for backward compat, so writing both `silent: true` and `sound_enabled: false` is belt-and-suspenders for older `notify.sh` copies that might still be on disk.
- **Stale "PID lifecycle" nice-to-have NOT rewritten yet**: the PROJECT_HANDOFF.md nice-to-have list mentions a Bats test for `_spinner.sh` PID lifecycle, but v1.3.0's refactor removed the PID file entirely (no background animator). Left alone — it's clearly a nice-to-have, not blocking, and rewording is cosmetic.
- **Historical version strings preserved**: `scripts/notify.sh:62` ("Silent by default in v1.3.0+"), `windows/wsl-toast.ps1:87` (same), and `setup.sh:414` ("v1.3.0 spinner helpers") describe *when* a feature was introduced and are intentionally not bumped.

**Problems encountered and resolutions**:
- **Running `setup.sh` reverted `~/.wsl-toast/config.json` to the v1.2.x sound-enabled default**: root cause was the stale `create_default_config` template. Fixed by flipping the template to `silent: true, sound_enabled: false`.
- **`tty` command fails from within a hook / agent subprocess**: documented earlier in v1.3.0; not a new issue — mentioned here only because current-session diagnostics returned "not a tty" and required a PPid-walk check to identify installed-vs-source drift.

**Items explicitly completed**:
- Fresh-clone installability of v1.3.0 runtime behavior
- Version bump to 1.3.1 across active declarations
- Handoff + log updated

**Verification performed**:
- `bash -n setup.sh` — passes
- `./setup.sh --dry-run` — runs clean; no Python exceptions in the embedded block
- Visual review of the modified `configure_claude_hooks()` Python: `UserPromptSubmit` registration path and `spinnerTipsEnabled` mutation both gated on `HOOK_ENABLE_USERPROMPTSUBMIT == "true"`
- User instructed to run `./setup.sh --force` and restart CC session for live verification (post-commit)

**Commits**:
- `d7f4b4d` fix: v1.3.1 — installer parity with v1.3.0 runtime

---

## Session 2026-04-17 14:30-14:50

**Coding CLI used**: Claude Code CLI (v2.1.112, Opus 4.7)

**Phase(s) worked on**:
- v1.3.2 bug fix — Windows PowerShell ExecutionPolicy block on WSL remote paths

**Context**:
After v1.3.1 install on the second machine, user reported the spinner worked but no Windows toast appeared. Diagnostics showed the Stop hook was firing correctly (logged the full assistant message to `~/.wsl-toast/logs/hooks.log`), but a manual `notify.sh` invocation returned:

```
File \\wsl.localhost\Ubuntu\home\juhur\.claude\hooks\wsl-toast\windows\wsl-toast.ps1
cannot be loaded. The file ... is not digitally signed. You cannot run
this script on the current system.
+ FullyQualifiedErrorId : UnauthorizedAccess
```

Root cause: Windows PowerShell treats UNC-style WSL paths (`\\wsl.localhost\Ubuntu\...`) as remote/network paths. The default `RemoteSigned` execution policy rejects unsigned scripts from remote paths. The v1.3.0 release happened to work on the original dev machine because its policy was configured more permissively (likely `Unrestricted` or `Bypass`). Fresh WSL installs on stock Windows hit this wall.

**Concrete changes implemented**:
1. `scripts/notify.sh` (line 291): `POWERSHELL_ARGS` now includes `-ExecutionPolicy Bypass` before the `-File` argument, with an inline comment explaining the WSL-remote-path rationale
2. Synced patched `scripts/notify.sh` to `~/.claude/hooks/wsl-toast/notify.sh` for immediate in-session verification
3. Manual test `notify.sh --title "Fix Test" --message "..."` returned `[INFO] Notification sent successfully`; user confirmed toast appeared
4. Version bumped 1.3.1 → 1.3.2 in `setup.sh`, `README.md`, `PROJECT_HANDOFF.md`
5. README.md: prepended 1.3.2 changelog entry
6. PROJECT_HANDOFF.md: new execution-plan row, last-updated + restart-instructions version refs

**Files/modules/functions touched**:
- `scripts/notify.sh` — one-line fix + comment in `POWERSHELL_ARGS` construction
- `~/.claude/hooks/wsl-toast/notify.sh` — synced (installed copy)
- `setup.sh` — header version
- `README.md` — version line, changelog
- `PROJECT_HANDOFF.md` — version, last-updated, execution plan, restart instructions
- `PROJECT_LOG.md` — this entry

**Key technical decisions and rationale**:
- **`-ExecutionPolicy Bypass` on the command line is safe and scoped**: it only applies to this PowerShell invocation, not the system policy. Corporate lockdown via Group Policy can still override it, but the common stock-Windows `RemoteSigned` default yields to command-line override.
- **Placed before `-File`**: PowerShell's argument parser requires execution-policy overrides to precede the script path.
- **Did not sign the script**: code signing would require a certificate + distribution infrastructure; Bypass is a one-line fix with equivalent effect for a personal tool.
- **Patch release, not minor**: bugfix only, no new features, no config or user-facing schema change. SemVer patch (1.3.1 → 1.3.2).

**Problems encountered and resolutions**:
- **Silent toast failure masked by "success" logs**: `Stop.sh` was firing and logging correctly; the failure was downstream in `notify.sh`'s PowerShell call. Had to run `notify.sh` manually (with stderr visible) to surface the PS error. Suggests adding toast-delivery status to `hooks.log` would help future diagnosis; not done here to keep the fix minimal.

**Items explicitly completed**:
- Toast delivery on machines with default `RemoteSigned` PowerShell policy

**Verification performed**:
- Manual `~/.claude/hooks/wsl-toast/notify.sh --title "Fix Test" --message "..."` → `[INFO] Notification sent successfully`
- User confirmed toast visible in Windows Action Center

**Commits**:
- (this commit) fix: v1.3.2 — add -ExecutionPolicy Bypass for WSL remote-path block

