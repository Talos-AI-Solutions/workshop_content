---
description: Validates Next.js project structure, configuration, and standards compliance
allowed-tools: Bash(npm run build:*), Bash(npx tsc:*), Bash(npm run dev -- --port 3001:*), Bash(next dev --port 3001:*), Bash(kill:*), Bash(pkill:*)
---

# Next.js Project Validation

Perform a comprehensive validation of the Next.js project structure, configuration, and standards compliance.

## Validation Steps

### 1. Project Type Verification
First, verify this is a Next.js project by checking @package.json for Next.js dependencies. If no Next.js dependency is found, report that this doesn't appear to be a Next.js project.

### 2. Next.js Configuration
Check for next.config.ts or next.config.js and validate:
- ✅ **Docker deployment**: Look for `output: "standalone"` configuration (required for Docker)
- ⚠️ **Missing config**: Report if no next.config file exists

### 3. TypeScript Configuration  
If tsconfig.json exists:
- ✅ **TypeScript setup**: Confirm TypeScript configuration is present
- ✅ **Strict mode**: Check if `"strict": true` is enabled
- ⚠️ **Recommendations**: Suggest enabling strict mode if disabled

### 4. App Router Structure
Validate the App Router implementation:
- ✅ **Structure**: Look for `src/app/` or `app/` directory
- ✅ **Required files**: Verify presence of:
  - `layout.tsx` (root layout - required)
  - `page.tsx` (root page - required)  
  - `globals.css` (global styles)
- ❌ **Missing structure**: Report if App Router structure is not found

### 5. Tailwind CSS Setup
Check Tailwind CSS v4 configuration:
- ✅ **Dependency**: Verify tailwindcss in @package.json
- ✅ **V4 import**: Look for `@import "tailwindcss"` in globals.css
- ✅ **PostCSS**: Check for `@tailwindcss/postcss` plugin in postcss.config.mjs
- ⚠️ **V3 syntax**: Alert if using old `@tailwind` directives

### 6. Development Tools
Validate development tooling:
- ✅ **ESLint**: Check for ESLint in dependencies
- ✅ **Prettier**: Check for Prettier in dependencies
- ⚠️ **Missing tools**: Recommend adding if not found

### 7. Build Validation
Test the build process:
- Run `npm run build` to verify the project builds successfully
- ✅ **Success**: Confirm if build completes without errors
- ❌ **Failure**: Report build failures and suggest running build command for details

### 8. Development Server Validation (Optional)
If development server testing is needed:
- **Port Configuration**: MUST use port 3001 to avoid conflicts with Docker (port 3000)
- **Command**: Use `npm run dev -- --port 3001` or `next dev --port 3001`
- **Cleanup Required**: ALWAYS stop the dev server when validation is complete
- **Process Management**: Use process IDs to ensure proper cleanup
- **Timeout**: Limit dev server testing to maximum 30 seconds
- **Cleanup Commands**: 
  - Use `kill <PID>` for specific process termination
  - Use `pkill -f "next dev"` or `pkill -f "npm run dev"` for cleanup by process name
  - Verify termination with `ps aux | grep next` or similar
- ⚠️ **Never leave running**: Dev servers must be terminated after validation
- ❌ **Port conflicts**: Never start on port 3000 (reserved for Docker)

## Reporting Format
Use emoji indicators:
- ✅ for passed validations
- ⚠️ for warnings and recommendations  
- ❌ for errors and missing requirements

## Reference Documentation
For detailed standards, reference:
- Docker: .claude/context/nextjs_frontend/nextjs-docker-standards.md
- Performance: .claude/context/nextjs_frontend/nextjs-performance-standards.md
- Structure: .claude/context/nextjs_frontend/nextjs-structure-standards.md
- Tailwind: .claude/context/nextjs_frontend/ui-tailwind-css-standards.md