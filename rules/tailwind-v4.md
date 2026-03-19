---
title: Tailwind CSS v4 Configuration
impact: HIGH
tags: [tailwind, css, styling]
---

# Tailwind CSS v4 Configuration

Use CSS-first configuration for Tailwind CSS v4 in Astro and React projects. This modern approach leverages CSS imports, OKLCH color space, and inline theme definitions for better performance and maintainability.

## Why

- **Performance**: CSS-first approach eliminates JavaScript configuration overhead
- **Maintainability**: Native CSS syntax is more familiar and easier to debug
- **OKLCH Colors**: Perceptually uniform color space for better visual consistency
- **Type Safety**: Better IDE support with CSS custom properties
- **Astro Compatibility**: Seamless integration with Astro's static generation

## CSS-First Configuration

Configure Tailwind v4 entirely through CSS instead of JavaScript config files.

```css
/* Bad: JavaScript config (Tailwind v3 style) */
/* tailwind.config.js */
module.exports = {
  content: ['./src/**/*.{astro,tsx}'],
  theme: {
    extend: {
      colors: {
        background: '#ffffff',
      },
    },
  },
}

/* Good: CSS-first config (Tailwind v4) */
/* src/styles/global.css */
@import "tailwindcss";
@import "tw-animate-css";

@custom-variant dark (&:is(.dark *));

@theme inline {
  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --font-family-sans: 'Inter Variable', sans-serif;
  --breakpoint-xs: 475px;
}

:root {
  --radius: 0.625rem;
  --background: oklch(1 0 0);
  --foreground: oklch(0.141 0.005 285.823);
}

.dark {
  --background: oklch(0.141 0.005 285.823);
  --foreground: oklch(0.985 0 0);
}

@layer base {
  * {
    @apply border-border outline-ring/50;
  }
  body {
    @apply bg-background text-foreground;
  }
}
```

## Use OKLCH Color Space

Use OKLCH for perceptually uniform colors that maintain consistent perceived brightness.

```css
/* Bad: Arbitrary hex colors with inconsistent perceived brightness */
:root {
  --background: #ffffff;
  --primary: #3b82f6;
  --destructive: #ef4444;
  --muted: #6b7280;
}

/* Good: OKLCH colors with consistent lightness */
:root {
  --background: oklch(1 0 0);
  --foreground: oklch(0.141 0.005 285.823);
  --primary: oklch(0.546 0.245 262.881);
  --primary-foreground: oklch(0.985 0 0);
  --destructive: oklch(0.577 0.245 27.325);
  --destructive-foreground: oklch(0.985 0 0);
  --muted: oklch(0.967 0.001 286.375);
  --muted-foreground: oklch(0.552 0.016 285.938);
  --accent: oklch(0.967 0.001 286.375);
  --accent-foreground: oklch(0.21 0.006 285.885);
}
```

## Semantic Color Variables

Use semantic naming for CSS variables that map to UI concepts.

```css
/* Bad: Literal color names */
:root {
  --blue: #3b82f6;
  --red: #ef4444;
  --gray: #6b7280;
}

/* Good: Semantic UI color names */
:root {
  --background: oklch(1 0 0);
  --foreground: oklch(0.141 0.005 285.823);
  --card: oklch(1 0 0);
  --card-foreground: oklch(0.141 0.005 285.823);
  --popover: oklch(1 0 0);
  --popover-foreground: oklch(0.141 0.005 285.823);
  --primary: oklch(0.546 0.245 262.881);
  --primary-foreground: oklch(0.985 0 0);
  --secondary: oklch(0.967 0.001 286.375);
  --secondary-foreground: oklch(0.21 0.006 285.885);
  --muted: oklch(0.967 0.001 286.375);
  --muted-foreground: oklch(0.552 0.016 285.938);
  --accent: oklch(0.967 0.001 286.375);
  --accent-foreground: oklch(0.21 0.006 285.885);
  --destructive: oklch(0.577 0.245 27.325);
  --destructive-foreground: oklch(0.985 0 0);
  --border: oklch(0.92 0.004 286.32);
  --input: oklch(0.92 0.004 286.32);
  --ring: oklch(0.546 0.245 262.881);
  --radius: 0.625rem;
}
```

## Typography Utilities

Define typography patterns as CSS utility classes using @layer.

```css
@layer utilities {
  h1 {
    @apply scroll-m-20 text-4xl font-extrabold tracking-tight lg:text-5xl;
  }
  h2:not(.dialog-title) {
    @apply scroll-m-20 pb-2 text-3xl font-semibold tracking-tight;
  }
  h3 {
    @apply scroll-m-20 text-2xl font-semibold tracking-tight;
  }
  h4 {
    @apply scroll-m-20 text-xl font-semibold tracking-tight;
  }
  p {
    @apply leading-7;
  }
  blockquote {
    @apply border-l-2 pl-6 italic;
  }
  ul {
    @apply my-6 ml-6 list-disc [&>li]:mt-2;
  }
  code {
    @apply bg-muted relative rounded px-[0.3rem] py-[0.2rem] font-mono text-sm font-semibold;
  }
}
```

## Rules

1. Use CSS-first configuration with `@import "tailwindcss"` instead of JavaScript config files
2. Define theme variables using `@theme inline` with CSS custom properties
3. Use OKLCH color space for all colors to ensure perceptual uniformity
4. Name colors semantically (background, primary, destructive) not literally (blue, red)
5. Provide both light and dark theme variable sets using `.dark` class
6. Use `@layer base` to apply default styles to HTML elements
7. Define typography utilities in `@layer utilities` for consistent text styling
8. Import animation libraries via CSS (e.g., `@import "tw-animate-css"`)
9. Map theme variables to Tailwind utilities using `@theme inline` declarations
