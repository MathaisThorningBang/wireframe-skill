---
name: wireframe
description: >
  Turns a live website URL into a grayscale HTML wireframe with placeholder text.
  Navigates the page with Playwright MCP, captures a full-page screenshot and
  accessibility snapshot, analyzes layout structure (grids, flex, sections,
  component types), then generates a self-contained HTML wireframe using only
  grayscale colors and placeholder copy. Saves to Desktop with auto-versioning,
  screenshots, and a reference markdown file. Triggers on: "wireframe",
  "/wireframe", "make a wireframe", "wireframe this site", "wireframe this URL".
---

# Wireframe — Live-Site-to-Grayscale-HTML Wireframe

Turn any live website URL into a faithful grayscale wireframe in a single
self-contained HTML file.

## Invocation

```
/wireframe {project-name} {url}
```

Example: `/wireframe superhi https://www.superhi.com`

## Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `project-name` | Yes | Kebab-case name used for the output folder |
| `url` | Yes | Full URL of the page to wireframe |

## Output Location

```
~/Downloads/wireframes/{project-name}-wireframes/
├── wireframe-v{N}.html      ← Self-contained grayscale wireframe
├── wireframe-v{N}.png       ← Full-page screenshot of the wireframe
├── original-v{N}.png        ← Full-page screenshot of the live site
└── WIREFRAMES.md             ← Version log with metadata
```

All project folders live inside `~/Downloads/wireframes/` as a shared parent directory.

Version number `N` auto-increments. If the folder already has `wireframe-v1.html`
and `wireframe-v2.html`, the next run creates `wireframe-v3.html`.

---

## Process (follow each phase in order)

### Phase 0: Setup

1. Parse the two arguments: `PROJECT_NAME` and `TARGET_URL`.
2. Set `OUTPUT_DIR` to `~/Downloads/wireframes/{PROJECT_NAME}-wireframes/`.
3. Create the directory if it does not exist:
   ```bash
   mkdir -p ~/Downloads/wireframes/{PROJECT_NAME}-wireframes
   ```
4. Determine the next version number:
   ```bash
   ls ~/Downloads/wireframes/{PROJECT_NAME}-wireframes/wireframe-v*.html 2>/dev/null | grep -oE 'v[0-9]+' | sed 's/v//' | sort -n | tail -1
   ```
   If no files exist, use `1`. Otherwise use `max + 1`.
   Store as `VERSION`.

### Phase 1: Navigate and Dismiss Overlays

1. Navigate to the target URL:
   ```
   mcp__playwright__browser_navigate  url={TARGET_URL}
   ```

2. Wait 2-3 seconds for the page to settle (lazy-loaded content, animations).

3. Dismiss cookie banners and popups. Run this JavaScript via `mcp__playwright__browser_evaluate`:
   ```javascript
   () => {
     const selectors = [
       '[id*="cookie"] button', '[class*="cookie"] button',
       '[id*="consent"] button', '[class*="consent"] button',
       'button[aria-label*="accept"]', 'button[aria-label*="Accept"]',
       'button[aria-label*="agree"]', '.cc-btn', '.cc-allow',
       '#onetrust-accept-btn-handler', '.js-cookie-accept',
       '[data-testid*="cookie"] button', '[data-testid*="accept"]'
     ];
     for (const sel of selectors) {
       const btn = document.querySelector(sel);
       if (btn) { btn.click(); break; }
     }
     document.querySelectorAll('[class*="overlay"], [class*="modal"], [id*="popup"]').forEach(el => {
       const style = getComputedStyle(el);
       if (style.position === 'fixed' || style.position === 'sticky') {
         if (el.offsetHeight < window.innerHeight * 0.5) el.remove();
       }
     });
   }
   ```

4. Wait 1 second for any dismiss animation.

### Phase 2: Trigger Lazy Content

Before capturing, force all lazy-loaded content to render by scrolling through
the entire page in a single evaluate call:

```
mcp__playwright__browser_evaluate
```

```javascript
() => new Promise(resolve => {
  const totalHeight = document.documentElement.scrollHeight;
  const step = window.innerHeight;
  let current = 0;
  const interval = setInterval(() => {
    current += step;
    window.scrollTo(0, current);
    if (current >= totalHeight) {
      clearInterval(interval);
      window.scrollTo(0, 0);
      setTimeout(resolve, 1000);
    }
  }, 300);
})
```

This scrolls the full page to trigger all lazy-loaded images, animations, and
intersection-observer content, then scrolls back to the top. One tool call
replaces the entire scroll loop.

### Phase 3: Capture the Full Page

Now capture everything in **two parallel tool calls**:

1. **Full-page screenshot** — captures the entire page in one image:
   ```
   mcp__playwright__browser_take_screenshot  type=png  fullPage=true  filename=~/Downloads/wireframes/{PROJECT_NAME}-wireframes/original-v{VERSION}.png
   ```

2. **Accessibility snapshot** — captures the full DOM tree with roles, text,
   and element hierarchy for the entire page (not just the viewport):
   ```
   mcp__playwright__browser_snapshot
   ```

These two calls together give you everything needed: the screenshot for visual
reference (proportions, spacing, imagery) and the snapshot for structural data
(element types, hierarchy, text content, grid/flex patterns).

**That's it. No scroll loop needed.** Both tools capture the full page.

### Phase 4: Analyze Layout

Before generating HTML, analyze the screenshot and snapshot together.
Identify and document:

1. **Page sections** (in order, top to bottom):
   - Announcement / top bar (if present)
   - Navigation bar (sticky? transparent? hamburger?)
   - Hero section (full-width? split layout? background image?)
   - Logo / partner bar
   - Feature/benefit sections (grid? alternating image+text?)
   - Card grids (how many columns? aspect ratio?)
   - Testimonials / social proof (carousel? grid? single?)
   - Pricing tables (how many tiers? toggle?)
   - FAQ / accordion sections
   - Newsletter / CTA sections
   - Footer (columns? link groups? newsletter?)

2. **Layout patterns** for each section:
   - Grid: column count, gap estimates
   - Flex direction and alignment
   - Max-width constraints
   - Padding and margin rhythm (identify the spacing scale)

3. **Typography hierarchy**:
   - Number of distinct heading sizes (map to h1-h6)
   - Body text size category (small / medium / large)
   - Font weight patterns

4. **Visual structure details**:
   - Border radius patterns (sharp / slightly rounded / pill)
   - Shadow usage (none / subtle / prominent)
   - Dividers / separators between sections
   - Background alternation pattern (which sections are lighter/darker)

### Phase 5: Generate Wireframe HTML

Create a single self-contained HTML file following these exact rules:

#### Grayscale Palette (use ONLY these colors — NO other colors allowed)

| Token | Hex | Usage |
|-------|-----|-------|
| `--white` | `#ffffff` | Page background, card backgrounds |
| `--gray-50` | `#fafafa` | Alternating section backgrounds |
| `--gray-100` | `#f0f0f0` | Subtle backgrounds, input fields |
| `--gray-200` | `#e0e0e0` | Borders, dividers, image placeholders |
| `--gray-300` | `#cccccc` | Secondary borders, disabled states |
| `--gray-400` | `#999999` | Placeholder text, captions, subtle labels |
| `--gray-500` | `#666666` | Body text, descriptions |
| `--gray-700` | `#333333` | Headings, primary text, nav links |
| `--gray-900` | `#1a1a1a` | Hero headings, bold emphasis, dark section bg |

**CRITICAL: No brand colors. No blues, greens, reds, or any hue whatsoever.
The entire wireframe must be shades of gray only.**

#### HTML Template

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{PROJECT_NAME} — Wireframe v{VERSION}</title>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
  <style>
    :root {
      --white: #ffffff;
      --gray-50: #fafafa;
      --gray-100: #f0f0f0;
      --gray-200: #e0e0e0;
      --gray-300: #cccccc;
      --gray-400: #999999;
      --gray-500: #666666;
      --gray-700: #333333;
      --gray-900: #1a1a1a;
    }

    *, *::before, *::after { margin: 0; padding: 0; box-sizing: border-box; }

    body {
      font-family: 'Inter', -apple-system, system-ui, sans-serif;
      background: var(--white);
      color: var(--gray-500);
      -webkit-font-smoothing: antialiased;
      line-height: 1.5;
    }

    /* ... all section styles inline here ... */
  </style>
</head>
<body>
  <!-- All wireframe sections here -->
</body>
</html>
```

#### CSS Rules

- Use `font-family: 'Inter', -apple-system, system-ui, sans-serif` everywhere.
- Set `max-width` on content containers to match the original site's content
  width (typically 1200px or 1440px, centered with `margin: 0 auto`).
- Preserve the original's spacing rhythm. If sections have ~80px vertical
  padding, use 80px. If cards have 24px gaps, use 24px.
- Use CSS Grid or Flexbox to match the original layout precisely.
- Sticky navs should be `position: sticky; top: 0; z-index: 100`.
- Use the gray CSS custom properties, not raw hex values.

#### Content Replacement Rules

**CRITICAL: ZERO real copy from the original site may appear in the wireframe.
Every single piece of text — headings, paragraphs, labels, section titles,
button text, stat labels, testimonials, footer text — must be replaced with
generic placeholder copy. This includes large display headings, section labels
(e.g. "WHY WE ARE HERE"), animated text, taglines, and any other visible text.
Do NOT copy any original wording, even paraphrased. The wireframe is a
structural/layout reference only.**

| Original Element | Wireframe Replacement |
|------------------|-----------------------|
| Logo | Gray rectangle with text "Logo" centered |
| Nav links | "Link One", "Link Two", "Link Three", etc. |
| Hero heading | "Main Heading Text Here" |
| Hero subheading | "Supporting description paragraph that provides context about the main message above." |
| Section labels / eyebrow text | Generic labels: "Section Label", "About Us", "Features", etc. |
| Large display headings | "A Bold Statement Heading That Spans Multiple Lines" or similar generic text of matching length |
| CTA buttons | "Get Started" / "Learn More" (generic labels) |
| Paragraphs | Lorem ipsum of approximately the same line count |
| Card titles | "Card Title Here" |
| Card descriptions | "Brief description text that explains this particular feature or item in a few words." |
| Images / Videos | `<div>` with `background: var(--gray-200)`, matching aspect ratio, with centered text "Image" in `var(--gray-400)` |
| Avatars | `<div>` with `border-radius: 50%; background: var(--gray-300);` sized to match |
| Icons | Small gray circles or squares (24x24 or 32x32) with `background: var(--gray-200)` |
| Partner/client logos | Gray rectangles in a row with text "Brand" |
| Stats/numbers | "100+", "50K", "99%" (generic numbers) |
| Stat labels | "Items processed", "Users served", etc. (generic) |
| Testimonial quotes | "This is a testimonial quote placeholder that represents customer feedback and social proof." |
| Testimonial names | "Person Name" |
| Testimonial titles | "Job Title, Company" |
| Input fields | Styled `<input>` or `<div>` with placeholder text |
| Footer column headings | "Category One", "Category Two", etc. |
| Footer links | "Footer Link" repeated appropriately |
| Social icons | Small gray circles in a row |
| Announcement bars | Gray bar with "Announcement text placeholder" |
| Badges/tags | Small rounded `<span>` with "Tag" text |
| Dropdown selects | Gray bordered rectangle with "Select option" |
| Copyright text | "© 2026 Company Name" |
| Location / legal text | "City, Country" / "Reg. 00 00 00 00" |

#### Section Background Alternation

Alternate section backgrounds between `var(--white)` and `var(--gray-50)` to
visually separate sections, matching the original's pattern. If the original
has a dark/colored section, use `var(--gray-900)` background with `var(--white)`
text. If the original has a colored section that is not dark (e.g. yellow, light
blue), use `var(--gray-100)` background.

#### Desktop-Only

The wireframe does NOT need to be responsive. Build it at **1440px viewport
width** to match the desktop capture.

### Phase 6: Save the Wireframe

Write the generated HTML to:
```
~/Downloads/wireframes/{PROJECT_NAME}-wireframes/wireframe-v{VERSION}.html
```

### Phase 7: Screenshot the Wireframe

1. Navigate Playwright directly to the file:
   ```
   mcp__playwright__browser_navigate  url=file:///Users/mathiasbang/Desktop/wireframes/{PROJECT_NAME}-wireframes/wireframe-v{VERSION}.html
   ```

2. Wait 2 seconds for fonts to load.

3. Take a full-page screenshot:
   ```
   mcp__playwright__browser_take_screenshot  type=png  fullPage=true  filename=~/Downloads/wireframes/{PROJECT_NAME}-wireframes/wireframe-v{VERSION}.png
   ```

No HTTP server needed — `file://` URLs work directly in Playwright.

### Phase 8: Update WIREFRAMES.md

Create or update `~/Downloads/wireframes/{PROJECT_NAME}-wireframes/WIREFRAMES.md`.

If the file does not exist, create it with this structure:

```markdown
# {PROJECT_NAME} Wireframes

## Version {VERSION}
- **Source URL:** {TARGET_URL}
- **Date:** {YYYY-MM-DD}
- **File:** [wireframe-v{VERSION}.html](wireframe-v{VERSION}.html)
- **Screenshot:**

![wireframe-v{VERSION}](wireframe-v{VERSION}.png)

- **Sections captured:**
  - (list each section identified in Phase 4 with brief description)
```

If the file already exists, insert the new version entry immediately after the
`# {PROJECT_NAME} Wireframes` heading line, pushing older versions down.
Newest version should always be at the top.

### Phase 9: Report to User

After completing all phases, provide a brief summary:

1. Confirm the wireframe was saved and the full path
2. List the sections that were captured
3. Note the version number
4. Mention the screenshot file
5. Provide the open command: `open ~/Downloads/wireframes/{PROJECT_NAME}-wireframes/wireframe-v{VERSION}.html`

---

## Edge Cases

### Single-page apps with client-side routing
If the page appears blank or minimal after navigation, wait 3-5 seconds
and retry the snapshot. Some SPAs need time to hydrate.

### Lazy-loaded sections that appear empty
The Phase 2 scroll-through should trigger all lazy content. If the snapshot
still shows empty containers after this, try a second scroll-through with
longer delays (500ms per step instead of 300ms).

### Sites requiring authentication
If the page redirects to a login page, inform the user and stop. Do not
attempt to bypass authentication.

### Pages with horizontal scroll or unconventional layouts
Capture at 1440px width. If the page has horizontal scroll, note it in
WIREFRAMES.md but wireframe only the visible 1440px width.

---

## Quality Checklist (verify before delivering)

Before reporting to the user, verify:

- [ ] HTML file is fully self-contained (only external dependency is Google Fonts)
- [ ] Zero color hues used — only the 9 grayscale values from the palette
- [ ] No real copy from the original site — all text is placeholder
- [ ] Layout structure matches the original (same grid columns, same flex patterns, same section order)
- [ ] Sticky nav stays sticky if the original had one
- [ ] Image placeholders have approximately correct aspect ratios
- [ ] Section count matches the original page
- [ ] WIREFRAMES.md is created/updated with correct metadata
- [ ] Screenshot PNG exists and shows the complete wireframe
- [ ] Full-page original screenshot is saved
