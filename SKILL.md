---
name: account-cultivation-generic
description: "Universal account cultivation skill for any website. Uses Camoufox (anti-detect Firefox fork) for Cloudflare bypass, UI-based interaction for human-like behavior, and humanizer for content polishing. Works on Medium, Reddit, Quora, X, LinkedIn, and any web platform."
---

# Universal Account Cultivation Skill

Build and maintain a real presence on any web platform using Camoufox (anti-detect Firefox fork). All interactions via UI — clicking, typing, scrolling — indistinguishable from a real user.

---

## Architecture

```
Agent
  → Camoufox Python API (anti-detect browser)
    → Persistent profile (cookies, login state)
      → Target website
        → UI interactions (click, type, scroll)
          → Humanized content (AI patterns removed)
```

**Key principle: Everything happens inside the browser, like a real user.** No API calls, no curl, no headless requests. The browser has real cookies, real fingerprints, real mouse movements.

---

## Step 0: Environment Setup

### Install Camoufox (once, all platforms)

```bash
pip install -U "camoufox[geoip]"
python -m camoufox fetch
```

Linux 额外依赖：
```bash
# Debian/Ubuntu
sudo apt install -y libgtk-3-0 libx11-xcb1 libasound2
# Arch
sudo pacman -S gtk3 libx11 libxcb cairo libasound alsa-lib
```

macOS 和 Windows 无需额外依赖。

### Profile location (auto-detect OS)

```python
import os, platform

def get_profile_path():
    """Auto-detect Camoufox profile path based on OS."""
    system = platform.system()
    home = os.path.expanduser("~")
    
    if system == "Linux":
        base = os.path.join(home, ".camoufox")
    elif system == "Darwin":  # macOS
        base = os.path.join(home, ".camoufox")
    elif system == "Windows":
        base = os.path.join(os.environ.get("LOCALAPPDATA", home), "camoufox")
    else:
        base = os.path.join(home, ".camoufox")
    
    # Find the default profile directory
    if os.path.exists(base):
        for item in os.listdir(base):
            if item.endswith(".default-default"):
                return os.path.join(base, item)
    return base

PROFILE = get_profile_path()
```

### Verify Camoufox works (cross-platform)

```python
import platform
from camoufox.sync_api import Camoufox

PROFILE = get_profile_path()

# Linux needs DISPLAY for headed mode; macOS/Windows don't
if platform.system() == "Linux":
    os.environ.setdefault("DISPLAY", ":1")

with Camoufox(headless=False, persistent_context=True, user_data_dir=PROFILE, humanize=True) as browser:
    page = browser.new_page()
    page.goto('https://target-site.com')
    page.wait_for_timeout(5000)
    print(f'Title: {page.title()}')
```

---

## Step 1: Anti-Detection (Why Camoufox)

### What Camoufox does (so you don't have to)

| Detection vector | How Camoufox defeats it |
|---|---|
| **Cloudflare Turnstile** | `solve_cloudflare=True` — solves challenges automatically |
| **navigator.webdriver** | C++ level patching — `navigator.webdriver` returns `undefined` |
| **Playwright/Juggler detection** | Sandboxed isolation — automation code runs outside page context |
| **Fingerprint consistency** | BrowserForge generates statistically realistic fingerprints (OS, screen, GPU match real-world distributions) |
| **Headless detection** | Headless mode patched to appear identical to headed mode |
| **WebGL/Canvas fingerprint** | C++ level spoofing — not detectable via JS inspection |
| **Mouse movement patterns** | `humanize=True` — C++ implementation of natural cursor trajectories |
| **Font fingerprinting** | Bundled fonts match target OS profile |

### What YOU must still handle

| Risk | Mitigation |
|---|---|
| **Posting too fast** | 4-8 second delays between actions |
| **No browsing before posting** | Always scroll, read, pause before interacting |
| **Identical comments** | Each comment must be unique |
| **AI-sounding text** | Always run through humanizer (Step 5) |
| **Posting at exact intervals** | Add random variance to delays |
| **Same subreddit/page every time** | Rotate target pages |

### Camoufox Python API pattern (cross-platform)

```python
import os, platform
from camoufox.sync_api import Camoufox

def get_profile_path():
    """Auto-detect Camoufox profile path based on OS."""
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

# Linux needs DISPLAY for headed mode
if platform.system() == "Linux":
    os.environ.setdefault("DISPLAY", ":1")

with Camoufox(
    headless=False,                    # Visible browser (headed mode)
    persistent_context=True,           # Keep cookies between sessions
    user_data_dir=PROFILE,             # Auto-detected profile path
    humanize=True,                     # Natural mouse movements
) as browser:
    pages = browser.pages
    page = pages[0] if pages else browser.new_page()
    # ... interactions ...
```

**Parameters explained:**
- `headless=False` — Required for cultivation. Headless may leak detection signals.
- `persistent_context=True` — Keeps cookies, login state, browsing history.
- `user_data_dir=PROFILE` — Auto-detected path, works on Linux/macOS/Windows.
- `humanize=True` — Enables C++ level human-like mouse movements.

---

## Step 2: Login Detection

Before any action, verify the account is logged in. Each site has different signals.

### Universal detection pattern

```python
page.goto("https://target-site.com", timeout=30000)
page.wait_for_timeout(5000)

# Check for login indicators (adapt per site)
login_signals = {
    "has_avatar": bool(page.query_selector('[aria-label*="user"]') or page.query_selector('[aria-label*="profile"]') or page.query_selector('[aria-label*="menu"]')),
    "has_signin_link": bool(page.query_selector('a[href*="signin"]') or page.query_selector('a[href*="login"]')),
    "has_create_button": bool(page.query_selector('a[href*="create"]') or page.query_selector('button:has-text("Create")')),
}

if login_signals["has_avatar"] and not login_signals["has_signin_link"]:
    print("LOGGED IN")
else:
    print("NOT LOGGED IN — manual login required")
```

### Per-site login indicators

| Site | Logged in signal | Not logged in signal |
|---|---|---|
| Medium | `a[href*="/@"]` profile link | `a[href*="signin"]` |
| Reddit | `button:has-text("Expand user menu")` | `a[href*="login"]` |
| Quora | URL stays at `/answer` | Redirects to `/login` |
| X/Twitter | `a[href*="/home"]` | `a[href*="/login"]` |
| LinkedIn | `a[href*="/feed/"]` | `a[href*="/login"]` |

### First-time login

If not logged in, you must log in manually once:
1. Open Camoufox in headed mode
2. Navigate to the site's login page
3. Log in with credentials (manually or via script)
4. The persistent profile saves cookies automatically
5. Future sessions reuse the saved login

---

## Step 3: Content Discovery (Browse Like a Human)

**NEVER navigate directly to a post/page and immediately interact.** Always browse first.

### Universal browsing pattern

```python
# 1. Navigate to the feed/listing page
page.goto("https://target-site.com/feed", timeout=30000)
page.wait_for_timeout(5000)

# 2. Scroll slowly (simulate reading)
page.mouse.wheel(0, 600)
page.wait_for_timeout(2000)
page.mouse.wheel(0, 600)
page.wait_for_timeout(2000)

# 3. Find content links
links = page.evaluate("""() => {
    let results = [];
    let seen = new Set();
    for (let a of document.querySelectorAll('a[href]')) {
        let text = (a.innerText || '').trim();
        let href = a.href;
        if (text.length > 15 && !seen.has(href) && href.includes('/post/') || href.includes('/article/') || href.includes('/comments/')) {
            seen.add(href);
            results.push({href: href, text: text.substring(0, 100)});
        }
    }
    return results.slice(0, 10);
}""")

# 4. Pick a target (avoid meta/announcement posts)
target = None
for link in links:
    if 'welcome' not in link['text'].lower() and 'rules' not in link['text'].lower():
        target = link
        break
```

### Per-site discovery

| Site | Feed URL | Link pattern |
|---|---|---|
| Medium | Homepage or tag page | `a[href*="/@"]` with article slug |
| Reddit | `/r/SUBREDDIT/new` | `a[href*="/comments/"]` |
| Quora | `/answer` queue | Answer buttons on question cards |
| X/Twitter | Home timeline | Tweet permalinks |
| LinkedIn | Feed page | `a[href*="/posts/"]` |

### Selection criteria

- **Engagement**: Posts with 1-20 comments (not too crowded, not dead)
- **Recency**: Posted within last 24 hours
- **Topic**: Match your account's niche
- **Avoid**: Controversial/political, meta posts, announcements

---

## Step 4: Content Interaction (Read Before You Write)

**Always read the content before commenting.** Anti-bot systems track whether you actually consumed the content.

### Universal reading pattern

```python
# 1. Navigate to the target content
page.goto(target['href'], timeout=30000)
page.wait_for_timeout(6000)

# 2. Scroll through like reading
page.mouse.wheel(0, 800)
page.wait_for_timeout(3000)    # Pause — "reading"
page.mouse.wheel(0, 800)
page.wait_for_timeout(2000)    # Pause — "reading"
page.mouse.wheel(0, 600)
page.wait_for_timeout(2000)    # Pause — "reading"

# 3. Check the content (optional — for personalizing your comment)
title = page.title()
print(f"Reading: {title}")
```

### Timing guidelines

| Action | Minimum delay | Recommended |
|---|---|---|
| Page load → first scroll | 3s | 5-8s |
| Between scrolls | 2s | 3-5s |
| Reading before commenting | 8s | 15-30s |
| Between posts/comments | 4s | 8-15s |

---

## Step 5: Content Humanization (MANDATORY)

**Every piece of text you post MUST go through humanization.** AI-sounding content gets downvoted, reported, and flagged.

### The 3-step process

```
1. DRAFT  → Write the raw comment/answer/post
2. HUMANIZE → Remove AI patterns, add human voice
3. POST   → Type the humanized version
```

### AI patterns to DELETE

**Words (kill on sight):**
- crucial, pivotal, vital, significant, key
- Moreover, Furthermore, Additionally, Notably
- underscores, highlights, emphasizes, showcases
- evolving landscape, tapestry, testament, vibrant
- delve, foster, garner, intricate, interplay
- serves as, stands as, represents a shift
- groundbreaking, revolutionary, innovative
- it's worth noting, it's important to note

**Structures (kill on sight):**
- Rule of three: "A, B, and C" → just say the most important one
- Negative parallelism: "It's not just X, it's Y" → just say Y
- Em dash overuse: "X — Y — Z" → use commas or periods
- Sycophantic openers: "Great question!", "Excellent point!" → delete entirely
- Hedging: "It could potentially possibly be argued that" → just say it
- Copula avoidance: "serves as", "stands as" → use "is", "are", "has"

### Human patterns to ADD

- **First-person perspective**: "I think", "In my experience", "I've found that"
- **Opinions**: Have a stance. Don't be neutral.
- **Specific details**: Concrete examples > vague claims
- **Varied sentence rhythm**: Mix short punchy sentences with longer ones
- **Uncertainty**: "I'm not sure but", "honestly", "I could be wrong"
- **Personal experience**: "I burned myself on this", "the last time I tried"
- **Imperfect structure**: Real humans don't write in perfect paragraphs

### Before/After examples

**❌ AI-sounding:**
> "Great overview! The distinction between hype and actual deployment is a crucial turning point. Moreover, the 'show me the money' moment for Chief AI Officers underscores the pivotal shift from experimental to production-grade AI. This really highlights how the landscape is evolving."

**✅ Human:**
> "The 'show me the money' part hit home. Most companies I've talked to are stuck between running impressive demos and actually shipping AI that moves the needle on revenue. Boards are done being wowed by benchmarks."

**❌ AI-sounding:**
> "Great question! This is a crucial topic that highlights the importance of fostering a deeper understanding. Moreover, it's worth noting that the evolving landscape presents both challenges and opportunities."

**✅ Human:**
> "Honestly it depends on what you're optimizing for. If you want speed, go simple. But if you're building for scale, it's worth investing the extra time upfront. I've burned myself on the 'quick and dirty' path more times than I'd like to admit."

---

## Step 6: Posting (UI Interaction)

**Always post via UI interaction** — click, type, submit. Never use internal APIs.

### Universal posting pattern

```python
# 1. Scroll to comment/response area
page.mouse.wheel(0, 1200)
page.wait_for_timeout(2000)

# 2. Find the input field
comment_box = None
for sel in ['textarea', '[contenteditable="true"]', 'div[role="textbox"]', 'input[type="text"]']:
    try:
        els = page.query_selector_all(sel)
        for el in els:
            if el.is_visible():
                comment_box = el
                break
        if comment_box:
            break
    except:
        pass

if comment_box:
    # 3. Click to focus
    comment_box.scroll_into_view_if_needed()
    page.wait_for_timeout(1000)
    comment_box.click(timeout=10000)
    page.wait_for_timeout(1000)

    # 4. Type humanized text (character by character, like a human)
    comment_box.type(humanized_text, delay=30)
    page.wait_for_timeout(2000)

    # 5. Find and click submit button
    submit_btn = None
    for btn in page.query_selector_all('button'):
        try:
            text = btn.inner_text().strip()
            if text in ["Comment", "Post", "Reply", "Respond", "Submit"] and btn.is_visible():
                submit_btn = btn
                break
        except:
            pass

    if submit_btn:
        submit_btn.click()
        page.wait_for_timeout(5000)
        print("Posted!")
```

### Per-site submit buttons

| Site | Button text | Notes |
|---|---|---|
| Medium | "Respond" | Inside responses dialog |
| Reddit | "Comment" | Below textarea |
| Quora | "Post" | Below editor, may be disabled initially |
| X/Twitter | "Post" / "Reply" | Blue button |
| LinkedIn | "Post" / "Comment" | Blue button |

### Verification

After posting, verify it worked:
```python
# Check if your text appears in the page
content = page.content()
if humanized_text[:30] in content:
    print("Verified: comment visible")
else:
    print("Warning: comment may not have posted")
```

---

## Step 7: Rate Limiting

### Universal limits

| Action | Minimum delay | Max per session | Max per day |
|---|---|---|---|
| Between page loads | 3s | — | — |
| Between scrolls | 2s | — | — |
| Reading before commenting | 8s | — | — |
| Between comments | 4-8s | — | — |
| Comments per session | — | 2 | — |
| Comments per day | — | — | 5-10 |

### Random variance

Add ±20% random variance to all delays to avoid pattern detection:
```python
import random
delay = base_delay * (0.8 + random.random() * 0.4)  # ±20%
```

---

## Step 8: Session Summary

**Always end with a summary.** This is for the user (or cron job output) to verify what happened.

```
## Cultivation Session Summary

| # | Platform | Target | Action | Result |
|---|----------|--------|--------|--------|
| 1 | Reddit | r/CasualConversation | Comment | ✅ Posted |
| 2 | Medium | @author/article | Comment | ✅ Posted |

**Profile:** ~/.camoufox/xyz.default-default
**Duration:** 2m 30s
**Humanized:** Yes (all content polished)
```

---

## Adding a New Site

To cultivate a new site, follow this checklist:

1. **Login**: Open Camoufox, navigate to site, log in manually. Cookies save to profile.
2. **Test Cloudflare**: Navigate to the site. If "Just a moment..." appears, wait. Camoufox should solve it.
3. **Find feed URL**: What's the listing page? (`/feed`, `/new`, `/home`, etc.)
4. **Find content links**: What CSS selector identifies real content links?
5. **Find input field**: What selector matches the comment/reply box?
6. **Find submit button**: What text does the submit button have?
7. **Test the full flow**: Browse → read → comment → verify.
8. **Document selectors**: Add to your skill file.

### Selector discovery script

```python
# Run this to discover a site's selectors
page.goto("https://new-site.com", timeout=30000)
page.wait_for_timeout(5000)

# Find all interactive elements
elements = page.evaluate("""() => {
    let results = [];
    for (let el of document.querySelectorAll('a, button, textarea, [contenteditable], input, [role="textbox"]')) {
        let text = (el.innerText || el.placeholder || el.value || '').trim().substring(0, 50);
        let tag = el.tagName;
        let cls = el.className.toString().substring(0, 50);
        let href = el.href || '';
        if (text || cls) {
            results.push({tag, text, cls, href: href.substring(0, 80)});
        }
    }
    return results.slice(0, 50);
}""")

for el in elements:
    print(f"[{el['tag']}] text='{el['text']}' cls='{el['cls']}' href='{el['href']}'")
```

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Cloudflare "Just a moment..." | IP-level block | Camoufox should solve it. If persistent, IP is flagged — wait or use proxy. |
| "Not logged in" | Profile cookies expired | Log in again manually in the Camoufox profile |
| Comment box not found | Wrong selector | Run selector discovery script (above) |
| Submit button disabled | React state not updated | Wait 3-5 seconds after typing, then retry |
| Comment not visible after posting | Silent write block (IP flagged) | Check if IP is datacenter-flagged. May need proxy. |
| Page won't load | Site down or blocking | Check site status, try again later |
| "Element not visible" | Hidden behind overlay | Close popups/modals, scroll to element |
| Camoufox crashes | Profile corruption | Restore from backup or create fresh profile |
| Type error on `comment_box.type()` | Element not focusable | Use `page.keyboard.type()` instead |

---

## Files and Paths (auto-detected by `get_profile_path()`)

| Item | Linux | macOS | Windows |
|------|-------|-------|---------|
| Camoufox binary | `~/.cache/camoufox/camoufox` | `~/Library/Caches/camoufox/camoufox` | `%LOCALAPPDATA%\camoufox\camoufox.exe` |
| Default profile | `~/.camoufox/<id>.default-default/` | `~/.camoufox/<id>.default-default/` | `%LOCALAPPDATA%\camoufox\<id>.default-default\` |
| GeoIP database | `~/.cache/camoufox/GeoLite2-City.mmdb` | `~/Library/Caches/camoufox/GeoLite2-City.mmdb` | `%LOCALAPPDATA%\camoufox\GeoLite2-City.mmdb` |
| Display env | `DISPLAY=:1` required | Not needed | Not needed |

---

## References

- Camoufox docs: https://camoufox.com/
- Camoufox Python API: https://camoufox.com/python/usage/
- Humanizer skill: `skill_view(name='humanizer')`
- Per-site skills: `medium-cultivate-ubuntu`, `reddit-cultivate-ubuntu`, `quora-cultivate-ubuntu`
