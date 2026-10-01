---
name: Modern Commerce Minimal
colors:
  surface: '#faf8ff'
  surface-dim: '#d2d9f4'
  surface-bright: '#faf8ff'
  surface-container-lowest: '#ffffff'
  surface-container-low: '#f2f3ff'
  surface-container: '#eaedff'
  surface-container-high: '#e2e7ff'
  surface-container-highest: '#dae2fd'
  on-surface: '#131b2e'
  on-surface-variant: '#444653'
  inverse-surface: '#283044'
  inverse-on-surface: '#eef0ff'
  outline: '#757684'
  outline-variant: '#c4c5d5'
  surface-tint: '#3755c3'
  primary: '#00288e'
  on-primary: '#ffffff'
  primary-container: '#1e40af'
  on-primary-container: '#a8b8ff'
  inverse-primary: '#b8c4ff'
  secondary: '#006c4a'
  on-secondary: '#ffffff'
  secondary-container: '#82f5c1'
  on-secondary-container: '#00714e'
  tertiary: '#4c2e00'
  on-tertiary: '#ffffff'
  tertiary-container: '#6b4200'
  on-tertiary-container: '#ffa929'
  error: '#ba1a1a'
  on-error: '#ffffff'
  error-container: '#ffdad6'
  on-error-container: '#93000a'
  primary-fixed: '#dde1ff'
  primary-fixed-dim: '#b8c4ff'
  on-primary-fixed: '#001453'
  on-primary-fixed-variant: '#173bab'
  secondary-fixed: '#85f8c4'
  secondary-fixed-dim: '#68dba9'
  on-secondary-fixed: '#002114'
  on-secondary-fixed-variant: '#005137'
  tertiary-fixed: '#ffddb8'
  tertiary-fixed-dim: '#ffb95f'
  on-tertiary-fixed: '#2a1700'
  on-tertiary-fixed-variant: '#653e00'
  background: '#faf8ff'
  on-background: '#131b2e'
  surface-variant: '#dae2fd'
typography:
  display:
    fontFamily: Plus Jakarta Sans
    fontSize: 48px
    fontWeight: '800'
    lineHeight: 56px
    letterSpacing: -0.025em
  display-mobile:
    fontFamily: Plus Jakarta Sans
    fontSize: 36px
    fontWeight: '800'
    lineHeight: 44px
    letterSpacing: -0.02em
  headline-lg:
    fontFamily: Plus Jakarta Sans
    fontSize: 32px
    fontWeight: '700'
    lineHeight: 40px
    letterSpacing: -0.02em
  headline-lg-mobile:
    fontFamily: Plus Jakarta Sans
    fontSize: 26px
    fontWeight: '700'
    lineHeight: 34px
    letterSpacing: -0.015em
  headline-md:
    fontFamily: Plus Jakarta Sans
    fontSize: 24px
    fontWeight: '600'
    lineHeight: 32px
    letterSpacing: -0.015em
  headline-sm:
    fontFamily: Plus Jakarta Sans
    fontSize: 20px
    fontWeight: '600'
    lineHeight: 28px
    letterSpacing: -0.01em
  body-lg:
    fontFamily: Plus Jakarta Sans
    fontSize: 18px
    fontWeight: '400'
    lineHeight: 28px
  body-md:
    fontFamily: Plus Jakarta Sans
    fontSize: 16px
    fontWeight: '400'
    lineHeight: 24px
  body-sm:
    fontFamily: Plus Jakarta Sans
    fontSize: 14px
    fontWeight: '400'
    lineHeight: 20px
  label-md:
    fontFamily: Plus Jakarta Sans
    fontSize: 14px
    fontWeight: '600'
    lineHeight: 20px
    letterSpacing: 0.01em
  label-sm:
    fontFamily: Plus Jakarta Sans
    fontSize: 12px
    fontWeight: '600'
    lineHeight: 16px
    letterSpacing: 0.02em
  price-lg:
    fontFamily: Plus Jakarta Sans
    fontSize: 24px
    fontWeight: '700'
    lineHeight: 28px
    letterSpacing: -0.02em
  price-md:
    fontFamily: Plus Jakarta Sans
    fontSize: 18px
    fontWeight: '700'
    lineHeight: 22px
    letterSpacing: -0.01em
rounded:
  sm: 0.25rem
  DEFAULT: 0.5rem
  md: 0.75rem
  lg: 1rem
  xl: 1.5rem
  full: 9999px
spacing:
  gutter: 1.5rem
  gutter-sm: 1rem
  margin: 2rem
  margin-sm: 1rem
  space-xs: 0.25rem
  space-sm: 0.5rem
  space-md: 1rem
  space-lg: 1.5rem
  space-xl: 2.5rem
---

## Brand & Style

The design system establishes a high-trust, transaction-focused digital storefront engineered for multi-category retail. The brand personality is dependable, frictionless, and understatedly premium. The interface gets out of the way of the merchandise while instilling an immediate sense of institutional reliability and financial security.

### Design Aesthetic
The visual language merges **Modern Functional Minimalism** with **Subtle Tonal Layering**. It leverages generous whitespace, precise typographic hierarchies, crisp borders, and soft ambient drop shadows. Visual noise is systematically reduced to prioritize product imagery, pricing transparency, and the checkout conversion funnel.

## Colors

The color system operates on high-contrast utility and clear semantic signals tailored to an e-commerce purchasing lifecycle:

- **Primary (`#1E40AF` - Deep Cobalt Indigo):** Represents authority, checkout confidence, and brand touchpoints. Used for primary CTAs, active navigation states, order confirmation highlights, and critical links.
- **Secondary (`#059669` - Emerald):** Drives transactional incentives, promotions, and positive states. Applied to discount badges, in-stock indicators, cart success banners, and free shipping trackers.
- **Tertiary (`#F59E0B` - Amber Warmth):** Reserved for contextual alerts, customer ratings (stars), limited-quantity countdowns, and urgency notices.
- **Neutral Palette:**
  - Canvas / Background: `#F8FAFC` (Slate 50) for outer frame contrast; pure `#FFFFFF` for product card surfaces and modals.
  - Borders & Dividers: `#E2E8F0` (Slate 200) for clean segmenting without heavy lines.
  - Text Primary: `#0F172A` (Slate 900) for deep, fatigue-free readability.
  - Text Muted: `#64748B` (Slate 500) for secondary metadata, SKUs, and specifications.

## Typography

Plus Jakarta Sans is utilized across all typographic scales to balance geometric precision with humane warmth. The large x-height ensures superior rendering on dense mobile catalog screens and small UI elements like badges and filter tags. Tabular figures (`tnum`) should be enabled for product prices, cart line totals, and inventory counts to preserve alignment in checkout summaries.

## Layout & Spacing

The layout is structured around an adaptive 12-column grid system capped at a maximum container width of `1280px` (`max-w-7xl`):

- **Desktop (1024px+):** 12 columns, `1.5rem` (`24px`) gutters, `2rem` (`32px`) lateral margins.
- **Tablet (768px - 1023px):** 8 columns, `1rem` (`16px`) gutters, `1.5rem` (`24px`) margins. Sidebars collapse into sticky filters or slide-out sheets.
- **Mobile (<768px):** 4 columns or single-column stack, `1rem` (`16px`) gutters, `1rem` (`16px`) outer margins. Product feeds render as a 2-column grid.

Spacing increments adhere to an 8pt rhythmic grid (`0.25rem` to `2.5rem`), keeping component padding tight enough for dense transactional browsing while giving hero promotions breathability.

## Elevation & Depth

Visual hierarchy uses ultra-soft ambient lighting paired with structural borders to maintain clarity without heavy shadows:

- **Surface Neutral (Ground):** `#F8FAFC` base application frame.
- **Surface Layer 1 (Card & Section):** Solid `#FFFFFF` enclosed by a 1px border in `#E2E8F0` with a subtle elevation of `0 1px 3px 0 rgba(15, 23, 42, 0.04)`.
- **Surface Layer 2 (Hover & Popover):** Elevated on interaction using `0 10px 25px -5px rgba(15, 23, 42, 0.08), 0 8px 10px -6px rgba(15, 23, 42, 0.03)` with no border color change.
- **Surface Layer 3 (Modals, Slide-over Cart & Drawers):** High-layer elevation at `0 20px 25px -5px rgba(15, 23, 42, 0.12)` complemented by an ambient backdrop blur (`backdrop-blur-sm bg-slate-900/40`).

## Shapes

The design system implements a refined **Rounded (0.5rem base)** contour profile:

- Small elements (inputs, tags, micro-buttons, badges): `0.375rem` (`rounded-md`).
- Primary interactive components (buttons, product cards, select menus): `0.5rem` (`rounded-lg`).
- Major surfaces (flyout cart, checkout modals, banner containers): `0.75rem` to `1rem` (`rounded-xl` to `rounded-2xl`).
- Status pills, avatars, and floating micro-actions: `9999px` (`rounded-full`).

## Components

### Buttons
- **Primary / Checkout CTA:** Solid `#1E40AF` fill with `#FFFFFF` text. Height of `44px` on mobile and `48px` on desktop for touch targets. Hover state transitions to `#1D4ED8` with a micro-scale of `1.01`. Focus ring is `2px` offset with `#93C5FD`.
- **Secondary:** `#FFFFFF` background with 1px `#E2E8F0` border and `#0F172A` text. Hover shifts to `#F1F5F9`.
- **Express / Promotion CTA:** Solid `#059669` fill with white text for quick checkout or flash-sale interactions.

### Product Cards
- Contained within `#FFFFFF` cards, `1px` border of `#E2E8F0`, rounded at `0.5rem`.
- Product media aspect ratio fixed at `1:1` or `4:5` on an off-white background (`#F1F5F9`) with smooth hover image swaps.
- Content stack: Category tag, truncated title (`line-clamp-2`), star rating row, price lockup (discounted price in `#0F172A` bold, original price in `#94A3B8` strikethrough), and a compact "Add to Cart" button.

### Badges & Chips
- **Discount & Deals:** Emerald background tint (`#ECFDF5`) with solid emerald text (`#047857`) and a subtle inner border (`border-emerald-200/60`).
- **Stock Indicators:** Small inline pill with a ping dot (`#10B981` for in-stock, `#F59E0B` for low inventory).
- **Category Filter Chips:** Neutral outlines that turn into `#1E40AF` filled badges when active.

### Form Inputs & Selects
- Height of `42px`, background `#FFFFFF`, border `#CBD5E1`, text `#0F172A`.
- Placeholder colored in `#94A3B8`.
- Focus state highlights with an active border in `#1E40AF` and a subtle halo ring `ring-2 ring-blue-100`.

### Checkboxes & Radios
- Size `18px`, rounded `4px` (checkbox) or circular (radio), `#1E40AF` fill on checked with white glyph.

### GDPR & Cookie Notice Banner
- Fixed bottom-floating dock, `rounded-xl`, high contrast (`#0F172A` deep slate background with `#F8FAFC` typography).
- Clear, un-coerced affirmative and preferences actions utilizing a compact secondary button and a mini primary button.