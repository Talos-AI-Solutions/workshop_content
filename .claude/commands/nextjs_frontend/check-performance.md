---
description: Runs comprehensive performance checks and optimization validation
allowed-tools: Bash(npm run build:*), Bash(find:*), Bash(du:*), Bash(ls:*)
---

# Performance Analysis and Optimization

Perform comprehensive performance analysis and optimization validation for web applications, with focus on Next.js performance best practices.

## Performance Analysis Steps

### 1. Project Verification
First, verify this is a web project by checking for @package.json. If not found, report this should be run from project root.

### 2. Bundle Analysis Setup
Check for bundle analysis capabilities:
- ✅ **Bundle Analyzer**: Look for `@next/bundle-analyzer` in @package.json
- ✅ **Analysis Script**: Check for `"analyze"` script in package.json
- ⚠️ **Missing Analyzer**: Recommend installing: `npm install --save-dev @next/bundle-analyzer`
- ⚠️ **Missing Script**: Suggest adding: `"analyze": "ANALYZE=true npm run build"`

### 3. Image Optimization Analysis
Examine component files for image usage patterns:
- Find sample component files (`.tsx`, `.ts`, `.jsx`, `.js`)
- For each file, check:
  - ✅ **next/image usage**: Look for `from 'next/image'` imports
  - ⚠️ **Regular img tags**: Find `<img` tags and suggest next/image
  - ✅ **Priority attributes**: Check for `priority` on above-fold images
- ✅ **Good optimization**: Report if no image issues found

### 4. Link Optimization Analysis
Validate navigation optimization:
- In component files, check for:
  - ✅ **next/link usage**: Look for `from 'next/link'` imports
  - ⚠️ **Regular anchors**: Find `<a href` tags and suggest next/link
  - ✅ **Prefetch config**: Check for `prefetch=` attributes
- ✅ **Good optimization**: Report if no link issues found

### 5. Font Optimization Assessment
Check font loading optimization:
- ✅ **next/font usage**: Look for `next/font` imports in components
- ⚠️ **External fonts**: Check layout files for `fonts.googleapis.com` links
- ⚠️ **Missing optimization**: Suggest using next/font for external fonts

### 6. Core Web Vitals Setup
Validate performance monitoring:
- ✅ **web-vitals package**: Check for web-vitals in @package.json
- ⚠️ **Missing package**: Recommend installing: `npm install web-vitals`
- Check layout files (`app/layout.tsx`, `src/app/layout.tsx`) for:
  - ✅ **Vitals reporting**: Look for `reportWebVitals`, `onCLS`, `onFID`, `onFCP`, `onLCP`, `onTTFB`
  - ⚠️ **No reporting**: Suggest adding Web Vitals reporting

### 7. Build Configuration Analysis
Examine build optimization settings:
- Check next.config.ts or next.config.js for:
  - ✅ **Compression**: Look for `compress` or `gzip` configuration
  - ⚠️ **Missing compression**: Recommend enabling compression
  - ✅ **Experimental features**: Check for `experimental` optimizations
  - ✅ **Bundle analyzer**: Look for `withBundleAnalyzer` integration

### 8. Build Performance Testing
Test actual build performance:
- Run `npm run build` and measure time
- ✅ **Excellent**: < 60 seconds
- ⚠️ **Acceptable**: < 120 seconds  
- ⚠️ **Slow**: > 120 seconds (suggest optimization)
- ❌ **Failed**: Report build failures

### 9. Bundle Size Analysis
After build completion, analyze output:
- Check `.next/static/chunks/` directory for:
  - ⚠️ **Large chunks**: Find files > 500KB and suggest code splitting
  - ✅ **Reasonable sizes**: Report if no excessively large chunks
- ✅ **Total size**: Report total static assets size from `.next/static`
- ⚠️ **No build**: Note if build output not found

## Performance Metrics Targets
Guide optimization efforts toward these targets:
- **Largest Contentful Paint (LCP)**: < 2.5 seconds
- **First Input Delay (FID)**: < 100 milliseconds  
- **Cumulative Layout Shift (CLS)**: < 0.1
- **Build time**: < 2 minutes for typical projects
- **Bundle chunks**: < 500KB per chunk

## Optimization Recommendations
When issues are found, provide specific guidance:

### Image Optimization
- Use `next/image` for all images with automatic optimization
- Add `priority` attribute for above-the-fold images
- Implement proper image sizing and responsive loading

### Navigation Optimization  
- Use `next/link` for all internal navigation
- Configure `prefetch` strategically for better UX
- Avoid regular `<a>` tags for internal routes

### Font Optimization
- Replace external font links with `next/font`
- Use font display strategies for better loading
- Preload critical fonts

### Bundle Optimization
- Enable bundle analysis with regular monitoring
- Implement code splitting for large chunks
- Use dynamic imports for heavy components
- Enable compression in Next.js config

### Monitoring Setup
- Install and configure web-vitals package
- Set up Core Web Vitals reporting
- Monitor performance metrics in production

## Reporting Format
Use clear status indicators:
- ✅ for optimized configurations and good performance
- ⚠️ for recommendations and potential improvements
- ❌ for errors and performance issues

## Reference Documentation
For detailed performance standards:
- Next.js Performance: .claude/context/nextjs_frontend/nextjs-performance-standards.md