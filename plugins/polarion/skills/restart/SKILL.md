---
name: polarion-restart
description: Restarts local ALM Polarion service with workspace cache clearing. Use when the user explicitly asks to restart Polarion, reload Polarion, or after extension code changes.
allowed-tools: Bash(net *), Bash(systemctl *), Bash(powershell *), Bash(rmdir *), Bash(rm *)
disable-model-invocation: true
---

# Polarion Restart

Restarts the local Polarion service with optional workspace cache clearing.

> **Key insight:** Polarion caches some specific files in `$POLARION_HOME/data/workspace/.config`. After
> extensions code changes, this cache folder **must be deleted** before starting the service — otherwise Polarion may
> load stale state and changes won't take effect. This is not obvious and easy to forget, so this skill
> ensures the cache clearing step is never skipped during a restart.

## Current Context

**Service status:**
`!`sc query Polarion 2>nul || systemctl is-active polarion 2>/dev/null``

**POLARION_HOME:**
`!`powershell -Command "[System.Environment]::GetEnvironmentVariable('POLARION_HOME', 'Machine')" 2>nul || echo $POLARION_HOME``

**Cache folder exists:**
`!`powershell -Command "if (Test-Path \"$([System.Environment]::GetEnvironmentVariable('POLARION_HOME', 'Machine'))\data\workspace\.config\") { 'YES - cache folder exists' } else { 'NO - cache folder not found' }" 2>nul || ([ -d "$POLARION_HOME/data/workspace/.config" ] && echo "YES - cache folder exists" || echo "NO - cache folder not found")``

## Execution Options

Choose based on the current context:

1. **Full restart with cache clear** (recommended after extension code changes):
   - Stop service
   - Delete `$POLARION_HOME/data/workspace/.config`
   - Start service

2. **Restart without cache clear** (for configuration-only changes):
   - Stop service
   - Start service

3. **Cache clear only** (service already stopped):
   - Delete `$POLARION_HOME/data/workspace/.config`

## Commands

Commands are OS dependent. Use the appropriate commands for the current operating system.

- **Stop**: `net stop Polarion` (Windows) or `systemctl stop polarion` (Linux). If already stopped, skip and proceed.
- **Delete cache**: Check that `POLARION_HOME` is set. On Windows, use `powershell -Command "[System.Environment]::GetEnvironmentVariable('POLARION_HOME', 'Machine')"` to read the system-level variable (do NOT use `echo %POLARION_HOME%` as it may not be visible in the current shell session). If set, delete `$POLARION_HOME/data/workspace/.config`. If not set, inform the user.
- **Start**: `net start Polarion` (Windows) or `systemctl start polarion` (Linux)

## On Failure

If any command fails:
1. Display the error output clearly
2. If stop fails with "service not started" — proceed (service is already stopped)
3. If cache deletion fails — check `POLARION_HOME` is set and the path exists, report to user
4. If start fails — show the error and suggest checking Polarion logs at `$POLARION_HOME/data/logs/`
