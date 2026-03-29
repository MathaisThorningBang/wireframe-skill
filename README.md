# Wireframe Skill

Turn any live website URL into a grayscale HTML wireframe with a single command. Built for [Claude Code](https://claude.ai/claude-code) using Playwright MCP.

## Install

```bash
npx skills add https://github.com/MathaisThorningBang/wireframe-skill --skill wireframe
```

## Usage

In Claude Code, run:

```
/wireframe {project-name} {url}
```

Example:

```
/wireframe superhi https://www.superhi.com
```

## What it does

1. Opens the URL in Playwright and dismisses cookie banners/popups
2. Scrolls through the page to trigger all lazy-loaded content
3. Captures a full-page screenshot + accessibility snapshot
4. Analyzes layout: sections, grids, typography, spacing, visual patterns
5. Generates a self-contained grayscale HTML wireframe with placeholder text
6. Screenshots the wireframe itself
7. Saves everything to `~/Downloads/wireframes/{project-name}-wireframes/`

## Output

```
~/Downloads/wireframes/{project-name}-wireframes/
├── wireframe-v1.html      # The wireframe
├── wireframe-v1.png       # Screenshot of the wireframe
├── original-v1.png        # Full-page screenshot of the live site
└── WIREFRAMES.md          # Version log
```

Versions auto-increment. Run it again on the same project and you get `v2`, `v3`, etc.

## Rules

- **Grayscale only** — 9 shades from white to `#1a1a1a`, zero color hues
- **Zero real copy** — all text replaced with generic placeholders
- **Desktop only** — built at 1440px width
- **Self-contained** — single HTML file, only external dependency is Google Fonts (Inter)

## Requirements

- [Claude Code](https://claude.ai/claude-code)
- [Playwright MCP server](https://github.com/anthropics/playwright-mcp) configured in Claude Code
