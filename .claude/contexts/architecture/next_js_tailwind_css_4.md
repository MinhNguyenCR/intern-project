# Next.js + TailwindCSS 4.0 Architecture (Production Standard)

---

## 🎯 Goals

- UI consistency across the entire system
- Scalable for large teams (10+ devs)
- Token-driven design system (single source of truth)
- Zero hardcoded styles
- Optimized for TailwindCSS 4.0 (CSS-first config)

---

## 🧠 Core Principles

1. **CSS-first configuration (`@theme`) is the source of truth**
2. **No Tailwind config unless absolutely necessary**
3. **All styling must go through tokens → components → UI**
4. **No Tailwind usage directly in pages**
5. **Variants must be centralized (CVA)**

---

## 📁 Folder Structure

```
/src
  /app
  /components
    /ui            # primitive UI (Button, Input, Card)
    /shared        # reusable business components
    /features      # domain-specific components
    /layouts       # layout components
  /styles
    globals.css
    theme.css      # Tailwind 4 @theme (MANDATORY)
    tokens.css     # raw design tokens (optional)
  /lib
    cn.ts
    variants.ts

postcss.config.mjs
```

---

## 🎨 Styling Architecture

### 1. Token Layer (optional but recommended)

`tokens.css`

```css
:root {
  --color-primary: 59 130 246;
  --color-secondary: 99 102 241;
  --color-danger: 239 68 68;

  --spacing-xs: 4px;
  --spacing-sm: 8px;
  --spacing-md: 12px;
  --spacing-lg: 16px;
  --spacing-xl: 24px;

  --radius-sm: 6px;
  --radius-md: 10px;
  --radius-lg: 14px;
}
```

---

### 2. Theme Layer (REQUIRED)

`theme.css`

```css
@theme {
  --color-primary: rgb(var(--color-primary));
  --color-secondary: rgb(var(--color-secondary));
  --color-danger: rgb(var(--color-danger));

  --spacing-xs: var(--spacing-xs);
  --spacing-sm: var(--spacing-sm);
  --spacing-md: var(--spacing-md);
  --spacing-lg: var(--spacing-lg);
  --spacing-xl: var(--spacing-xl);

  --radius-sm: var(--radius-sm);
  --radius-md: var(--radius-md);
  --radius-lg: var(--radius-lg);
}
```

---

## 🧱 Component Architecture

### Layering

- `ui/` → primitive components
- `shared/` → reusable business logic UI
- `features/` → domain-based UI
- `app/` → pages only (NO styling)

---

### RULE: No Tailwind in pages

❌ BAD

```
<div className="p-4 bg-white rounded-lg" />
```

✅ GOOD

```
<Card />
```

---

## 🎛️ Variant System (CVA)

```ts
import { cva } from "class-variance-authority";

export const buttonVariants = cva(
  "inline-flex items-center justify-center font-medium transition",
  {
    variants: {
      variant: {
        primary: "bg-primary text-white",
        secondary: "bg-secondary text-white",
        danger: "bg-danger text-white",
      },
      size: {
        sm: "h-8 px-sm",
        md: "h-10 px-md",
        lg: "h-12 px-lg",
      },
    },
    defaultVariants: {
      variant: "primary",
      size: "md",
    },
  }
);
```

---

## 🧩 Utility Layer

### cn() helper

```ts
import { clsx } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs) {
  return twMerge(clsx(inputs));
}
```

---

## 🎯 Naming Convention (STRICT)

### Tokens

- Colors → `primary`, `secondary`, `danger`
- Spacing → `xs, sm, md, lg, xl`
- Radius → `sm, md, lg`

---

### Components

```
Button.tsx
Card.tsx
UserCard.tsx
```

---

## 🚫 Anti-patterns (FORBIDDEN)

- ❌ Hardcoded Tailwind colors (`bg-red-500`)
- ❌ Arbitrary values (`p-[13px]`)
- ❌ Inline conditional class logic
- ❌ Styling inside pages
- ❌ Duplicate UI

---

## 🧠 Layout System

### Container

```
max-w-screen-xl mx-auto px-lg
```

---

### Spacing Scale Only

```
xs / sm / md / lg / xl
```

---

## 🌗 Theming (Dark mode / Multi-theme)

```css
[data-theme="dark"] {
  --color-primary: 147 197 253;
}
```

---

## 🧪 PR Checklist

- [ ] No hardcoded styles
- [ ] Uses @theme tokens
- [ ] No Tailwind in pages
- [ ] Uses UI components
- [ ] Uses CVA variants
- [ ] Uses cn()

---

## 🚀 When to use tailwind.config.ts

Only if:

- Using plugins
- Complex variants
- Monorepo shared config

Otherwise → DO NOT USE

---

## 🧬 Final Architecture

```
@theme (CSS)      → Design Tokens
Components        → UI Contract
CVA               → Variants
Tailwind Engine   → Rendering
```

---

## 🔥 Advanced Recommendations

- Integrate Figma → tokens pipeline
- Multi-theme SaaS support
- Storybook for UI testing
- Build internal design system package

---

**This architecture is optimized for TailwindCSS 4.0 and modern Next.js App Router applications at scale.**
