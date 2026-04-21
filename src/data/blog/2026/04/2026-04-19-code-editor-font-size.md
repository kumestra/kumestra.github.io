---
title: Choosing the Right Default Font Size for a Code Editor
description: A practical guide to selecting a sensible default font size for a code editor, considering screen type, industry norms, and user demographics.
pubDatetime: 2026-04-19T10:06:19Z
tags: [code-editor, font-size, ui-design, developer-tools, typography]
---

## 🖋️ Why Default Font Size Matters

When building a code editor, the default font size shapes the first impression for every new user. Choose too small and text strains the eyes; choose too large and it feels like a toy. Getting this right requires knowing your audience's hardware.

---

## 📊 Industry Defaults

Most established editors lean conservative — prioritizing information density over comfort:

| Editor        | Default Font Size |
|---------------|:-----------------:|
| VS Code       | 13px              |
| JetBrains IDEs| 13px              |
| Sublime Text  | ~13px (10pt)      |
| Zed           | 15px              |

The industry norm clusters around **13–14px**, tuned for modern high-DPI and Retina displays where pixels are dense and small text still looks sharp.

---

## 🖥️ Screen Type Changes Everything

Font size perception varies significantly by display technology:

- **High-DPI / Retina screens** — 14px renders crisply; text is comfortable at smaller sizes.
- **Low-DPI / old screens** — pixels are larger and less dense; 13–14px can feel cramped or blurry.

If your users are primarily on **older or low-resolution monitors**, the standard defaults will feel too small. The rendered characters are physically larger per pixel, but anti-aliasing and sub-pixel rendering are limited, making small text harder to read.

---

## ✅ Recommended Default

| Audience                        | Recommended Default |
|----------------------------------|:-------------------:|
| General / mixed screen types     | **14–15px**         |
| Primarily high-DPI screens       | **13–14px**         |
| Primarily old / low-DPI screens  | **16px**            |

> For users on old and low-resolution screens, **16px** is the sweet spot — readable without feeling oversized, and clearly legible even without sharp pixel rendering.

---

## 🔧 Best Practice: Make It Adjustable

No single default satisfies every user. Always expose font size as a configurable setting so users can dial it up or down to suit their screen, vision, and preference. The default is just the starting point.

```json
{
  "editor.fontSize": 16
}
```

This pattern — a sensible default with easy override — is the standard across all major editors and keeps accessibility within reach for every user.
