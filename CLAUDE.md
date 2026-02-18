# CLAUDE.md — WebsiteBusinessCard

This file provides guidance for AI assistants working on this repository.

---

## Project Overview

A personal digital business card / portfolio website for Marvin Graupner, an IT specialist based in Zürich, Switzerland. The site is a bilingual (German/English) single-page application built entirely with vanilla HTML, CSS, and JavaScript — no build tools, no package manager, no framework.

Live at: **www.marvininja.ch** (deployed via GitHub Pages)

---

## Repository Structure

```
WebsiteBusinessCard/
├── index.html               # Entire application (HTML + embedded CSS + embedded JS)
├── CNAME                    # GitHub Pages custom domain: www.marvininja.ch
├── CLAUDE.md                # This file
├── vcards/
│   └── marvin_business.vcf  # Downloadable vCard (v3.0) for contact saving
└── img/
    ├── bg.jpg               # Background/hero image
    ├── pb2.jpg              # Profile photo
    ├── favicons-1.png       # ACTIVE favicon (35KB — used in <link> tags)
    ├── favicon.png          # Unused favicon variant (1.5MB)
    ├── favicon2.png         # Unused favicon variant (1.1MB)
    ├── favicon3.png         # Unused favicon variant (1.5MB)
    └── favicons.png         # Unused favicon variant (1.3MB)
```

> **Note:** `favicons-1.png` is the only favicon actually referenced in `index.html`. The other favicon variants are unused and can be safely ignored.

---

## Technology Stack

| Layer | Choice |
|---|---|
| Markup | HTML5 (semantic elements) |
| Styling | CSS3 (embedded in `<style>` block inside `index.html`) |
| Scripting | Vanilla JavaScript ES6+ (embedded in `<script>` block inside `index.html`) |
| Icons | Font Awesome 6.4.2 (loaded from CDN) |
| Fonts | System font stack: `Inter`, `-apple-system`, `BlinkMacSystemFont`, `Segoe UI` |
| Hosting | GitHub Pages |
| Build system | **None** — no npm, no Webpack, no Vite, no transpilation |

---

## Development Workflow

### No build step required

There is no build process. Edit `index.html` directly and open it in a browser to preview changes. No `npm install`, no `npm run build`, no compilation.

### Running locally

```bash
# Any of these will work:
open index.html                        # macOS
xdg-open index.html                    # Linux
python3 -m http.server 8080            # Simple local server (recommended)
```

A local HTTP server (e.g. `python3 -m http.server`) is preferred over opening the file directly so that relative paths resolve correctly.

### Deployment

Push to the `master` branch (or `main` on the remote). GitHub Pages automatically serves the site from the repository root. The `CNAME` file configures the custom domain `www.marvininja.ch`.

### Branches

| Branch | Purpose |
|---|---|
| `master` | Primary development branch |
| `claude/*` | Feature branches created by AI assistants |
| `remotes/origin/main` | Remote primary branch |

---

## Code Architecture

### Everything lives in `index.html`

The file (~1,230 lines) has three parts:

1. **`<head>`** — meta tags, favicon links, Font Awesome CDN link, and the entire `<style>` block (~500 lines of CSS)
2. **`<body>`** — full HTML structure of the page
3. **`<script>`** block at the end of `<body>` (~130 lines of JavaScript)

### HTML Patterns

**Language-specific content** — Elements meant for only one language carry a `data-lang` attribute:

```html
<span data-lang="de">Über mich</span>
<span data-lang="en">About me</span>
```

JavaScript toggles `display` on these elements when the user switches language.

**Section/tab system** — Main sections use `.about-content` + an `id`, revealed by adding the `.open` class:

```html
<div class="about-content" id="leben"> ... </div>
```

Sub-tabs inside sections use `.tab-content` + `.tab-content-<group>` + an `id`:

```html
<div class="tab-content tab-content-leben" id="interests"> ... </div>
```

**Inline event handlers** are used throughout (the codebase does not use `addEventListener`):

```html
<a href="#" onclick="showSection('leben', event)">...</a>
<button onclick="showTab('leben', 'interests', this)">...</button>
```

**Data attributes for sub-navigation grouping:**

```html
<div class="subnav" data-tab-group="leben"> ... </div>
```

### CSS Patterns

**Color palette** (keep consistent with these values):

| Token | Value | Usage |
|---|---|---|
| Primary accent | `#28e1da` | Cyan — highlights, borders, glows |
| Background dark | `#0a0e1a` | Page background |
| Background mid | `#1a1f35` | Gradient mid-stop |
| Background card | `rgba(30,38,54,0.8)` | Glassmorphic card backgrounds |
| Text primary | `#e8e8e8` | Body text |
| Text muted | `rgba(232,232,232,0.6)` | Secondary text |
| Border subtle | `rgba(255,255,255,0.1)` | Card borders |

**Design system — glassmorphism:**

Cards use a frosted-glass look achieved with:
```css
background: rgba(30,38,54,0.8);
backdrop-filter: blur(20px);
border: 1px solid rgba(255,255,255,0.1);
border-radius: 16px;
```

**Key CSS classes:**

| Class | Purpose |
|---|---|
| `.glass` | Glassmorphic container |
| `.bento-card` | Grid card tiles |
| `.timeline-entry` | Career timeline rows |
| `.about-content` | Collapsible main section |
| `.tab-content` | Collapsible sub-tab |
| `.open` | Reveals a hidden section or tab |
| `.active-btn` | Highlights the currently active nav button |
| `.lang-toggle` | Language switcher pill |
| `.orb`, `.orb1`, `.orb2` | Animated background glow orbs |
| `.private-number` | Blurred/privacy-hidden text |

**Responsive breakpoint:** `max-width: 768px` — a single breakpoint for mobile layout.

**Animations defined:**

| Keyframe | Description |
|---|---|
| `bgMove` | Slow scrolling dot-grid background |
| `float` | Orb floating up/down |
| `fadeInUp` | Entry fade-in for cards |

### JavaScript Patterns

All JS is in a single `<script>` block at the bottom of `<body>`. There are no modules, no classes, no external dependencies.

**Global state variables:**

```js
let currentLang = 'de';      // Active language: 'de' | 'en'
let currentRole = 0;         // Index into roles array for typing animation
let charIndex = 0;           // Current character position in typing animation
let reverse = false;         // Whether typing animation is erasing
```

**Key functions:**

| Function | Signature | Description |
|---|---|---|
| `setLanguage` | `(lang: 'de' \| 'en') => void` | Switches all `data-lang` elements, updates toggle buttons, resets typing animation |
| `showSection` | `(sectionId: string, event: Event) => void` | Shows a main section, hides others, activates first sub-tab |
| `showTab` | `(tabGroup: string, tabId: string, element: HTMLElement) => void` | Shows a sub-tab within a section |
| `typeAnim` | `() => void` | Recursive timeout-based typing animation |

**Typing animation** cycles through `rolesDE` / `rolesEN` arrays at 70ms per character forward, 40ms per character backward, with 1500ms pause at full text and 500ms pause between roles.

---

## Content Sections

### LEBEN (Life)
- **Interessen (Interests):** Lernen, Sport, Motorräder, Psychologie, Philosophie, Religion
- **Geschichte (Story):** Immigration from Germany to Switzerland in early 2024

### ARBEIT (Work)
- **Timeline:** Career from apprenticeship (2017–2020) → IT Admin (2020–2024) → RMM & PSA Specialist (2025)
- **Zertifikate (Certificates):** Leadership training, time management courses
- **Projekte (Projects):**
  1. Automated Server Patch Management (N-Able N-sight RMM)
  2. NinjaOne Transformation (centralized scripting and monitoring)
  3. Self-Hosted API Dashboard (Python-based alert prioritization)

### KONTAKT (Contact)
- Email: `contact@marvininja.ch`
- Business phone: `+49 15679 652 219`
- Private phone: visually blurred with `.private-number` class

---

## Key Conventions for AI Assistants

1. **Single-file discipline** — All changes to logic, style, and structure go into `index.html`. Do not create separate `.css` or `.js` files unless explicitly asked.

2. **Bilingual parity** — Every piece of user-visible text must have both a `data-lang="de"` and a `data-lang="en"` version. Never add content in only one language.

3. **No framework introduction** — Do not introduce React, Vue, Tailwind, or any other framework/library. Keep the stack vanilla.

4. **No build tooling** — Do not add `package.json`, `vite.config.js`, or any build configuration without explicit instruction.

5. **Preserve the design language** — Use the established color palette and glassmorphism patterns. Do not introduce flat design, light mode, or conflicting visual styles.

6. **Respect the privacy blur** — The `.private-number` class intentionally blurs private contact info. Do not remove or work around it.

7. **Inline event handlers** — The codebase uses inline `onclick` handlers. Match this pattern for consistency unless refactoring is explicitly requested.

8. **Active favicon** — Only `img/favicons-1.png` is used. Do not change favicon references without good reason.

9. **`robots` meta tag** — The site uses `noindex, nofollow`. Do not remove this unless explicitly instructed.

10. **Commit messages** — Write clear, imperative commit messages describing what changed and why (e.g., `Add English translation for certificates section`).

---

## vCard Format

`vcards/marvin_business.vcf` follows vCard 3.0 spec. If contact details change in `index.html`, update the `.vcf` file to match.

```
VERSION:3.0
FN:Marvin
TEL;TYPE=BUSINESS:+49 15679 652 219
EMAIL:contact@marvininja.ch
URL:https://www.marvininja.ch
ADR:;;Zürich;;CH
```

---

## What Does Not Exist (and Should Not Be Added Without Discussion)

- Tests (no test framework, no test files)
- Linting / formatting config (no ESLint, Prettier, etc.)
- CI/CD pipelines
- Server-side code
- Database or API integrations
- Dark/light mode toggle (site is dark-only by design)
