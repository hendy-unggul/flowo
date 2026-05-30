# Flowo — Brand Kit v1.0

## Brand Identity

**Name:** Flowo  
**Tagline:** Work flows. Results follow.  
**Category:** Productivity / Project Management SaaS  
**Tone:** Confident, clean, builder-focused. Not corporate. Not playful. *Precise.*

---

## Logo

### The Mark
A 2×2 grid of offset squares with a thin bar below — representing structured flow, progress layers, and momentum. The offset creates visual dynamism without chaos.

### Clearspace
Minimum clearspace = height of one grid square on all sides.

### Minimum Size
- Digital: 24px height minimum
- Print: 8mm height minimum

### Don'ts
- Don't stretch or distort the mark
- Don't use on low-contrast backgrounds without adjustment
- Don't recolor individual elements
- Don't add drop shadows or effects

---

## Color Palette

### Primary
| Name         | Hex       | Usage                          |
|--------------|-----------|--------------------------------|
| Violet Core  | `#5B4FE8` | Primary CTA, active states     |
| Violet Light | `#7B6FFF` | Hover, secondary accent        |
| Violet Soft  | `#B8B4FF` | Text on dark, subtle accent    |

### Secondary
| Name         | Hex       | Usage                          |
|--------------|-----------|--------------------------------|
| Teal Flow    | `#38D9A9` | Success, progress, growth      |
| Teal Deep    | `#2BC89A` | Teal hover state               |

### Neutrals (Dark Theme — Primary)
| Name         | Hex       | Usage                          |
|--------------|-----------|--------------------------------|
| Void         | `#080810` | Page background                |
| Depth        | `#0A0A14` | Secondary background           |
| Surface      | `#0F0F1E` | Card / panel background        |
| Elevated     | `#141428` | Hover surface                  |
| Border       | `#1A1A2C` | Default border                 |
| Border+      | `#22223A` | Emphasized border              |

### Text (Dark Theme)
| Name         | Hex       | Usage                          |
|--------------|-----------|--------------------------------|
| Primary      | `#F2F0FF` | Headings, primary text         |
| Secondary    | `#8A87C0` | Body, descriptions             |
| Tertiary     | `#3D3A6A` | Labels, placeholders           |
| Hint         | `#2A2848` | Disabled, meta text            |

### Semantic
| Name    | Hex       | Usage        |
|---------|-----------|--------------|
| Warning | `#E8A030` | Overdue, alert |
| Danger  | `#E24B4A` | Error, critical |
| Info    | `#378ADD` | Informational  |

---

## Typography

### Display / Headings
- **Font:** Syne (Google Fonts)
- **Weights:** 700, 800
- **Usage:** Hero titles, section headings, feature names
- **Letter-spacing:** -0.03em to -0.05em (tight)

### Body / UI
- **Font:** DM Sans (Google Fonts)
- **Weights:** 300 (light body), 400 (regular), 500 (medium/label)
- **Usage:** Paragraphs, nav, buttons, captions
- **Line-height:** 1.7 for body, 1.05–1.15 for headings

### Monospace (Code/URLs)
- **Font:** system monospace stack
- **Usage:** URLs, code snippets, version numbers

### Scale
| Token     | Size   | Weight | Use                  |
|-----------|--------|--------|----------------------|
| `hero`    | 64px+  | 800    | Landing hero         |
| `h1`      | 48px   | 700    | Page titles          |
| `h2`      | 32px   | 700    | Section headings     |
| `h3`      | 22px   | 600    | Card/panel titles    |
| `body-lg` | 18px   | 300    | Lead paragraphs      |
| `body`    | 16px   | 400    | Standard body        |
| `small`   | 13px   | 400    | Captions, meta       |
| `label`   | 11px   | 500    | Tags, UI labels      |
| `micro`   | 9–10px | 500    | Badges, eyebrow text |

---

## Spacing System (8px base)

```
4px   — micro (icon gap, tight pairs)
8px   — xs
12px  — sm
16px  — md (default component padding)
24px  — lg
32px  — xl
48px  — 2xl
64px  — 3xl
96px  — 4xl (section padding)
```

---

## Border Radius

| Token  | Value  | Use                    |
|--------|--------|------------------------|
| `sm`   | 6px    | Chips, small badges    |
| `md`   | 8px    | Buttons, inputs        |
| `lg`   | 12px   | Cards, panels          |
| `xl`   | 16px   | Large cards            |
| `2xl`  | 24px   | Containers, modals     |
| `full` | 9999px | Pills, avatar          |

---

## Logo Color Variants

### On Dark Background
- Mark: Violet Core `#5B4FE8` + Violet Light `#7B6FFF` + Teal `#38D9A9`
- Wordmark: `#F2F0FF`
- Accent letter "o": `#7B6FFF`

### On Light Background
- Mark: `#3D35C4` + `#5448E0` + `#1BA882`
- Wordmark: `#1A1A2E`
- Accent letter "o": `#3D35C4`

### Monochrome Dark
- All: `#F2F0FF`

### Monochrome Light
- All: `#1A1A2E`

---

## Motion

### Principles
- Purposeful: animate to guide attention, not decorate
- Fast in, slow out: enter 300ms ease-out, exit 200ms ease-in
- Stagger reveals: 60ms delay between sequential elements

### Tokens
```css
--ease-out:   cubic-bezier(0.16, 1, 0.3, 1);
--ease-in:    cubic-bezier(0.4, 0, 1, 1);
--ease-both:  cubic-bezier(0.16, 1, 0.3, 1);
--dur-fast:   150ms;
--dur-base:   250ms;
--dur-slow:   400ms;
--dur-enter:  600ms;
```

---

## Voice & Tone

| Context     | Tone                              | Example                              |
|-------------|-----------------------------------|--------------------------------------|
| Hero        | Bold, declarative                 | "Work flows. Results follow."        |
| Feature     | Direct, benefit-first             | "Sprint boards that don't slow you down." |
| CTA         | Action, zero friction             | "Start for free →"                   |
| Error       | Calm, helpful                     | "Something went wrong. Try again."   |
| Empty state | Encouraging, human                | "No tasks yet — add your first one." |

---

## File Deliverables (this kit)

- `BRAND_KIT.md` — this document
- `index.html` — production landing page (deploy to Vercel)
- `vercel.json` — Vercel config
- `favicon.svg` — browser favicon

---

*Flowo Brand Kit v1.0 — 2025*
