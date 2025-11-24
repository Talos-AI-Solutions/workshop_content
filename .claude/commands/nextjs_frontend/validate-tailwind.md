---
description: Validates Tailwind CSS v4 implementation and utility-first patterns
allowed-tools: Bash(npm list tailwindcss:*), Bash(npm run build:*)
---

# Tailwind CSS Configuration Validation

Perform comprehensive validation of Tailwind CSS v4 implementation and utility-first design patterns.

## Validation Steps

### 1. Tailwind CSS Installation
First, verify Tailwind CSS is installed by checking @package.json for tailwindcss dependency. If not found, report that Tailwind CSS is not installed.

### 2. Version Detection
Check the installed Tailwind CSS version:
- ✅ **v4 detected**: Report if Tailwind CSS v4.x is installed (recommended)
- ⚠️ **v3 detected**: If v3.x is found, recommend upgrading to v4 for improved performance
- ⚠️ **Other version**: Report any other version found

### 3. PostCSS Configuration
Validate PostCSS setup for proper Tailwind integration:
- Check for postcss.config.mjs (preferred) or postcss.config.js
- ✅ **v4 plugin**: Look for `@tailwindcss/postcss` plugin (v4 requirement)
- ✅ **v3 plugin**: Confirm `tailwindcss` plugin for v3 installations
- ❌ **No config**: Report missing PostCSS configuration
- ⚠️ **Missing plugin**: Alert if Tailwind plugin not found in config

### 4. CSS Import Configuration
Examine global CSS files for proper Tailwind imports:
- Look for CSS files in: `src/app/globals.css`, `app/globals.css`, `styles/globals.css`, `src/styles/globals.css`
- For each CSS file found:
  - ✅ **v4 import**: Look for `@import "tailwindcss"` syntax
  - ⚠️ **v3 directives**: If using `@tailwind` directives, suggest upgrading to v4 import
  - ✅ **Theme config**: Check for `@theme` directive (v4 CSS-first feature)
  - ✅ **Component layer**: Look for `@layer components` customizations
  - ❌ **No imports**: Report if no Tailwind imports found

### 5. Project Structure Analysis
Validate component organization:
- Check for component directories: `src/components`, `components`, `src/app/_components`
- ✅ **Components found**: Report discovered component directories
- ✅ **UI components**: Look for `ui/` subdirectories for component organization
- ⚠️ **No structure**: Note if no component directories are found

### 6. Utility Class Usage Patterns
Analyze sample component files for Tailwind usage patterns:
- Find sample files matching: `*.tsx`, `*.ts`, `*.jsx`, `*.js` containing "component", "page", or "layout"
- For each file, check:
  - ⚠️ **Long classNames**: Look for className strings over 100 characters (suggest component variants)
  - ⚠️ **Inline styles**: Check for `style=` attributes (suggest Tailwind utilities instead)
- ✅ **Good patterns**: Report if no obvious organization issues found
- ⚠️ **No files**: Note if no component files found to analyze

### 7. Responsive Design Patterns
Validate mobile-first responsive design:
- In sample files, check for proper responsive patterns:
  - ⚠️ **Desktop-first**: Look for anti-patterns like `lg:...md:...sm:` (should be mobile-first)
  - ✅ **Mobile-first**: Confirm proper `sm: → md: → lg:` progression
- ✅ **Good responsive**: Report if no responsive design issues found

### 8. Build Integration Testing
Test Tailwind CSS build process:
- Run `npm run build` to verify Tailwind CSS works in build
- ✅ **Build success**: Confirm project builds successfully with Tailwind
- ❌ **Build failure**: Report build failures and suggest running build for details
- ⚠️ **npm unavailable**: Note if npm isn't available for testing

## Reporting Format
Use emoji indicators for clear status communication:
- ✅ for confirmed best practices and working configurations
- ⚠️ for recommendations, suggestions, and potential improvements
- ❌ for errors and missing required configurations

## Quick Improvements Guide
When issues are found, provide actionable recommendations:
- Upgrade to Tailwind CSS v4 for better performance
- Use `@import "tailwindcss"` syntax instead of `@tailwind` directives
- Implement `@theme` directive for design tokens
- Organize className patterns: Layout → Sizing → Spacing → Colors → Interactive
- Use mobile-first responsive design (sm: → md: → lg: progression)
- Extract long className strings to component variants
- Replace inline styles with Tailwind utilities

## CSS-First Configuration (v4)
For Tailwind v4 projects, validate CSS-first approach:
- Theme customization via `@theme` in CSS files
- Design tokens defined in CSS rather than config files
- PostCSS plugin configuration over traditional config.js

## Reference Documentation
For comprehensive Tailwind CSS standards:
- Complete guide: .claude/context/nextjs_frontend/ui-tailwind-css-standards.md