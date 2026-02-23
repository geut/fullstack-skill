---
title: shadcn/ui Component Patterns
impact: HIGH
tags: [react, components, shadcn, styling, tailwind]
---

# shadcn/ui Component Patterns

Use consistent patterns when building React components with shadcn/ui, CVA, and Tailwind CSS. These patterns ensure composability, type safety, and maintainable styling across your frontend applications.

## Why

- **Composability**: `asChild` pattern and Slot component allow flexible component composition
- **Type Safety**: CVA provides type-safe variants with autocomplete support
- **Maintainability**: Consistent structure makes components easy to read and modify
- **Developer Experience**: `data-slot` attributes enable easy debugging and testing
- **Styling**: `cn()` utility prevents Tailwind class conflicts

## Use data-slot Attributes

All components should include `data-slot` attributes for debugging, testing selectors, and styling hooks.

```tsx
// Bad: Missing data-slot
function Button({ children }) {
  return <button>{children}</button>
}

// Good: Include data-slot for component identification
function Button({ children }) {
  return <button data-slot="button">{children}</button>
}
```

## Use CVA for Variants

Define component variants using class-variance-authority for type-safe, composable styling.

```tsx
// Bad: Manual className manipulation
function Button({ variant, size, className, children }) {
  const classes = [
    'inline-flex items-center justify-center rounded-md',
    variant === 'destructive' ? 'bg-red-500' : 'bg-blue-500',
    size === 'sm' ? 'h-8 px-3' : 'h-9 px-4',
    className
  ].join(' ')
  return <button className={classes}>{children}</button>
}

// Good: Type-safe variants with CVA
import { cva, type VariantProps } from 'class-variance-authority'

const buttonVariants = cva(
  'inline-flex items-center justify-center rounded-md font-medium transition-colors',
  {
    variants: {
      variant: {
        default: 'bg-primary text-primary-foreground hover:bg-primary/90',
        destructive: 'bg-destructive text-destructive-foreground hover:bg-destructive/90',
        outline: 'border border-input bg-background hover:bg-accent hover:text-accent-foreground',
        secondary: 'bg-secondary text-secondary-foreground hover:bg-secondary/80',
        ghost: 'hover:bg-accent hover:text-accent-foreground',
        link: 'text-primary underline-offset-4 hover:underline',
      },
      size: {
        default: 'h-9 px-4 py-2',
        sm: 'h-8 rounded-md px-3 text-xs',
        lg: 'h-10 rounded-md px-8',
        icon: 'h-9 w-9',
      },
    },
    defaultVariants: {
      variant: 'default',
      size: 'default',
    },
  }
)

interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  asChild?: boolean
}

function Button({ className, variant, size, asChild = false, ...props }: ButtonProps) {
  const Comp = asChild ? Slot : 'button'
  return (
    <Comp
      data-slot="button"
      className={cn(buttonVariants({ variant, size, className }))}
      {...props}
    />
  )
}
```

## Use the cn() Utility

Always use the `cn()` utility (tailwind-merge + clsx) for className merging to avoid conflicts.

```tsx
// Bad: Direct string concatenation
function Card({ className, children }) {
  return (
    <div className={`rounded-lg border bg-card text-card-foreground shadow-sm ${className}`}>
      {children}
    </div>
  )
}

// Good: Use cn() utility for proper class merging
import { cn } from '@/lib/utils'

function Card({ className, children }) {
  return (
    <div
      data-slot="card"
      className={cn('rounded-lg border bg-card text-card-foreground shadow-sm', className)}
    >
      {children}
    </div>
  )
}
```

## Use asChild Pattern for Composition

Enable flexible composition by supporting `asChild` prop with Radix Slot.

```tsx
// Bad: Inflexible component structure
function DropdownMenuTrigger({ children }) {
  return <button className="dropdown-trigger">{children}</button>
}

// Good: Support asChild for flexible composition
import { Slot } from '@radix-ui/react-slot'

function DropdownMenuTrigger({
  className,
  asChild = false,
  ...props
}: React.ComponentProps<typeof DropdownMenuPrimitive.Trigger>) {
  const Comp = asChild ? Slot : DropdownMenuPrimitive.Trigger
  return (
    <Comp
      data-slot="dropdown-menu-trigger"
      className={cn('inline-flex', className)}
      {...props}
    />
  )
}

// Usage: Can render as any element
<DropdownMenuTrigger asChild>
  <a href="/profile">Profile</a>
</DropdownMenuTrigger>
```

## Rules

1. Always include `data-slot` attributes on component root elements for debugging and testing
2. Use CVA (class-variance-authority) for all variant-based styling with explicit default variants
3. Use the `cn()` utility from `@/lib/utils` for all className merging
4. Support `asChild` prop using Radix Slot for flexible component composition
5. Expose variant props as data attributes (e.g., `data-variant`, `data-size`) when helpful for styling
6. Forward refs properly for all interactive components
7. Use semantic HTML elements as the default (button, not div with onClick)
