# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Project Overview

Binary patch for Claude Code that enables the `--channels` feature without requiring claude.ai OAuth authentication. Forked from [wuzf/claude-channels-patch](https://github.com/wuzf/claude-channels-patch), adapted for **Claude Code v2.1.179+**.

This fork was created with AI assistance (Claude Code / DeepSeek V4 Pro backend).

## Key Changes from Original

- **Threshold relaxed**: Pattern match minimum reduced from 2 to 1 (newer Claude Code versions merged dual JS bundles)
- **Auth/Allowlist bypasses made optional**: Patterns that changed in v2.1.179 now skip gracefully instead of failing
- **Tested on v2.1.179** with DeepSeek third-party API backend

## Architecture

Python patch script (`patch.py`) that:

1. Auto-detects Claude Code binaries (native `.exe` + npm versions)
2. Replaces `@bytecode` markers with `@source__` to force source-mode execution
3. Modifies feature flag defaults (`tengu_harbor`, `tengu_harbor_permissions`) from `false` to `true`
4. Patches allowlist/server bypass checks
5. Handles platforms: Windows, Linux, macOS (auto codesign)

All patches use pattern-matching with stable anchors (string literals), not hardcoded offsets.

## Usage

```bash
python patch.py                    # auto-detect and patch
python patch.py --check            # dry run
python patch.py --binary <path>    # specific binary
python patch.py revert             # restore from backup
```
