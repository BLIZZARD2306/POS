---
name: pos
description: "Use whenever the user wants to design, build, scaffold or extend a Point of Sale (POS) system for any shop (sari-sari store, milk tea shop, cafe, restaurant, retail, salon). Trigger on \"POS\", \"point of sale\", \"cashier system\", or \"inventory + sales system\"."
---

# POS Skill

This skill turns a proven POS architecture (a Filipino sari-sari store POS) plus the **POS Kit design system** into a repeatable recipe. Every new POS reuses the same modules and data model, and picks one of three distinct looks instead of the old default blue-and-white shadcn style.

If a `references/examples/` folder exists next to this file, check it for more reference builds before starting. Everything required to build is in this file, so the skill works without that folder.

## Workflow

1. **Read this whole file first.** If `references/examples/sari-sari-store-pos/` exists, skim its `OVERVIEW.md` and `.tsx` files for real code patterns (state shape, localStorage use). If other example folders exist, use whichever is closest to the new business.
2. **Ask briefly about the business** if the user hasn't said already:
   - product or menu categories
   - whether the shop sells on credit ("utang" or a tab)
   - whether items have sizes, add-ons or options (drink size, sugar level, toppings)
   - which payments they take besides cash (GCash, Maya, card)
3. **Pick a look** from POS Kit (see "Choosing a look"). If the user doesn't care, use the default for their business type. Say which look you picked and why in one line. Never fall back to the original blue `#0052CC` on white unless the user asks for the sari-sari look.
4. **Build** with the universal architecture and the POS Kit components. Adapt the modules, categories and wording to the business. Don't copy the sari-sari example word for word.
5. **Check the build** against "Build checklist" at the bottom.
6. **Offer to save it as a reference.** After finishing, tell the user they can ask to add the build under `references/examples/`. Only add it when they ask.

---

## Universal architecture

Keep this for every POS, whatever the business type.

- **Stack:** React 18, TypeScript, Vite and Tailwind CSS v4, with Radix UI/shadcn-style primitives (restyled with POS Kit tokens), Recharts for charts, lucide-react for icons and Sonner for toasts. It runs in the browser only; add a backend only if the user asks.
  - If the user wants a single-file artifact instead, build the same modules in one React or HTML file. Keep state in memory unless the environment supports localStorage.
- **Persistence:** use `localStorage` with a versioned schema key (e.g. `<app>-data-version`) so seed data can be migrated or reset safely. Keep separate keys for products, sales and customers.
- **Data model.** Adapt names, keep the shape:
  - `Product`: `id, name, price, stock, cost?, category?, image?, maxStock?, unit?`, plus `variants?: {label, priceDelta}[]`, `addOns?: {label, price}[]` and `choices?: {label, options: string[]}[]` for food and drink shops.
  - `CartItem`: `productId, qty, variant?, addOns?, choices?, unitPrice`.
  - `Sale`: `id, items[], total, date, paymentMethod ('cash' | 'wallet' | 'credit'), walletProvider?, reference?, paymentAmount?, change?, customerId?, customerName?`.
  - `Customer` (credit businesses only): `id, name, phone?, totalBalance, transactions[]`. Each transaction is `{id, date, items, amount, remaining}`. Payments settle the oldest open transaction first (FIFO).
- **Modules** (tabs). Keep all five unless the user drops one:
  1. **Cashier:** searchable product grid, category filters, cart with quantity editing, scan/lookup (QR or barcode via `html5-qrcode`, with manual ID entry as a fallback), payment methods and automatic stock deduction on sale.
     - Cash with change calculation is mandatory.
     - Add e-wallet and/or credit where relevant.
  2. **Inventory:** full CRUD, image upload or image URL, low-stock alerts, categories and units.
  3. **Analytics:** week/month toggle, revenue and profit (from cost vs price), top 5 sellers, stock-out prediction from sales speed, a trend chart and payment mix.
  4. **Credit / Tab (optional):** per-customer balance, itemised history, partial payments using FIFO.
     - Include it only when the business really lets regulars buy on credit. Sari-sari stores do; milk tea and fast-casual shops usually don't.
  5. **About / Help:** a plain-language explainer, the accepted payment methods, and where data is stored.
- **Layout:** desktop uses a top navigation bar (or a side rail in the Night Shift look). Mobile uses a bottom tab bar, a single column and touch-sized controls.

### Reference build: sari-sari store POS (summary)

- **Cashier:**
  - product grid with search and categories: Canned Foods, Instant Noodles, Snacks, Drinks, Fresh Items, Personal Care, Household Items, Other
  - cart with quantity editing and line removal
  - camera QR/barcode scan (front/rear camera, error when there's no match or the item is out of stock)
  - manual product ID entry, because camera scanning fails inside preview iframes and needs HTTPS
  - Cash (amount received and change), GCash (recorded at the exact total) and Utang (charged to a chosen customer)
  - stock deducted on every sale
- **Storage:**
  - CRUD with name, price, cost, stock, max stock, category and image
  - low-stock flag
  - live stock updates
- **Analytics:**
  - weekly/monthly views, revenue and profit, top 5 by revenue
  - stock-out prediction (stock ÷ average daily units)
  - line and bar charts
  - transaction history with payment badges
- **Utang:**
  - customer directory (name, optional phone) with a running balance
  - itemised transactions showing the remaining amount on each
  - FIFO partial payments
  - summary cards: customers, total outstanding, active balances
  - removing a customer is blocked while they still owe money
- **Data:** three localStorage keys plus a versioned seed catalogue of 60+ items. There's no auth and no sync, and the app runs on one device only.

## What must change per business type

- **Categories and menu structure.** Example for milk tea: Milk Tea, Fruit Tea, Coffee, Snacks, Seasonal.
- **Customisation model.** Food and drink items carry `variants` / `addOns` / `choices` on one menu item. Never create a separate product per size.
- **Credit feature.** Include it only if it matches how the business really works.
- **Payment methods.** Always keep cash. Add or remove e-wallets and card to suit the business and region.
- **Units and low-stock rules.** Use pcs, ml, g or cups as fits.
  - The default low-stock rule is `stock <= maxStock * 0.3`; tune it per business.
- **Branding.** Choose the POS Kit look, set the store name, and write the copy.

---

# POS Kit — design system

One component anatomy, three looks. Components only read **semantic tokens**, so switching the look never changes component code.

## Choosing a look

| Look | Feel | Default for | Fonts (Google Fonts) |
|---|---|---|---|
| **Receipt** | Thermal-paper calm: monospace text, hard black rules, red "stamp" accents, square corners | Sari-sari stores, convenience stores, hardware, pharmacies | Anton (display) + IBM Plex Mono (body and numbers) |
| **Market** | Loud, friendly stall energy: cream ground, tomato red and sun yellow, chunky 2px outlines, pill buttons, big tiles | Milk tea, cafés, food stalls, bakeries, salons | Bricolage Grotesque (display) + Figtree (body) |
| **Night Shift** | Dark, dense and keyboard-first: lime accent, 1px lines, monospace numbers, F-key shortcuts | Minimarts, bars, high-volume retail tills | Chakra Petch (display) + Manrope (body) + JetBrains Mono (numbers) |

**How to pick:**
- If the user names brand colours, keep that look's *structure* (shapes, layout, type roles) and swap in their colours.
- Never mix two looks in one app.

## Tokens

Put these in `src/styles/theme.css` and set `data-look="receipt|market|night"` on `<html>`. With Tailwind v4, map them in `@theme inline` so classes like `bg-surface`, `text-ink` and `border-line` work.

```css
@import url("https://fonts.googleapis.com/css2?family=Anton&family=IBM+Plex+Mono:wght@400;500;600;700&family=Bricolage+Grotesque:opsz,wght@12..96,600;12..96,800&family=Figtree:wght@400;500;600;700;800&family=Chakra+Petch:wght@500;600;700&family=Manrope:wght@400;500;600;700;800&family=JetBrains+Mono:wght@400;600;700&display=swap");

:root, [data-look="market"] {
  --ground:#FFF3DE; --surface:#FFFFFF; --raised:#FFF9EE; --soft:#FFE3B0;
  --ink:#221A14; --muted:#62544A; --line:#221A14;
  --accent:#C9381A; --accent-ink:#FFFFFF; --accent-soft:#FFE0D6;
  --sun:#F4B400; --sun-ink:#221A14;
  --ok:#1E7A46; --ok-soft:#D8F0E0; --warn:#9A5000; --warn-soft:#FFE8C7;
  --danger:#B3261E; --danger-soft:#FBDAD7;
  --credit:#6B3FA0; --credit-soft:#EADCF7; --wallet:#0B6399; --wallet-soft:#D6ECFA;
  --font-display:"Bricolage Grotesque","Trebuchet MS",sans-serif;
  --font-body:"Figtree","Segoe UI",sans-serif; --font-num:"Figtree","Segoe UI",sans-serif;
  --r:14px; --r-lg:24px; --bd:2px solid #221A14; --bd-soft:2px solid #F1DDBA;
  --shadow:0 4px 0 #221A14; --display-case:none; --display-track:-0.02em;
}
[data-look="receipt"] {
  --ground:#F3EEE3; --surface:#FFFDF7; --raised:#FFFDF7; --soft:#E7E0D0;
  --ink:#1B1A17; --muted:#5E5A50; --line:#1B1A17;
  --accent:#C8321E; --accent-ink:#FFFFFF; --accent-soft:#F6D9D2;
  --sun:#1B1A17; --sun-ink:#FFFDF7;
  --ok:#2F6B3A; --ok-soft:#DDEBDD; --warn:#8F4E00; --warn-soft:#F5E3C8;
  --danger:#B3261E; --danger-soft:#F4D6D3;
  --credit:#3B4FA0; --credit-soft:#DCE0F2; --wallet:#1B6680; --wallet-soft:#D5E9EF;
  --font-display:"Anton",Impact,"Arial Narrow",sans-serif;
  --font-body:"IBM Plex Mono",ui-monospace,Menlo,monospace; --font-num:var(--font-body);
  --r:2px; --r-lg:4px; --bd:1.5px solid #1B1A17; --bd-soft:1px dashed #8C8676;
  --shadow:4px 4px 0 #1B1A17; --display-case:uppercase; --display-track:0.02em;
}
[data-look="night"] {
  --ground:#0E1311; --surface:#161D1A; --raised:#1E2723; --soft:#1E2723;
  --ink:#E8EFEA; --muted:#9AA8A0; --line:#2C3833;
  --accent:#C6F432; --accent-ink:#0E1311; --accent-soft:#2A3616;
  --sun:#C6F432; --sun-ink:#0E1311;
  --ok:#5BD68A; --ok-soft:#16301F; --warn:#F5B545; --warn-soft:#33280F;
  --danger:#FF7A6E; --danger-soft:#3A1A17;
  --credit:#B79CFF; --credit-soft:#271F3D; --wallet:#5CC8FF; --wallet-soft:#0F2A38;
  --font-display:"Chakra Petch","Segoe UI",sans-serif;
  --font-body:"Manrope","Segoe UI",sans-serif; --font-num:"JetBrains Mono",ui-monospace,monospace;
  --r:6px; --r-lg:10px; --bd:1px solid #2C3833; --bd-soft:1px solid #242E2A;
  --shadow:none; --display-case:uppercase; --display-track:0.04em;
}
body { background: var(--ground); color: var(--ink); font-family: var(--font-body); }
.num { font-family: var(--font-num); font-variant-numeric: tabular-nums; }
.display { font-family: var(--font-display); text-transform: var(--display-case); letter-spacing: var(--display-track); font-weight: 800; }
```

**Token roles:**

| Token | Used for |
|---|---|
| `ground` | page background |
| `surface` | cards, sheets |
| `raised` | inputs, wells, keypad keys |
| `soft` | image placeholders, meter tracks |
| `ink` | text and strokes; also the fill of an *active* chip or segment, with `ground` as its text |
| `muted` | secondary text |
| `accent` | the ONE primary action per view, and the active tab |
| `sun` | highlight button (Scan), badges |
| `ok` | in stock, paid, change, cash |
| `warn` | low stock |
| `danger` | out of stock, delete, "short by" |
| `credit` | credit / utang everywhere |
| `wallet` | e-wallet everywhere |
| `*-soft` | background behind the matching coloured text |

**Shared scale:**
- Spacing is 4-based: 4, 8, 12, 16, 24, 32, 48, 64.
- Touch targets are **44px minimum**. Money actions (Complete sale, Charge, Save payment) are **56–68px** and full width on phones.
- Type sizes:

| Style | Size |
|---|---|
| Display | 44–104px |
| Heading | 22–28px |
| Body | 15–16px |
| Caption | 12–13px |
| Totals | 36–64px `.num` |

- Breakpoints: 390 phone, 768 tablet, 1280+ desktop.

## Shared rules

1. **Cash is always there.** E-wallet and credit are optional per business.
2. **Stock has three states** with the same colours everywhere (tiles, rows, meters, badges):
   - **In stock** (`ok`)
   - **Low** (`warn`): `stock <= maxStock * 0.3`
   - **Out** (`danger`)
   - Out-of-stock items stay visible but disabled (opacity .55, no add).
3. **Money uses tabular figures** (`.num`). Format with `₱` + `toLocaleString('en-PH', {minimumFractionDigits: 2})`, or the local currency.
4. **Payment colours are fixed by meaning:** cash = `ok`, e-wallet = `wallet`, credit = `credit`. Use them in the cashier, analytics badges, receipts and toasts.
5. **One primary (accent) button per view.** A disabled primary button says *why* ("Not enough cash", "Pick a customer", "Add items first").
6. **Icons:** lucide-react only, stroke style. **No emoji in the UI** (the old build used emoji tab icons; replace them). Mapping:

| Area | Icon |
|---|---|
| Cashier | `ShoppingCart` |
| Inventory | `Package` |
| Analytics | `LineChart` |
| Credit | `BookOpen` |
| About | `Info` |
| Scan | `ScanLine` |
| Camera | `Camera` |
| Cash | `Banknote` |
| E-wallet | `Wallet` |
| Low stock | `AlertTriangle` |
| Delete | `Trash2` |
| Credit guard | `Lock` |
| Days-left | `Clock` |
| Drink item | `CupSoda` |

7. **Accessibility:**
   - Use real `<button>` / `<a>` / `<input>` + `<label>`, and `aria-label` on icon-only buttons.
   - Use `role="radiogroup"` for the payment switch.
   - Use `role="alert"` for scan errors and `role="status"` for toasts.
   - Text contrast must be at least 4.5:1.
8. **Avoid:** gradient washes, cards with a coloured left border, Inter/Roboto/Arial, the default shadcn blue, invented stats.
   - Use placeholders like `[STORE NAME]` for facts you don't have.
   - Seed or demo data is fine but must look realistic for the business.

## Components

Codes let you refer to parts precisely. Every component uses only the tokens above.

### Cashier (C)

- **C-01 Product tile.** A `<button>` containing:
  - an image, or a `soft` placeholder with an icon, 116–128px tall
  - category (caption, `muted`)
  - name (bold, 2-line min-height)
  - price (`.num`, 18–20px, bold)
  - stock badge (`ok`: "42 left"; `warn`: `AlertTriangle` + "6 left"; `danger`: "Out of stock")

  States:
  - **default**
  - **low**
  - **out:** disabled, opacity .55
  - **in cart:** 3px `accent` outline with 3px offset, plus a round qty bubble ("×2") at the top-right

  Card styling: `surface`, `--bd`, `--r-lg`, `--shadow`.
- **C-02 Category chips.** Pills 44px tall, wrapping. The active chip is filled with `ink` and uses `ground` text (`aria-pressed`). Inactive chips use `surface` with `--bd-soft`.
- **C-03 Search with scan trigger.** A 52px input in `raised` with a search icon and a "/" shortcut hint, plus a square 52px **Scan** button in `sun`.
- **C-04 Scan panel.**
  - header "Scan to add" with a Rear/Front camera segment
  - dark viewfinder (#111412, ~230px) with a dashed target box, a `sun` scan line, and the hint "Point at a barcode or QR code"
  - manual fallback: field "No camera? Type the product ID" plus an **Add** button
  - error box (`danger-soft` / `danger`, `role="alert"`): "No product matches 4800016. Check the code, or add it in Inventory. Camera scanning needs HTTPS."
- **C-05 Cart line and totals.** Each row has:
  - name, with "₱16.00 each" as a caption
  - stepper: [−] qty [+], each 44×44, inside a `--bd` box
  - line total (`.num`, right-aligned)
  - trash icon button

  Under the rows: an "Items" count and **Total** (display label, 36px `.num` value). There's a "Clear" text button in `danger`. The empty state reads "No items yet. Tap a product or scan a code to start a sale."
  - Food and drink shops: show the modifiers under the name in `muted` ("Large · 50% sugar · Pearls").
- **C-06 Payment method switch.** A `radiogroup` of 3 buttons, each 72px tall: Cash ("With change"), E-wallet ("GCash · Maya"), Credit ("Utang / tab"). Each has a coloured dot (ok / wallet / credit). The selected button is filled with `ink` and uses `ground` text. Hide the options the business doesn't use.
- **C-07 Tender panel.** It changes with the method:
  - **Cash:**
    - an "Amount received" display (`raised`, 40px `.num`)
    - quick chips: Exact, ₱100, ₱200, ₱500, ₱1,000
    - a 3×4 keypad (1–9, 00, 0, ⌫), keys 56px tall
    - a Change row (`ok-soft`/`ok`), or "Short by ₱X" (`danger-soft`/`danger`) when there isn't enough cash
  - **E-wallet:**
    - provider pills (GCash / Maya / Card), with the active pill in `wallet`
    - a `wallet-soft` box with a `[STORE QR]` placeholder, "Recorded at the exact total", the total, and "No change is given for e-wallet sales"
    - an optional "Reference no." field
  - **Credit:**
    - "Charge to customer": a radio list of customers, each 56px tall, with a `credit` initials avatar, name, and "owes ₱X"; the selected one uses `credit-soft` with a 2px `credit` border
    - a "+ New customer" button
    - a `credit-soft` note: "Adds ₱X to Name's balance → ₱Y"
  - **Complete button:** 60px, display font, `accent`, with the label "Complete sale · ₱X". When disabled it uses `soft`/`muted` and gives the reason.
- **C-08 Receipt summary.**
  - `[STORE NAME]` with address and date
  - dashed rules
  - "qty × name … amount" lines
  - TOTAL, "Paid via", Received/Change (cash) or Customer (credit)
  - "Thank you · Salamat!"
  - then a success status ("Sale saved · stock updated for N products") and the buttons **New sale** (primary) and **Print**

### Inventory (I)

- **I-01 Stock meter.** A label and "42 / 60 pcs" above an 8px rounded track (`soft`) with a fill in `ok` / `warn` / `danger`. Caption: "Low = stock ≤ 30% of max stock."
- **I-02 Inventory row.**
  - Desktop table columns: Product (48px thumb + name + category) | Unit | Cost | Price | Margin (`ok`) | Stock meter | Edit + Delete icon buttons (44px)
  - Filter pills above the table: All n / Low n / Out n, plus an **Add product** primary button
  - Phone: stacked cards with the same fields
- **I-03 Low-stock banner and counts.**
  - `warn-soft` banner with a round `warn` alert icon: "5 products are running low", naming 2 items "and N more", with a **Show low stock** button
  - a row of 4 stat boxes: Products, Low stock (`warn`), Out of stock (`danger`), Stock value at cost
- **I-04 Product form** (add and edit share it):
  - a 150px dashed photo drop zone ("Add photo · Upload or paste an image URL")
  - fields: Name; Category (select); Unit (select: pcs, g, ml, cups); Cost (₱); Selling price (₱); Stock on hand; Max stock
  - a live strip: "Margin ₱8.00 · 19%" and "Low-stock alert at 9 pcs"
  - buttons: Cancel (ghost) and **Save product**
- **I-05 Sizes, add-ons and choices** (food and drink only; omit for single-SKU retail):
  - **Sizes:** label + price delta rows ("Large 22 oz" / "+ ₱15")
  - **Add-ons:** label + price rows ("Pearls" / "₱10")
  - "+ Add option" dashed button
  - **Choices** with no price: pills, e.g. sugar level 0 / 25 / 50 / 75 / 100%, with the selected pill in `sun`
- **I-06 Delete confirm.**
  - a `dialog` with a `danger-soft` trash icon
  - text: "Delete Iced tea 1 L?" / "It disappears from the cashier. Past sales keep their record. This can't be undone."
  - buttons: Keep it / **Delete** (`danger`)

### Analytics (A)

- **A-01 Period switch.** Tabs "This week" / "This month"; the active tab is filled with `ink`.
- **A-02 KPI tiles** (4): Revenue, Profit ("22% margin"), Transactions, Average sale. Each shows a 34px `.num` value and a change line (▲ `ok` / ▼ `danger`).
- **A-03 Trend chart.** Recharts `LineChart`, at most 2 series:
  - Revenue: `accent`, 3.5px, with dots
  - Profit: `ok`, 3px
  - gridlines in `soft`, axis text in `muted` `.num`
  - label the peak value, and add a legend at the top-right
- **A-04 Top 5 sellers.** Horizontal bars (14px) with rank + name + value. The leader uses `accent`; the others use `ink` at 75%.
- **A-05 Stock-out prediction.** Cards with:
  - a days-left badge (`Clock`; `danger` for ≤2 days, `warn` for ≤5)
  - "~3 sold / day", the name, "6 left", and a **Restock** button
  - formula note: "Days left = stock on hand ÷ average units sold per day over the chosen period. Items with no sales are skipped."
- **A-06 Payment mix and history.**
  - a stacked bar (cash `ok`, e-wallet `wallet`, credit `credit`) with a legend
  - transaction rows: time | "3 items" (+ customer name for credit) | method badge | amount

### Credit (K), optional

- **K-01 Summary cards** (3): Customers, Total outstanding (`credit`), With a balance. Each has a round `credit-soft` icon.
- **K-02 Customer card.**
  - initials avatar (`credit`), name, phone (masked), Balance (30px `.num`)
  - when the customer owes money:
    - "Oldest open: Sep 05 · 12 days"
    - buttons **Record payment** + Ledger
    - the remove button is a disabled `Lock` icon (aria: "Can't remove: still has a balance")
  - when paid up: "All paid · safe to remove" (`ok`), with Ledger + **Remove** (`danger`)
- **K-03 FIFO ledger.**
  - header: "Name · ledger" / "Oldest first. Payments settle the oldest open entry first." with the balance at the right
  - columns: Date | Items | Amount | Left | Status
  - status badges: Paid `ok` / Part paid `warn` / Open `credit`
  - paid rows stay for history
- **K-04 Record payment.**
  - "Amount paid" display, quick chips ("Pay full ₱320", ₱100, ₱200)
  - **"Where it goes" preview** listing each open entry, the amount applied, and the result ("Cleared" / "₱120 left")
  - "New balance", then **Save payment** (56px)
- **K-05 Add customer.** Name, Phone (optional), **Save customer**.

### Shell (N)

- **N-01 Desktop top nav.**
  - brand mark (`sun` square) + `[STORE NAME]` in the display font
  - tabs with icons, 48px tall; the active tab uses `accent` with `aria-current="page"`
  - on the right: "Saved on this device · data vN"
- **N-02 Mobile tab bar.**
  - 5 tabs (4 without Credit), each ≥58px, icon above label; the active one is filled with `accent`
  - a sticky cart bar sits above it (`ink` fill): "3 items · ₱105.00 · Review"
- **N-03 Buttons.**
  - Kinds: primary (`accent`), sun (Scan), secondary (`surface` + `--bd`), ghost, danger, and disabled (`soft`/`muted`, with a reason)
  - Sizes: 44 default, 56 for money actions
- **N-04 Toasts** (Sonner, styled with `surface` + `--bd` + `--shadow`, and a round soft-colour icon):

| Event | Title | Detail |
|---|---|---|
| Sale done (`ok`) | "Sale completed · ₱X" | "Change ₱Y · stock updated" |
| Out of stock (`danger`) | "Item is out of stock" | |
| Camera needs HTTPS (`wallet`) | "Camera needs a secure link" | "Use the product ID for now." |
| Credit (`credit`) | "₱X added to Name" | "New balance ₱Y" |

  - Placement: bottom-right on desktop, top on phone.
- **N-05 Empty states.**
  - Layout: a dashed box with a round `soft` icon, a title, one line saying what to do next, and a button.
  - Copy:
    - "No products yet" / "No customers on credit" / "Not enough sales yet"
- **N-06 Dialog.**
  - header with a close button, body text, and a footer in `raised` with a secondary and a primary/danger action
  - desktop: centred dialog; phone: bottom sheet with the same parts
  - always confirm destructive actions ("Clear this sale?", "Delete product?")
- **N-07 About / Help.**
  - title, plain explanation ("runs in your browser; saved on this device only")
  - "We accept" badges (Cash / GCash-Maya / Credit)
  - warning: "Clearing browser data erases sales and products. Export a backup first."
  - support contact placeholder

## Layout recipes (make each look *structurally* different)

The looks are not just colour swaps. Use the layout that matches the chosen look.

### Receipt: desktop cashier

- **Header:** a 64px bar with a bottom rule, an uppercase Anton store name, text tabs with a 3px `accent` underline on the active one, and "DATA VN · SAVED ON THIS DEVICE" at the right.
- **Left:** a 210px category rail (uppercase list with counts; the active item has a 4px `accent` left bar).
- **Centre:**
  - a big 72px **scan/ID input** ("SCAN OR TYPE ID / NAME", "ENTER ↵ ADDS"), with a hard shadow
  - below it, a **ledger list** of products instead of tiles: `ID | NAME (uppercase) | stock (LOW/OUT coloured) | price | [+]` rows, 52px each, divided by dashed rules
  - out-of-stock rows are struck through
- **Right** (440px, darker paper #E9E2D3): the cart as a **paper receipt**:
  - "SALE #[SALE NO.]", dashed rules, lines with small steppers
  - Anton TOTAL
  - payment as three **stamp buttons** (outlined in `accent`; the selected one is filled and rotated −2°)
  - RECEIVED / CHANGE
  - a black **CLOSE SALE · ₱X** button
- **Phone:** stacked receipt cards, an uppercase Anton header, and a tab bar with a `--bd` top rule. The active tab is filled with `ink`.

### Market: desktop cashier (food and drink)

- **Header:** a rounded `sun` pill containing the shop name, pill tabs (the active one white with an outline) and a search pill.
- **Main:**
  - a big friendly heading ("What are they having?")
  - large category chips (52px; the active one in `accent`)
  - a **3-column grid of big tiles** with 24px radius and a hard 4px bottom shadow
  - each tile shows "from ₱95" and "2 sizes"
  - the selected tile gets a 4px `accent` ring
- **Middle column** (330px): a **customise card** for the selected drink:
  - Size segment (Regular / Large +₱15)
  - Sugar pills (0–100)
  - Add-on checkboxes with prices
  - **Add to order · ₱X** (60px `accent` pill)
- **Right column** (350px, dark `ink` panel):
  - "Order #[ORDER NO.] · Dine-in"
  - a cream inner card with lines ("1×", name, modifiers)
  - a big Total
  - payment segment (Cash / GCash / Maya)
  - "Received … Change …" in `sun`
  - a huge **Charge ₱X** pill in `sun`
- **Phone:**
  - a 2-column tile grid with horizontally scrolling chips
  - a sticky dark pill ("3 · View order · ₱320")
  - 4 tabs: Order, Menu, Sales, About (no credit)

### Night Shift: desktop register

- **Left:** an 84px **icon rail** (Sell, Stock, Stats, Tabs, Help; the active item filled with lime).
- **Centre:**
  - "REGISTER 01" in Chakra Petch, with `[STORE] · [CASHIER]` beside it
  - a 64px **command bar** with a lime border and glow: "Scan or type SKU / name", key hints `ENTER` add, `F1` camera
  - below it, the **cart as a dense table**, 52px rows: `# | SKU | Item (+LOW badge) | qty stepper | price | subtotal`
  - the selected row is `raised` with a 3px lime inset bar
  - the table footer shows "2 items in this sale are running low" and the keyboard hints "DEL removes line · ↑↓ select · +/− qty"
- **Right** (400px, `surface`):
  - "TOTAL · 15 ITEMS" with a 64px lime `.num` total
  - payment list with F-key chips: `F2` Cash, `F3` E-wallet, `F4` Credit
  - Received display, a 3×4 numpad (52px keys), and a Change line
  - a lime **`F9` COMPLETE SALE** button
- **Footer** (40px status bar, mono): a green dot and "Saved on this device · data vN", today's sales count, "5 low · 2 out", and the date/time.
- **Keyboard:** bind the shortcuts in a focused register component (not global handlers that break inputs). Enter adds, F2–F4 pick payment, F9 completes, Del removes the line.
- **Phone:** a Stats screen with a segmented Week/Month control, a 2×2 KPI grid, a sparkline, and a "Running out soon" list with `~2 DAYS` badges.

### Other modules per look

Keep each look's traits consistent across all tabs:
- **Receipt:** uppercase labels, dashed dividers, ledger-style tables, stamp-like status tags.
- **Market:** rounded cards, big pills, friendly sentence-case copy, thick outlines.
- **Night Shift:** dense tables, mono numbers, 1px lines, keyboard hints, dark surfaces.

## Implementation notes

- Put look switching in `App.tsx`: `document.documentElement.dataset.look = settings.look`. Optionally expose a picker in About/Settings. Save the choice in localStorage.
- Restyle the shadcn primitives by pointing their CSS variables at the POS Kit tokens (`--primary: var(--accent)`, `--radius: var(--r)`, etc.). Don't leave the default slate/blue.
- Recharts colours: pass `var(--accent)`, `var(--ok)`, `var(--wallet)` and `var(--credit)` via `stroke` / `fill`, with gridlines in `var(--soft)`.
- The scan viewfinder always stays dark, whatever the look.
- Keep modals as `Dialog` on desktop and `Drawer` (bottom sheet) on mobile.

## Build checklist

- [ ] One POS Kit look is chosen, set with `data-look`, and all colours come from tokens (no stray hex, no default blue).
- [ ] The fonts from the table load. No Inter/Roboto/Arial as the main face.
- [ ] Five modules are present, or Credit is removed on purpose, with the tabs updated.
- [ ] Cash with change works. The other payment methods match the business.
- [ ] Stock deducts on sale. Low (≤30% of max) and Out states look the same everywhere.
- [ ] Scan works on HTTPS, with manual ID entry as the fallback.
- [ ] Food and drink businesses use variants / add-ons / choices on the menu item.
- [ ] Credit (if present): FIFO settlement, a payment preview, and removal blocked while there's a balance.
- [ ] Analytics: week/month, revenue + profit, top 5, stock-out days, payment mix.
- [ ] Touch targets ≥44px, money buttons ≥56px, no emoji icons, labels on inputs, aria-labels on icon buttons.
- [ ] localStorage keys are versioned. The About tab says data is saved on this device only.
- [ ] The layout follows the chosen look's recipe on both desktop and phone.