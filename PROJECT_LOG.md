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
