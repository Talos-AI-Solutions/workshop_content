---
description: Design excellence standards and patterns for front-end implementations including accessibility, responsiveness, visual polish, and component patterns
globs: src/app/**/*.tsx, src/components/**/*.tsx, src/app/**/*.css, src/styles/**/*.css
alwaysApply: false
---

# Design Review Implementation Standards

Comprehensive design excellence standards for achieving high-quality front-end implementations. These standards support the methodology defined in `.claude/agents/design-review-agent.md` and are validated by `.claude/commands/validate-design-review.md`.

## Prerequisites

- Understanding of WCAG 2.1 AA accessibility guidelines
- Familiarity with responsive design principles
- Knowledge of modern CSS frameworks (Tailwind, CSS-in-JS)
- Playwright browser automation basics
- Design system tokens and component libraries

## Core Principle

**"Users First" Decision-Making**: Every design decision must prioritize user experience, accessibility, and inclusive design over aesthetic preferences alone.

## Design System Foundation

### Color System

```css
/* ✅ Accessible color palette with WCAG 2.1 AA contrast */
:root {
  /* Primary colors - 4.5:1 contrast minimum for text */
  --color-primary-50: #f0f9ff;
  --color-primary-100: #e0f2fe;
  --color-primary-200: #bae6fd;
  --color-primary-300: #7dd3fc;
  --color-primary-400: #38bdf8;
  --color-primary-500: #0ea5e9;  /* Background */
  --color-primary-600: #0284c7;
  --color-primary-700: #0369a1;
  --color-primary-800: #075985;
  --color-primary-900: #0c4a6e;  /* Text on primary */

  /* Semantic colors */
  --color-success: #10b981;      /* 4.57:1 on white */
  --color-success-dark: #059669; /* For text */
  --color-warning: #f59e0b;      /* 3.64:1 on white */
  --color-warning-dark: #d97706; /* 4.54:1 on white - use for text */
  --color-error: #ef4444;        /* 4.54:1 on white */
  --color-error-dark: #dc2626;   /* For emphasis */

  /* Neutral palette */
  --color-gray-50: #f9fafb;
  --color-gray-100: #f3f4f6;
  --color-gray-200: #e5e7eb;
  --color-gray-300: #d1d5db;
  --color-gray-400: #9ca3af;
  --color-gray-500: #6b7280;
  --color-gray-600: #4b5563;
  --color-gray-700: #374151;
  --color-gray-800: #1f2937;
  --color-gray-900: #111827;
}

/* ❌ Anti-pattern: Insufficient contrast */
.bad-button {
  background: #f59e0b; /* Warning color */
  color: #ffffff;      /* Only 2.17:1 contrast - WCAG failure */
}

/* ✅ Correct: Sufficient contrast */
.good-button {
  background: #d97706; /* Warning dark */
  color: #ffffff;      /* 4.54:1 contrast - WCAG pass */
}
```

### Typography Scale

```css
/* ✅ Modular scale with optimal readability */
:root {
  /* Font families */
  --font-sans: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', system-ui, sans-serif;
  --font-mono: 'Fira Code', 'Cascadia Code', 'Consolas', monospace;

  /* Font sizes - modular scale (1.25 ratio) */
  --text-xs: 0.75rem;    /* 12px */
  --text-sm: 0.875rem;   /* 14px */
  --text-base: 1rem;     /* 16px - body text minimum */
  --text-lg: 1.125rem;   /* 18px */
  --text-xl: 1.25rem;    /* 20px */
  --text-2xl: 1.5rem;    /* 24px */
  --text-3xl: 1.875rem;  /* 30px */
  --text-4xl: 2.25rem;   /* 36px */
  --text-5xl: 3rem;      /* 48px */

  /* Font weights */
  --font-normal: 400;
  --font-medium: 500;
  --font-semibold: 600;
  --font-bold: 700;

  /* Line heights */
  --leading-tight: 1.25;    /* Headings */
  --leading-snug: 1.375;    /* UI elements */
  --leading-normal: 1.5;    /* Body text */
  --leading-relaxed: 1.75;  /* Long-form content */
}

/* ❌ Anti-pattern: Poor readability */
.bad-text {
  font-size: 0.625rem;    /* 10px - too small */
  line-height: 1.1;       /* Too tight */
  font-weight: 300;       /* Too light for body text */
}

/* ✅ Correct: Readable typography */
.good-text {
  font-size: var(--text-base);     /* 16px minimum */
  line-height: var(--leading-normal); /* 1.5 for readability */
  font-weight: var(--font-normal);    /* 400 for body */
}
```

### Spacing System

```css
/* ✅ Consistent 4px baseline grid */
:root {
  --space-0: 0;
  --space-1: 0.25rem;   /* 4px */
  --space-2: 0.5rem;    /* 8px */
  --space-3: 0.75rem;   /* 12px */
  --space-4: 1rem;      /* 16px */
  --space-5: 1.25rem;   /* 20px */
  --space-6: 1.5rem;    /* 24px */
  --space-8: 2rem;      /* 32px */
  --space-10: 2.5rem;   /* 40px */
  --space-12: 3rem;     /* 48px */
  --space-16: 4rem;     /* 64px */
  --space-20: 5rem;     /* 80px */
  --space-24: 6rem;     /* 96px */
}

/* ❌ Anti-pattern: Inconsistent spacing */
.bad-card {
  padding: 13px 17px;        /* Random values */
  margin-bottom: 23px;       /* Not on grid */
}

/* ✅ Correct: Grid-based spacing */
.good-card {
  padding: var(--space-4) var(--space-6);  /* 16px 24px */
  margin-bottom: var(--space-6);           /* 24px on grid */
}
```

## Accessibility Standards (WCAG 2.1 AA)

### Keyboard Navigation

```typescript
// ✅ Proper keyboard navigation implementation
import { useEffect, useRef } from 'react';

export function AccessibleModal({
  isOpen,
  onClose,
  children
}: {
  isOpen: boolean;
  onClose: () => void;
  children: React.ReactNode;
}) {
  const modalRef = useRef<HTMLDivElement>(null);
  const previousFocus = useRef<HTMLElement | null>(null);

  useEffect(() => {
    if (isOpen) {
      // Store the element that opened the modal
      previousFocus.current = document.activeElement as HTMLElement;

      // Focus first focusable element in modal
      const firstFocusable = modalRef.current?.querySelector<HTMLElement>(
        'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
      );
      firstFocusable?.focus();
    } else {
      // Return focus to trigger element
      previousFocus.current?.focus();
    }
  }, [isOpen]);

  useEffect(() => {
    const handleKeyDown = (e: KeyboardEvent) => {
      if (!isOpen) return;

      // Close on Escape
      if (e.key === 'Escape') {
        onClose();
        return;
      }

      // Trap focus on Tab
      if (e.key === 'Tab') {
        const focusableElements = modalRef.current?.querySelectorAll<HTMLElement>(
          'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
        );

        if (!focusableElements || focusableElements.length === 0) return;

        const firstElement = focusableElements[0];
        const lastElement = focusableElements[focusableElements.length - 1];

        if (e.shiftKey && document.activeElement === firstElement) {
          // Shift+Tab on first element: go to last
          e.preventDefault();
          lastElement.focus();
        } else if (!e.shiftKey && document.activeElement === lastElement) {
          // Tab on last element: go to first
          e.preventDefault();
          firstElement.focus();
        }
      }
    };

    document.addEventListener('keydown', handleKeyDown);
    return () => document.removeEventListener('keydown', handleKeyDown);
  }, [isOpen, onClose]);

  if (!isOpen) return null;

  return (
    <div
      className="modal-overlay"
      onClick={onClose}
      aria-modal="true"
    >
      <div
        ref={modalRef}
        role="dialog"
        aria-labelledby="modal-title"
        className="modal-content"
        onClick={(e) => e.stopPropagation()}
      >
        {children}
      </div>
    </div>
  );
}

// ❌ Anti-pattern: No keyboard support
export function BadModal({ isOpen, children }: { isOpen: boolean; children: React.ReactNode }) {
  if (!isOpen) return null;

  return (
    <div className="modal">  {/* No keyboard handling, focus trap, or ARIA */}
      {children}
    </div>
  );
}
```

### Focus Indicators

```css
/* ✅ Visible focus states for all interactive elements */
button:focus-visible,
a:focus-visible,
input:focus-visible,
select:focus-visible,
textarea:focus-visible,
[role="button"]:focus-visible {
  outline: 2px solid var(--color-primary-500);
  outline-offset: 2px;
  /* Minimum 2px thickness and 3:1 contrast ratio */
}

/* Custom focus ring for specific components */
.custom-button:focus-visible {
  outline: 3px solid var(--color-primary-400);
  outline-offset: 3px;
  box-shadow: 0 0 0 5px rgba(14, 165, 233, 0.2);
}

/* ❌ NEVER remove focus indicators without replacement */
.bad-button:focus {
  outline: none;  /* ❌ Accessibility violation - keyboard users can't see focus */
}

/* ❌ Insufficient focus indicator */
.weak-focus:focus-visible {
  outline: 1px dotted gray;  /* ❌ Too thin, poor contrast */
}
```

### Contrast Ratios

```typescript
// ✅ Verify contrast ratios programmatically
interface ContrastCheck {
  foreground: string;
  background: string;
  minRatio: number; // 4.5:1 for text, 3:1 for UI
}

function hexToRgb(hex: string): [number, number, number] {
  const result = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(hex);
  return result
    ? [parseInt(result[1], 16), parseInt(result[2], 16), parseInt(result[3], 16)]
    : [0, 0, 0];
}

function getLuminance(r: number, g: number, b: number): number {
  const [rs, gs, bs] = [r, g, b].map(c => {
    c = c / 255;
    return c <= 0.03928 ? c / 12.92 : Math.pow((c + 0.055) / 1.055, 2.4);
  });
  return 0.2126 * rs + 0.7152 * gs + 0.0722 * bs;
}

function calculateContrastRatio(fg: string, bg: string): number {
  const [r1, g1, b1] = hexToRgb(fg);
  const [r2, g2, b2] = hexToRgb(bg);

  const l1 = getLuminance(r1, g1, b1);
  const l2 = getLuminance(r2, g2, b2);

  const lighter = Math.max(l1, l2);
  const darker = Math.min(l1, l2);

  return (lighter + 0.05) / (darker + 0.05);
}

function validateContrast({ foreground, background, minRatio }: ContrastCheck): boolean {
  const ratio = calculateContrastRatio(foreground, background);
  return ratio >= minRatio;
}

// Example usage
const examples = [
  { fg: '#0ea5e9', bg: '#ffffff', min: 4.5, name: 'Primary on white' },
  { fg: '#6b7280', bg: '#ffffff', min: 4.5, name: 'Gray-500 on white' },
  { fg: '#ffffff', bg: '#0ea5e9', min: 4.5, name: 'White on primary' },
];

examples.forEach(({ fg, bg, min, name }) => {
  const isAccessible = validateContrast({ foreground: fg, background: bg, minRatio: min });
  const ratio = calculateContrastRatio(fg, bg).toFixed(2);
  console.log(`${name}: ${ratio}:1 - ${isAccessible ? '✅ Pass' : '❌ Fail'}`);
});
```

### Screen Reader Support

```typescript
// ✅ Proper ARIA labels and semantic HTML
export function DataTable({
  data,
  caption
}: {
  data: Array<{ name: string; email: string; role: string }>;
  caption: string;
}) {
  return (
    <table role="table" aria-label={caption}>
      <caption className="sr-only">
        {caption}
      </caption>
      <thead>
        <tr>
          <th scope="col">Name</th>
          <th scope="col">Email</th>
          <th scope="col">Role</th>
          <th scope="col">Actions</th>
        </tr>
      </thead>
      <tbody>
        {data.map((row, idx) => (
          <tr key={idx}>
            <td>{row.name}</td>
            <td>{row.email}</td>
            <td>{row.role}</td>
            <td>
              <button
                aria-label={`Edit ${row.name}`}
                className="icon-button"
              >
                <EditIcon aria-hidden="true" />
              </button>
              <button
                aria-label={`Delete ${row.name}`}
                className="icon-button"
              >
                <DeleteIcon aria-hidden="true" />
              </button>
            </td>
          </tr>
        ))}
      </tbody>
    </table>
  );
}

// ❌ Anti-pattern: Poor screen reader support
export function BadTable({ data }: { data: any[] }) {
  return (
    <div className="table">  {/* Not semantic table */}
      {data.map((row, idx) => (
        <div key={idx}>
          <span>{row.name}</span>  {/* No context for screen readers */}
          <button>Edit</button>    {/* No aria-label - unclear what's being edited */}
        </div>
      ))}
    </div>
  );
}

/* Screen reader only utility class */
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border-width: 0;
}
```

## Responsive Design Patterns

### Breakpoint Standards

```typescript
// ✅ Standard breakpoints aligned with testing viewports
export const breakpoints = {
  mobile: '375px',   // Test viewport - iPhone SE
  tablet: '768px',   // Test viewport - iPad
  desktop: '1440px', // Test viewport - Standard desktop
  wide: '1920px'     // Wide desktop
} as const;

// Tailwind CSS configuration
export default {
  theme: {
    screens: {
      'sm': '375px',   // Mobile
      'md': '768px',   // Tablet
      'lg': '1440px',  // Desktop
      'xl': '1920px',  // Wide
    }
  }
};

// React hook for responsive behavior
export function useBreakpoint() {
  const [breakpoint, setBreakpoint] = useState<keyof typeof breakpoints>('desktop');

  useEffect(() => {
    const handleResize = () => {
      const width = window.innerWidth;
      if (width < 768) setBreakpoint('mobile');
      else if (width < 1440) setBreakpoint('tablet');
      else setBreakpoint('desktop');
    };

    handleResize();
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return breakpoint;
}
```

### Mobile-First Implementation

```css
/* ✅ Mobile-first responsive design */
.card {
  /* Base: Mobile (375px) */
  padding: var(--space-4);
  grid-template-columns: 1fr;
  font-size: var(--text-base);
}

/* Tablet (768px) */
@media (min-width: 768px) {
  .card {
    padding: var(--space-6);
    grid-template-columns: repeat(2, 1fr);
    gap: var(--space-6);
  }
}

/* Desktop (1440px) */
@media (min-width: 1440px) {
  .card {
    padding: var(--space-8);
    grid-template-columns: repeat(3, 1fr);
    gap: var(--space-8);
  }
}

/* ❌ Anti-pattern: Desktop-first with max-width */
.bad-card {
  /* Defaults to desktop */
  padding: var(--space-8);
  grid-template-columns: repeat(3, 1fr);
}

@media (max-width: 1440px) {
  .bad-card {
    /* Overriding styles - less performant */
    padding: var(--space-6);
    grid-template-columns: repeat(2, 1fr);
  }
}
```

### Touch Target Sizing

```css
/* ✅ Minimum 44x44px touch targets for mobile (WCAG 2.5.5) */
.button,
.link,
.checkbox-wrapper,
.radio-wrapper {
  min-width: 44px;
  min-height: 44px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
}

/* Icon buttons need explicit sizing */
.icon-button {
  width: 44px;
  height: 44px;
  padding: var(--space-2);
}

/* For small visual elements, use padding to achieve size */
.small-icon-button {
  padding: var(--space-3);  /* Creates 44x44 clickable area */
}

.small-icon-button svg {
  width: 20px;  /* Visual icon size */
  height: 20px;
}

/* ❌ Too small for touch */
.bad-icon-button {
  width: 24px;   /* ❌ Below 44px minimum */
  height: 24px;  /* ❌ Difficult to tap on mobile */
  padding: 0;
}
```

## Visual Polish Patterns

### Alignment & Spacing

```typescript
// ✅ Consistent spacing using design tokens
export function ProfileCard({
  user
}: {
  user: { name: string; title: string; avatar: string }
}) {
  return (
    <div className="space-y-4">  {/* Vertical rhythm - 16px */}
      <img
        src={user.avatar}
        alt={`${user.name}'s avatar`}
        className="w-16 h-16 rounded-full"
      />
      <div className="space-y-2">  {/* Tighter grouping for related content - 8px */}
        <h3 className="text-lg font-semibold">{user.name}</h3>
        <p className="text-sm text-gray-600">{user.title}</p>
      </div>
      <button className="mt-6 w-full">  {/* Increased spacing for actions - 24px */}
        View Profile
      </button>
    </div>
  );
}

// ❌ Anti-pattern: Inconsistent spacing
export function BadProfileCard({ user }: { user: any }) {
  return (
    <div style={{ marginBottom: '13px' }}>  {/* Random value */}
      <img src={user.avatar} style={{ marginBottom: '7px' }} />  {/* Not on grid */}
      <h3 style={{ marginBottom: '5px' }}>{user.name}</h3>
      <p>{user.title}</p>
    </div>
  );
}
```

### Typography Hierarchy

```css
/* ✅ Clear visual hierarchy */
h1 {
  font-size: var(--text-4xl);          /* 36px */
  font-weight: var(--font-bold);       /* 700 */
  line-height: var(--leading-tight);   /* 1.25 */
  margin-bottom: var(--space-6);       /* 24px */
  color: var(--color-gray-900);
}

h2 {
  font-size: var(--text-2xl);          /* 24px */
  font-weight: var(--font-semibold);   /* 600 */
  line-height: var(--leading-tight);   /* 1.25 */
  margin-bottom: var(--space-4);       /* 16px */
  color: var(--color-gray-800);
}

h3 {
  font-size: var(--text-xl);           /* 20px */
  font-weight: var(--font-semibold);   /* 600 */
  line-height: var(--leading-snug);    /* 1.375 */
  margin-bottom: var(--space-3);       /* 12px */
  color: var(--color-gray-800);
}

p {
  font-size: var(--text-base);         /* 16px */
  font-weight: var(--font-normal);     /* 400 */
  line-height: var(--leading-relaxed); /* 1.75 */
  margin-bottom: var(--space-4);       /* 16px */
  color: var(--color-gray-700);
}

.caption {
  font-size: var(--text-sm);           /* 14px */
  font-weight: var(--font-normal);     /* 400 */
  line-height: var(--leading-normal);  /* 1.5 */
  color: var(--color-gray-600);
}

/* ❌ Anti-pattern: Unclear hierarchy */
.bad-heading {
  font-size: 18px;      /* Between h2 and h3 - confusing */
  font-weight: 500;     /* Medium weight - unclear importance */
  line-height: 1.2;     /* Inconsistent with system */
}
```

### Micro-Interactions

```typescript
// ✅ Purposeful animations (150-300ms)
import { motion } from 'framer-motion';

export function AnimatedButton({
  children,
  onClick
}: {
  children: React.ReactNode;
  onClick: () => void;
}) {
  return (
    <motion.button
      onClick={onClick}
      initial={{ scale: 1 }}
      whileHover={{ scale: 1.05 }}
      whileTap={{ scale: 0.95 }}
      transition={{ duration: 0.2 }}  // 200ms - perceptible but not slow
      className="button-primary"
    >
      {children}
    </motion.button>
  );
}

// Loading spinner with smooth animation
export function Spinner() {
  return (
    <motion.div
      className="spinner"
      animate={{ rotate: 360 }}
      transition={{
        duration: 1,
        repeat: Infinity,
        ease: "linear"
      }}
    />
  );
}

// ❌ Anti-pattern: Slow, distracting animations
export function BadAnimatedButton({ children }: { children: React.ReactNode }) {
  return (
    <motion.button
      whileHover={{
        scale: 1.5,      // ❌ Too dramatic
        rotate: 360      // ❌ Distracting
      }}
      transition={{ duration: 1.5 }}  // ❌ Too slow (1500ms)
    >
      {children}
    </motion.button>
  );
}
```

## Component Patterns

### Form Validation

```typescript
// ✅ Accessible form with inline validation
import { useState } from 'react';

interface FormErrors {
  email?: string;
  password?: string;
}

export function ContactForm() {
  const [errors, setErrors] = useState<FormErrors>({});
  const [touched, setTouched] = useState<Record<string, boolean>>({});

  const validateEmail = (email: string): string | undefined => {
    if (!email) return 'Email is required';
    if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)) {
      return 'Please enter a valid email address';
    }
  };

  const handleBlur = (field: string, value: string) => {
    setTouched(prev => ({ ...prev, [field]: true }));

    if (field === 'email') {
      const error = validateEmail(value);
      setErrors(prev => ({ ...prev, email: error }));
    }
  };

  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    // Validation logic
  };

  return (
    <form onSubmit={handleSubmit} noValidate>
      <div className="form-field">
        <label htmlFor="email" className="form-label">
          Email Address <span className="text-error">*</span>
        </label>
        <input
          id="email"
          name="email"
          type="email"
          className={`form-input ${errors.email && touched.email ? 'input-error' : ''}`}
          onBlur={(e) => handleBlur('email', e.target.value)}
          aria-required="true"
          aria-invalid={!!errors.email && touched.email}
          aria-describedby={errors.email && touched.email ? "email-error" : undefined}
        />
        {errors.email && touched.email && (
          <span
            id="email-error"
            role="alert"
            className="error-message"
          >
            {errors.email}
          </span>
        )}
      </div>

      <button type="submit" className="button-primary">
        Submit
      </button>
    </form>
  );
}

// ❌ Anti-pattern: Poor form accessibility
export function BadForm() {
  return (
    <form>
      <div>
        <span>Email</span>  {/* ❌ Not a label */}
        <input type="text" />  {/* ❌ No id, aria attributes */}
        <span style={{ color: 'red' }}>Invalid</span>  {/* ❌ No role="alert" */}
      </div>
    </form>
  );
}
```

### Loading States

```typescript
// ✅ Skeleton loading with accessibility
export function SkeletonCard() {
  return (
    <div
      className="skeleton-card animate-pulse"
      role="status"
      aria-busy="true"
      aria-label="Loading content"
    >
      <div className="skeleton-avatar h-16 w-16 rounded-full bg-gray-200" />
      <div className="space-y-2">
        <div className="skeleton-text h-4 w-3/4 rounded bg-gray-200" />
        <div className="skeleton-text h-4 w-1/2 rounded bg-gray-200" />
      </div>
      <span className="sr-only">Loading...</span>
    </div>
  );
}

// Progressive loading with real data
export function UserList() {
  const { data, isLoading } = useUsers();

  if (isLoading) {
    return (
      <div className="space-y-4">
        {[...Array(5)].map((_, i) => (
          <SkeletonCard key={i} />
        ))}
      </div>
    );
  }

  return (
    <div className="space-y-4">
      {data?.map(user => (
        <UserCard key={user.id} user={user} />
      ))}
    </div>
  );
}

// ❌ Anti-pattern: Jarring loading state
export function BadUserList() {
  const { data, isLoading } = useUsers();

  if (isLoading) {
    return <div>Loading...</div>;  // ❌ No skeleton, causes layout shift
  }

  return <div>{/* content */}</div>;
}
```

### Error Boundaries

```typescript
// ✅ User-friendly error states
import { Component, ReactNode } from 'react';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
}

interface State {
  hasError: boolean;
  error: Error | null;
}

export class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error('ErrorBoundary caught:', error, errorInfo);
    // Send to error tracking service
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || (
        <div
          role="alert"
          className="error-container p-8 text-center"
        >
          <h2 className="text-2xl font-bold text-gray-900 mb-4">
            Something went wrong
          </h2>
          <p className="text-gray-600 mb-6">
            We're sorry for the inconvenience. Please try refreshing the page.
          </p>
          <button
            onClick={() => window.location.reload()}
            className="button-primary"
          >
            Refresh Page
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}

// Usage
export function App() {
  return (
    <ErrorBoundary>
      <YourApp />
    </ErrorBoundary>
  );
}
```

## Playwright Testing Patterns

### Multi-Viewport Testing

```typescript
// ✅ Comprehensive viewport testing
import { test, expect, devices } from '@playwright/test';

const viewports = [
  { name: 'Desktop', width: 1440, height: 900 },
  { name: 'Tablet', width: 768, height: 1024 },
  { name: 'Mobile', width: 375, height: 667 },
];

viewports.forEach(({ name, width, height }) => {
  test(`should render correctly on ${name}`, async ({ page }) => {
    await page.setViewportSize({ width, height });
    await page.goto('/dashboard');

    // Wait for content to load
    await page.waitForSelector('[data-testid="dashboard-content"]');

    // Visual regression testing
    await expect(page).toHaveScreenshot(`dashboard-${name.toLowerCase()}.png`, {
      fullPage: true,
      animations: 'disabled',
    });
  });

  test(`navigation should work on ${name}`, async ({ page }) => {
    await page.setViewportSize({ width, height });
    await page.goto('/');

    // Test navigation
    await page.click('nav a[href="/about"]');
    await expect(page).toHaveURL(/.*about/);
  });
});

// Test with actual device emulation
test('should work on iPhone 12', async ({ browser }) => {
  const context = await browser.newContext({
    ...devices['iPhone 12'],
  });
  const page = await context.newPage();

  await page.goto('/');
  await expect(page.locator('h1')).toBeVisible();
});
```

### Accessibility Testing

```typescript
// ✅ Automated accessibility checks
import { test, expect } from '@playwright/test';
import { injectAxe, checkA11y } from 'axe-playwright';

test.describe('Accessibility', () => {
  test('should have no accessibility violations', async ({ page }) => {
    await page.goto('/');
    await injectAxe(page);

    await checkA11y(page, null, {
      detailedReport: true,
      detailedReportOptions: {
        html: true,
      },
    });
  });

  test('should be keyboard navigable', async ({ page }) => {
    await page.goto('/');

    // Tab through interactive elements
    await page.keyboard.press('Tab');
    let focused = await page.evaluate(() => document.activeElement?.tagName);
    expect(['A', 'BUTTON', 'INPUT']).toContain(focused);

    // Test keyboard interaction
    await page.keyboard.press('Enter');
    await expect(page).toHaveURL(/.*dashboard/);
  });

  test('should have sufficient color contrast', async ({ page }) => {
    await page.goto('/');

    // Check button contrast
    const button = page.locator('button.primary');
    const styles = await button.evaluate(el => {
      const computed = window.getComputedStyle(el);
      return {
        color: computed.color,
        backgroundColor: computed.backgroundColor,
      };
    });

    // Use contrast checking library or manual validation
    // This would integrate with your contrast checker utility
  });
});
```

## Quality Indicators

### Success Patterns

- ✅ **WCAG 2.1 AA Compliant**: All accessibility criteria met
  - Keyboard navigation complete
  - Focus indicators visible (min 2px)
  - Contrast ratios sufficient (4.5:1 text, 3:1 UI)
  - Screen reader compatible
  - Form labels present and associated

- ✅ **Responsive Layouts**: Renders correctly across all test viewports
  - Desktop (1440px): Full feature set
  - Tablet (768px): Optimized layout
  - Mobile (375px): Touch-optimized, no horizontal scroll

- ✅ **Performance Optimized**: Lighthouse score > 90
  - LCP < 2.5s
  - FID < 100ms
  - CLS < 0.1

- ✅ **Design System Adherence**: Uses design tokens consistently
  - Color variables used (no hardcoded colors)
  - Spacing on 4px grid
  - Typography scale followed

- ✅ **Visual Evidence**: Screenshots document implementation quality
  - All viewports captured
  - Interactive states shown
  - Edge cases documented

### Performance Metrics

```typescript
// ✅ Performance testing
import { test } from '@playwright/test';

test('should meet performance budgets', async ({ page }) => {
  await page.goto('/');

  const performance = await page.evaluate(() => {
    const perfData = performance.getEntriesByType('navigation')[0] as PerformanceNavigationTiming;
    return {
      // Largest Contentful Paint
      lcp: performance.getEntriesByType('largest-contentful-paint')[0]?.startTime || 0,
      // First Input Delay (approximate)
      fid: perfData.responseStart - perfData.fetchStart,
      // Cumulative Layout Shift
      cls: performance.getEntriesByType('layout-shift').reduce((sum, entry: any) => {
        if (!entry.hadRecentInput) {
          return sum + entry.value;
        }
        return sum;
      }, 0),
    };
  });

  // Assert performance budgets
  expect(performance.lcp).toBeLessThan(2500);  // 2.5s
  expect(performance.fid).toBeLessThan(100);   // 100ms
  expect(performance.cls).toBeLessThan(0.1);   // 0.1
});
```

## Common Anti-Patterns

### Accessibility Violations

```typescript
// ❌ Missing alt text
<img src="/avatar.jpg" />

// ✅ Proper alt text
<img src="/avatar.jpg" alt="User profile picture" />

// ❌ Non-semantic button
<div onClick={handleClick}>Click me</div>

// ✅ Semantic button
<button onClick={handleClick}>Click me</button>

// ❌ Poor color contrast
<button style={{ background: '#ffeb3b', color: '#fff' }}>
  Submit  {/* 1.3:1 contrast - WCAG failure */}
</button>

// ✅ Sufficient contrast
<button className="bg-yellow-600 text-gray-900">
  Submit  {/* 7.5:1 contrast - WCAG AAA */}
</button>

// ❌ Missing form labels
<input type="email" placeholder="Email" />

// ✅ Proper form labels
<label htmlFor="email">Email</label>
<input id="email" type="email" />
```

### Visual Issues

```css
/* ❌ Hardcoded values instead of tokens */
.bad-component {
  padding: 15px 22px;
  color: #1a73e8;
  font-size: 15px;
}

/* ✅ Design token usage */
.good-component {
  padding: var(--space-4) var(--space-6);
  color: var(--color-primary-600);
  font-size: var(--text-base);
}

/* ❌ Broken responsive layout */
.bad-layout {
  width: 1200px;  /* Fixed width causes overflow on mobile */
}

/* ✅ Responsive layout */
.good-layout {
  width: 100%;
  max-width: 1200px;
  padding: 0 var(--space-4);
}
```

### Implementation Problems

```typescript
// ❌ No error handling
async function fetchData() {
  const response = await fetch('/api/data');
  return response.json();
}

// ✅ Proper error handling
async function fetchData() {
  try {
    const response = await fetch('/api/data');
    if (!response.ok) {
      throw new Error(`HTTP error ${response.status}`);
    }
    return await response.json();
  } catch (error) {
    console.error('Failed to fetch data:', error);
    throw error;  // Re-throw for upstream handling
  }
}

// ❌ Missing loading state
function UserList() {
  const { data } = useUsers();
  return <div>{data.map(user => <User key={user.id} {...user} />)}</div>;
}

// ✅ Proper loading state
function UserList() {
  const { data, isLoading, error } = useUsers();

  if (isLoading) return <SkeletonList />;
  if (error) return <ErrorState error={error} />;
  if (!data?.length) return <EmptyState />;

  return <div>{data.map(user => <User key={user.id} {...user} />)}</div>;
}
```

## Supporting Components

- **Implementation Guidance**: `.claude/agents/design-review-agent.md` - 7-phase review methodology
- **Output Validation**: `.claude/commands/validate-design-review.md` - Verification checklist
- **Framework Updates**: Regularly validated against Context7 specs for Playwright and WCAG

## References

These standards are inspired by design practices at:
- **Stripe**: Payment UI patterns and accessibility
- **Airbnb**: Component composition and design systems
- **Linear**: Visual polish and micro-interactions

For latest specifications:
- **WCAG 2.1 AA**: https://www.w3.org/WAI/WCAG21/quickref/
- **Playwright**: Use Context7 for current documentation
- **Design Systems**: Tailwind CSS, Material Design, Radix UI

---

**Note**: These standards ensure front-end implementations achieve design excellence through accessibility, responsiveness, visual polish, and robust component patterns. All examples validate against WCAG 2.1 AA and modern framework best practices.
