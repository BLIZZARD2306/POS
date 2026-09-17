# POS

A Claude Code plugin that turns "build me a POS system" into a repeatable recipe — one prompt, a full point-of-sale app.

## What this is

This repo packages a **skill** (`skills/pos/SKILL.md`) that Claude Code loads whenever a request involves designing, building, scaffolding, or extending a Point of Sale system for any kind of shop — sari-sari store, milk tea shop, café, restaurant, retail, salon, etc.

Instead of generating a generic, default-blue-and-white CRUD app every time, the skill encodes:

- **A proven architecture** — React 18 + TypeScript + Vite + Tailwind v4, with a shared data model (`Product`, `CartItem`, `Sale`, `Customer`) and five standard modules: Cashier, Inventory, Analytics, Credit/Tab, and About/Help.
- **POS Kit**, a small design system with three distinct visual "looks" (Receipt, Market, Night Shift) so every build has real personality instead of the same default template — while keeping component code identical across looks via semantic design tokens.
- **A build checklist** covering payments, stock deduction, low-stock thresholds, scanning fallbacks, and per-look layout recipes for both desktop and mobile.

The skill is self-contained: everything needed to build a POS from scratch is in `SKILL.md`, including a full reference build (a Filipino sari-sari store POS) that new builds are adapted from rather than copied.

## Structure

```
.claude-plugin/
  plugin.json       # Plugin manifest — name, description, and trigger phrases
skills/
  pos/
    SKILL.md        # The full skill: architecture, data model, design system, build checklist
```

## Using it

Install this as a Claude Code plugin, then just ask Claude Skill to build a POS for your business (e.g. "build me a POS for my milk tea shop that lets regulars pay on credit"). Claude will:

1. Ask a few quick questions about the business if needed (menu categories, credit/tab support, item variants, payment methods).
2. Pick one of the three POS Kit looks (Receipt, Market, or Night Shift) based on the business type, or match your brand colors.
3. Build the five core modules using the shared architecture and data model.
4. Check the result against the build checklist before handing it back.

## License

No license specified yet.
