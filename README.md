# Account Cultivation Skill (Universal)

A universal skill for building real presence on any web platform. Uses [Camoufox](https://camoufox.com/) (anti-detect Firefox fork) for Cloudflare bypass, UI-based interaction for human-like behavior, and content humanization to remove AI writing patterns.

## What it does

```
Agent → Camoufox (anti-detect browser) → Target Website → UI Interactions → Humanized Content
```

- **Anti-detection**: Camoufox bypasses Cloudflare, fingerprints, and bot detection at the C++ level
- **UI interaction**: All actions via clicking, typing, scrolling — indistinguishable from a real user
- **Content humanization**: Removes AI writing patterns (crucial, Moreover, pivotal, etc.) before posting
- **Cross-platform**: Works on Linux, macOS, and Windows

## Quick start

```bash
# Install Camoufox
pip install -U "camoufox[geoip]"
python -m camoufox fetch

# Linux only: install dependencies
sudo apt install -y libgtk-3-0 libx11-xcb1 libasound2
```

```python
import os, platform
from camoufox.sync_api import Camoufox

def get_profile_path():
    system = platform.system()
    home = os.path.expanduser("~")
    if system == "Windows":
        base = os.path.join(os.environ.get("LOCALAPPDATA", home), "camoufox")
    else:
        base = os.path.join(home, ".camoufox")
    if os.path.exists(base):
        for item in os.listdir(base):
            if item.endswith(".default-default"):
                return os.path.join(base, item)
    return base

PROFILE = get_profile_path()

if platform.system() == "Linux":
    os.environ.setdefault("DISPLAY", ":1")

with Camoufox(headless=False, persistent_context=True, user_data_dir=PROFILE, humanize=True) as browser:
    page = browser.new_page()
    page.goto("https://target-site.com")
    page.wait_for_timeout(5000)
    print(f"Title: {page.title()}")
```

## Workflow (8 steps)

| Step | What | Why |
|------|------|-----|
| 0. Environment | Install Camoufox, detect OS/profile | Setup |
| 1. Anti-detection | Camoufox handles Cloudflare, fingerprints, etc. | Don't get banned |
| 2. Login detection | Check if account is logged in via UI signals | Need auth to post |
| 3. Content discovery | Browse feed, scroll, find posts | Act like a human |
| 4. Content interaction | Read the post (scroll, pause) | Anti-bot systems track this |
| 5. Humanization | Remove AI patterns from your text | AI text gets downvoted |
| 6. Posting | Click input, type, submit via UI | Real interaction |
| 7. Rate limiting | 4-8s delays, max 2 per session | Don't trigger rate limits |
| 8. Session summary | Log what was posted | Verification |

## Supported platforms (tested)

| Site | Login | Browse | Comment | Notes |
|------|-------|--------|---------|-------|
| Medium | ✅ | ✅ | ✅ | Responses dialog → contenteditable |
| Reddit | ✅ | ✅ | ✅ | textarea + Comment button |
| Quora | ✅ | ✅ | ✅ | /answer queue → contenteditable → Post |
| X/Twitter | Needs testing | — | — | |
| LinkedIn | Needs testing | — | — | |

## Adding a new site

1. Log in manually in Camoufox (cookies save to profile)
2. Run the selector discovery script (in SKILL.md)
3. Find: feed URL, content link selector, input field selector, submit button text
4. Test the full flow: browse → read → humanize → comment → verify

## Content humanization

Every piece of text posted must go through humanization:

**❌ AI-sounding:**
> "Great overview! The distinction between hype and actual deployment is a crucial turning point. Moreover, the 'show me the money' moment underscores the pivotal shift..."

**✅ Human:**
> "The 'show me the money' part hit home. Most companies I've talked to are stuck between running impressive demos and actually shipping AI that moves the needle on revenue."

See [SKILL.md](SKILL.md) for the full AI pattern deletion list and human pattern additions.

## Cross-platform support

| | Linux | macOS | Windows |
|---|---|---|---|
| Camoufox | ✅ | ✅ | ✅ |
| Profile path | `~/.camoufox/` | `~/.camoufox/` | `%LOCALAPPDATA%\camoufox\` |
| DISPLAY env | Required (`:1`) | Not needed | Not needed |
| Extra deps | `libgtk-3-0 libx11-xcb1 libasound2` | None | None |

## License

MIT
