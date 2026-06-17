# claude-channels-patch (v2.1.179+ fork)

> **This project was created with AI assistance (Claude Code / DeepSeek V4 Pro backend)**, forked from [wuzf/claude-channels-patch](https://github.com/wuzf/claude-channels-patch) and adapted for Claude Code v2.1.179+.
>
> **Disclaimer**: This project is for educational and research purposes only. Do not use on software you do not own or are not authorized to analyze.

A binary patch for **Claude Code** that enables `--channels` without claude.ai OAuth authentication.

## What It Does

Claude Code's `--channels` feature is gated by three layers:

| Layer | Check | Block Reason |
|------|------|------|
| **Feature Flag** | GrowthBook `tengu_harbor` / `tengu_harbor_permissions` | Defaults to `false`; API key / proxy users can't reach GrowthBook |
| **OAuth** | `accessToken` check | Third-party backend users (DeepSeek, etc.) don't have claude.ai tokens |
| **Allowlist** | Plugin/server whitelist | Unofficial plugins (e.g., WeChat weixin) not on the list |

This patch bypasses these restrictions by modifying the binary.

## Requirements

- Python 3.10+
- Claude Code v2.1.80+

## Compatibility

- ✅ Claude Code v2.1.179 (tested)
- ✅ Windows / Linux / macOS
- ✅ Third-party API proxy users (DeepSeek backend verified)
- ✅ Auto-detect Native Install / Homebrew / WinGet

## Usage

```bash
git clone git@github.com:842655waxdss/claude-channels-patch.git
cd claude-channels-patch

python patch.py                    # auto-detect and apply
python patch.py --check            # dry run
python patch.py --binary <path>    # specific binary
python patch.py revert             # restore from backup
```

## How It Works

### Core: Bun Source Fallback

Claude Code is a Node.js SEA (Single Executable Application). The patch replaces `@bytecode` markers with `@source__`, forcing the runtime to execute modified source code instead of compiled bytecode.

### Feature Flag Default Override

Changes `tengu_harbor` and `tengu_harbor_permissions` defaults from `false` to `true`, enabling channels even when GrowthBook is unreachable.

### Allowlist Bypass

Inverts the server-side allowlist check condition so that unlisted plugins can register.

### v2.1.179 Adaptations

Newer Claude Code versions (v2.1.179+) restructured some code paths, merging dual bundles into one and changing auth-related patterns. This fork:
- Relaxes pattern match thresholds from >=2 to >=1
- Makes auth/noAuth/bypasses optional (skip gracefully if pattern not found)
- Relies on bun source fallback + feature flag patches as the primary bypass mechanism

## Safety

- Automatically creates `.bak` backup before modification
- Atomic writes via temp file + `os.replace()`
- `python patch.py revert` to restore
- All patches use pattern matching, no hardcoded offsets

## License

MIT

## Credits

- Original: [wuzf/claude-channels-patch](https://github.com/wuzf/claude-channels-patch)
- Haleclipse: [Claude Code Channels 工人能智版](https://linux.do/t/topic/1787422)
