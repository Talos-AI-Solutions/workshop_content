---
description: Used when writing Tailwind CSS code
globs: 
alwaysApply: false
---
# Tailwind CSS Standards

Establishes utility-first styling patterns and component organization standards for consistent and maintainable UI development using Tailwind CSS v4 in TypeScript Next.js projects.

## Prerequisites

State all requirements upfront:
- Next.js project with TypeScript already initialized
- Tailwind CSS v4 installed and configured via create-next-app
- PostCSS configured with @tailwindcss/postcss plugin
- Windows PowerShell or Command Prompt
- VS Code or compatible editor with Tailwind CSS IntelliSense extension

## Core Principle

**Use utility-first approach with Tailwind CSS v4's CSS-first configuration, prioritizing composition over custom CSS, while maintaining component readability and reusability through consistent patterns and @theme directive design tokens.**

## Step-by-Step Instructions

### Step 1: Verify Tailwind CSS v4 Installation

Check if Tailwind CSS v4 is properly installed with PostCSS:

```powershell
# Verify Tailwind CSS v4 is installed
npm list tailwindcss
# Expected output: Shows tailwindcss@^4.x.x

# Check PostCSS configuration for v4
Get-Content postcss.config.mjs | Select-String "@tailwindcss/postcss"
# Expected output: Shows "@tailwindcss/postcss" plugin

# Verify no traditional config file (v4 uses CSS-first approach)
Test-Path tailwind.config.ts
# Expected output: False (or True if migrating from v3)
```

**Expected result**: Tailwind CSS v4 is installed with PostCSS plugin, using CSS-first configuration approach.

### Step 2: Configure Tailwind v4 CSS-First Design System

Configure design tokens using v4's CSS-first approach in globals.css:

```css
@import "tailwindcss";

@theme {
  /* Primary color palette - automatically generates bg-primary-*, text-primary-*, etc. */
  --color-primary-50: #eff6ff;
  --color-primary-100: #dbeafe;
  --color-primary-200: #bfdbfe;
  --color-primary-300: #93c5fd;
  --color-primary-400: #60a5fa;
  --color-primary-500: #3b82f6;
  --color-primary-600: #2563eb;
  --color-primary-700: #1d4ed8;
  --color-primary-800: #1e40af;
  --color-primary-900: #1e3a8a;
  --color-primary-950: #172554;

  /* Secondary color palette */
  --color-secondary-50: #f9fafb;
  --color-secondary-100: #f3f4f6;
  --color-secondary-200: #e5e7eb;
  --color-secondary-300: #d1d5db;
  --color-secondary-400: #9ca3af;
  --color-secondary-500: #6b7280;
  --color-secondary-600: #4b5563;
  --color-secondary-700: #374151;
  --color-secondary-800: #1f2937;
  --color-secondary-900: #111827;
  --color-secondary-950: #030712;

  /* Custom spacing - automatically generates p-18, m-88, etc. */
  --spacing-18: 4.5rem;
  --spacing-88: 22rem;

  /* Font families */
  --font-family-sans: Inter, system-ui, sans-serif;

  /* Custom animations */
  --animate-fade-in: fadeIn 0.5s ease-in-out;
  --animate-slide-up: slideUp 0.3s ease-out;
}

@keyframes fadeIn {
  0% { opacity: 0; }
  100% { opacity: 1; }
}

@keyframes slideUp {
  0% { transform: translateY(10px); opacity: 0; }
  100% { transform: translateY(0); opacity: 1; }
}
```

**Expected result**: CSS-first design system configured with automatic utility class generation.

**CRITICAL v4 Configuration Changes:**
- **NO traditional config file needed** - everything is in CSS
- Use `@import "tailwindcss"` instead of `@tailwind base/components/utilities`
- Use `@theme` directive for design tokens instead of JavaScript config
- Color variables use `--color-` prefix for automatic utility generation
- Spacing variables use `--spacing-` prefix for automatic spacing classes
- PostCSS plugin handles compilation automatically

### Step 3: Complete Global CSS with Component Layers

Add component layers to the v4 configuration:

```css
/* Previous @import "tailwindcss" and @theme sections from Step 2... */

/* Custom base layer - foundational styles */
@layer base {
  html {
    @apply scroll-smooth;
  }
  
  body {
    @apply bg-white text-gray-900 antialiased;
  }

  /* Focus styles for accessibility */
  *:focus-visible {
    @apply outline-none ring-2 ring-primary-500 ring-offset-2;
  }
}

/* Custom component styles */
@layer components {
  .btn {
    @apply inline-flex items-center justify-center rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-primary-500 disabled:pointer-events-none disabled:opacity-50;
  }
  
  .btn-primary {
    @apply bg-primary-600 text-white hover:bg-primary-700 focus-visible:ring-primary-600;
  }
  
  .btn-secondary {
    @apply bg-secondary-200 text-secondary-900 hover:bg-secondary-300 focus-visible:ring-secondary-600;
  }

  .btn-outline {
    @apply border border-gray-300 bg-white text-gray-700 hover:bg-gray-50 focus-visible:ring-gray-600;
  }

  /* Card foundation classes */
  .card {
    @apply bg-white border border-gray-200 rounded-lg shadow-sm;
  }

  .card-hover {
    @apply hover:shadow-md transition-shadow duration-200;
  }
}
```

**Expected result**: Complete v4 configuration with design tokens and component layers.

**CRITICAL v4 Component Layer Notes:**
- Component layers work the same as v3 - use `@layer components`
- Fix invalid ring references: use `ring-primary-500` not `ring-ring`
- All design token references now use the CSS custom properties defined in `@theme`
- Base layer provides foundational accessibility and styling patterns

### Step 4: Create Component Utility Patterns

Create consistent patterns for common UI components:

```powershell
# Create components directory if it doesn't exist
if (-not (Test-Path "src/components/ui")) { 
  New-Item -ItemType Directory -Path "src/components/ui" -Force 
}
# Expected output: Directory created
```

Create example component following Tailwind patterns:

```typescript
// File: src/components/ui/Button.tsx
interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'outline' | 'ghost';
  size?: 'sm' | 'md' | 'lg';
  children: React.ReactNode;
}

export const Button: React.FC<ButtonProps> = ({
  variant = 'primary',
  size = 'md',
  className = '',
  children,
  ...props
}) => {
  const baseClasses = 'inline-flex items-center justify-center rounded-md font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 disabled:pointer-events-none disabled:opacity-50';
  
  const variantClasses = {
    primary: 'bg-primary-600 text-white hover:bg-primary-700 focus-visible:ring-primary-600',
    secondary: 'bg-secondary-200 text-secondary-900 hover:bg-secondary-300 focus-visible:ring-secondary-600',
    outline: 'border border-gray-300 bg-white text-gray-700 hover:bg-gray-50 focus-visible:ring-gray-600',
    ghost: 'text-gray-700 hover:bg-gray-100 focus-visible:ring-gray-600',
  };
  
  const sizeClasses = {
    sm: 'h-9 px-3 text-sm',
    md: 'h-10 px-4 text-sm',
    lg: 'h-11 px-6 text-base',
  };
  
  const combinedClasses = `${baseClasses} ${variantClasses[variant]} ${sizeClasses[size]} ${className}`;
  
  return (
    <button className={combinedClasses} {...props}>
      {children}
    </button>
  );
};
```

**Expected result**: Reusable component with proper Tailwind utility composition.

### Step 5: Install ShadCN with Tailwind v4

ShadCN provides pre-built, accessible components that integrate seamlessly with Tailwind v4:

```powershell
# Initialize ShadCN (interactive prompts will appear)
npx shadcn@latest init

# Configuration prompts - use these values:
# - Would you like to use TypeScript? Yes
# - Which style would you like to use? Default
# - Which color would you like to use as base color? Slate
# - Where is your global CSS file? src/app/globals.css
# - Would you like to use CSS variables for colors? Yes
# - Where is your tailwind.config located? (Press Enter for default)
# - Configure the import alias for components? @/components
# - Configure the import alias for utils? @/lib/utils

# Expected output: Created components.json, updated globals.css
```

**Expected result**: ShadCN initialized with Tailwind v4 CSS-first configuration.

### Step 6: Add ShadCN Components

Install commonly used ShadCN components:

```powershell
# Add core UI components
npx shadcn@latest add button
npx shadcn@latest add card
npx shadcn@latest add input
npx shadcn@latest add form

# Verify components were added
Test-Path "src/components/ui/button.tsx"
# Expected output: True

# View all available components
npx shadcn@latest add
# Shows interactive list of all components
```

**Expected result**: ShadCN components available in `src/components/ui/`.

### Step 7: Customize ShadCN with @theme

ShadCN components automatically use your Tailwind v4 design tokens from `@theme`:

```css
/* Your existing @theme in globals.css already configures ShadCN */
@theme {
  /* Primary colors - used by ShadCN button variant="default" */
  --color-primary-500: #3b82f6;
  --color-primary-600: #2563eb;

  /* Border radius - used by ShadCN cards, buttons, inputs */
  --radius-lg: 0.5rem;
  --radius-md: 0.375rem;
  --radius-sm: 0.25rem;
}
```

Usage example with ShadCN components:

```typescript
// File: src/app/page.tsx
import { Button } from "@/components/ui/button"
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from "@/components/ui/card"
import { Input } from "@/components/ui/input"

export default function Home() {
  return (
    <div className="container mx-auto p-6">
      <Card className="max-w-md">
        <CardHeader>
          <CardTitle>ShadCN with Tailwind v4</CardTitle>
          <CardDescription>
            Components automatically use your @theme tokens
          </CardDescription>
        </CardHeader>
        <CardContent className="space-y-4">
          <Input placeholder="Email address" />
          <Button className="w-full">
            Uses --color-primary-500 from @theme
          </Button>
        </CardContent>
      </Card>
    </div>
  )
}
```

**Expected result**: ShadCN components styled with your project's design tokens.

**CRITICAL ShadCN Integration Notes:**
- Components automatically respect your `@theme` color variables
- Use `@/components/ui/*` import alias for all ShadCN components
- Customize components by editing files in `src/components/ui/`
- All components include accessibility features (ARIA attributes, keyboard navigation)
- Components are server-compatible (work in Next.js Server Components)

## Utility Class Organization Standards

### Class Order Convention
Always organize Tailwind classes in this order:
1. **Layout**: `flex`, `grid`, `block`, `inline`, `absolute`, `relative`
2. **Sizing**: `w-*`, `h-*`, `min-*`, `max-*`
3. **Spacing**: `m-*`, `p-*`, `space-*`
4. **Typography**: `text-*`, `font-*`, `leading-*`, `tracking-*`
5. **Colors**: `bg-*`, `text-*`, `border-*`
6. **Borders**: `border`, `border-*`, `rounded-*`
7. **Effects**: `shadow-*`, `opacity-*`, `transform`, `transition-*`
8. **Interactive**: `hover:*`, `focus:*`, `active:*`
9. **Responsive**: `sm:*`, `md:*`, `lg:*`, `xl:*`

### Responsive Design Patterns

Use mobile-first responsive design:

```typescript
// ✅ Correct: Mobile-first approach
<div className="w-full md:w-1/2 lg:w-1/3">
  <div className="p-4 md:p-6 lg:p-8">
    <h2 className="text-xl md:text-2xl lg:text-3xl font-bold">
      Title
    </h2>
  </div>
</div>

// ❌ Incorrect: Desktop-first approach
<div className="lg:w-1/3 md:w-1/2 w-full">
```

### Color Usage Standards

Use semantic color naming:

```typescript
// ✅ Correct: Semantic color usage
<div className="bg-primary-500 text-white">Primary Action</div>
<div className="bg-red-500 text-white">Error State</div>
<div className="bg-green-500 text-white">Success State</div>

// ❌ Incorrect: Non-semantic colors
<div className="bg-blue-500 text-white">Delete Button</div>
<div className="bg-yellow-500 text-black">Success Message</div>
```

### Spacing Consistency

Use consistent spacing scale:

```typescript
// ✅ Correct: Consistent spacing
<div className="space-y-4"> {/* 1rem */}
  <div className="p-4">Content</div>
  <div className="mb-8">Section</div> {/* 2rem */}
</div>

// ❌ Incorrect: Arbitrary spacing
<div className="space-y-3">
  <div className="p-5">Content</div>
  <div className="mb-7">Section</div>
</div>
```

## Verification Steps

How to confirm the rule was followed correctly:
- [ ] All components use utility-first approach with minimal custom CSS
- [ ] Class order follows the established convention
- [ ] Responsive design uses mobile-first approach
- [ ] Colors follow semantic naming patterns
- [ ] Spacing uses consistent scale values
- [ ] ShadCN initialized and components installed
- [ ] ShadCN components use @theme design tokens

**Success criteria verification:**
```powershell
# Check if Tailwind v4 is properly configured
Get-Content postcss.config.mjs | Select-String "@tailwindcss/postcss"
# Should show PostCSS plugin

# Verify global CSS includes v4 syntax
Get-Content src/app/globals.css | Select-String "@import.*tailwindcss"
Get-Content src/app/globals.css | Select-String "@theme"
# Should show v4 import and theme directives

# Verify ShadCN is initialized
Test-Path "components.json"
# Should return True

# Verify ShadCN components directory exists
Test-Path "src/components/ui"
# Should return True

# Check for installed ShadCN components
Get-ChildItem -Path "src/components/ui" -Filter "*.tsx" | Select-Object Name
# Should show installed component files

# Test build process
npm run build
# Should complete without Tailwind-related errors
```

## Troubleshooting Common Issues

### Problem: Tailwind Classes Not Applied
**Symptoms**: 
- Styles not appearing in browser
- Classes show in HTML but no visual effect

**Solution**: 
```powershell
# Check if PostCSS is configured for v4
Get-Content postcss.config.mjs | Select-String "@tailwindcss/postcss"
# Verify PostCSS plugin is present

# Check if CSS import is correct
Get-Content src/app/globals.css | Select-String "@import.*tailwindcss"
# Should show v4 import syntax

# Rebuild the project
npm run dev
# Restart development server
```

### Problem: Large Bundle Size
**Symptoms**: 
- Slow page loads
- Large CSS file sizes

**Solution**: 
```powershell
# Verify v4 configuration doesn't include unused styles
Get-Content src/app/globals.css | Select-String "@theme"
# Review design tokens for necessity

# Check for unused custom CSS
Get-Content src/app/globals.css | Select-String "@layer"
# Review custom component styles for necessity
```

### Problem: Inconsistent Component Styling
**Symptoms**: 
- Components look different across pages
- Styling conflicts between components

**Solution**: 
```powershell
# Create component style guide
# Use consistent base classes for similar components
# Follow the class organization standards above
```

## Success Criteria

The rule implementation is complete when:
- [x] All components use utility-first Tailwind approach
- [x] Class organization follows established order convention
- [x] Responsive design consistently uses mobile-first approach
- [x] Color usage follows semantic naming patterns
- [x] Spacing maintains consistent scale throughout the application
- [x] ShadCN installed and configured with Tailwind v4
- [x] Core ShadCN components available in src/components/ui/
- [x] ShadCN components styled with project @theme tokens

## Examples

✅ **Correct Implementation:**
```typescript
// Card component with proper Tailwind patterns
export const Card: React.FC<{ children: React.ReactNode; className?: string }> = ({ 
  children, 
  className = '' 
}) => {
  return (
    <div className={`
      flex flex-col
      w-full max-w-md
      p-6
      bg-white
      border border-gray-200 rounded-lg
      shadow-sm
      hover:shadow-md
      transition-shadow duration-200
      ${className}
    `}>
      {children}
    </div>
  );
};

// Usage with responsive design
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4 md:gap-6">
  <Card>
    <h3 className="text-lg md:text-xl font-semibold text-gray-900 mb-2">
      Card Title
    </h3>
    <p className="text-sm md:text-base text-gray-600">
      Card description with responsive text sizing.
    </p>
  </Card>
</div>
```

❌ **Incorrect Implementation:**
```typescript
// Mixing custom CSS with Tailwind utilities
export const Card: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  return (
    <div style={{ padding: '24px', backgroundColor: '#ffffff' }} 
         className="shadow-md border-gray-200">
      {children}
    </div>
  );
};

// Poor class organization and non-mobile-first responsive design
<div className="hover:shadow-md border text-gray-900 lg:grid-cols-3 bg-white transition-shadow md:grid-cols-2 shadow-sm grid-cols-1 rounded-lg p-6 grid border-gray-200 gap-4">
```

---

**Note**: This rule ensures consistent utility-first styling patterns that maintain scalability and readability while leveraging Tailwind CSS's design system effectively.

