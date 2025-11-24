---
description: When writing NextJS code and seeking guidance on optimal performance best practices
globs: 
alwaysApply: false
---
# NextJS Performance Standards

Essential performance optimization practices for NextJS 15 applications.

## Prerequisites

- NextJS 15+ with App Router
- TypeScript configured
- Windows PowerShell for commands

## Core Principle

**ALWAYS optimize for Web Core Vitals using NextJS built-in performance features.**

## Built-in Component Optimization

### Image Optimization
```typescript
// ✅ Correct - Use next/image
import Image from 'next/image';

export function Hero() {
  return (
    <Image
      src="/hero.jpg"
      alt="Hero image"
      width={800}
      height={600}
      priority // Above fold images
    />
  );
}

// ❌ Incorrect - Regular img tag
<img src="/hero.jpg" alt="Hero" />
```

### Link Prefetching
```typescript
// ✅ Correct - Use next/link
import Link from 'next/link';

export function Navigation() {
  return (
    <Link href="/dashboard" prefetch={true}>
      Dashboard
    </Link>
  );
}
```

### Font Optimization
```typescript
// ✅ Correct - Use next/font
import { Inter } from 'next/font/google';

const inter = Inter({ subsets: ['latin'] });

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body className={inter.className}>{children}</body>
    </html>
  );
}
```

## Bundle Analysis Setup

### Step 1: Install Bundle Analyzer
```powershell
npm install --save-dev @next/bundle-analyzer
```

### Step 2: Configure next.config.ts
```typescript
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
});

module.exports = withBundleAnalyzer({
  // Your Next.js config
});
```

### Step 3: Add Analysis Script
```json
// package.json
{
  "scripts": {
    "analyze": "ANALYZE=true npm run build"
  }
}
```

## Lazy Loading Implementation

```typescript
// ✅ Dynamic imports for code splitting
import dynamic from 'next/dynamic';

const DynamicComponent = dynamic(() => import('./HeavyComponent'), {
  loading: () => <p>Loading...</p>,
});

export function Page() {
  return (
    <div>
      <h1>Page Title</h1>
      <DynamicComponent />
    </div>
  );
}
```

## Streaming with Suspense

### Progressive Rendering Pattern

Use React Suspense to stream content and improve perceived performance:

```typescript
// app/dashboard/page.tsx
import { Suspense } from 'react';

// Fast component - renders immediately
function DashboardHeader() {
  return <h1>Dashboard</h1>;
}

// Slow component - fetches data
async function UserStats() {
  const stats = await fetchUserStats(); // Async data fetch
  return (
    <div>
      <p>Total Users: {stats.totalUsers}</p>
      <p>Active Today: {stats.activeToday}</p>
    </div>
  );
}

// Streaming page with Suspense boundaries
export default function Dashboard() {
  return (
    <div>
      <DashboardHeader /> {/* Renders immediately */}
      <Suspense fallback={<p>Loading stats...</p>}>
        <UserStats /> {/* Streams in when ready */}
      </Suspense>
    </div>
  );
}
```

### Multiple Suspense Boundaries

Stream different sections independently for optimal UX:

```typescript
// app/profile/page.tsx
import { Suspense } from 'react';

async function UserProfile({ userId }: { userId: string }) {
  const user = await fetchUser(userId);
  return <div>{user.name}</div>;
}

async function UserPosts({ userId }: { userId: string }) {
  const posts = await fetchUserPosts(userId);
  return <div>{posts.length} posts</div>;
}

async function UserActivity({ userId }: { userId: string }) {
  const activity = await fetchUserActivity(userId);
  return <div>Last active: {activity.lastActive}</div>;
}

export default function ProfilePage({ params }: { params: { id: string } }) {
  return (
    <div>
      {/* Each section streams independently */}
      <Suspense fallback={<div>Loading profile...</div>}>
        <UserProfile userId={params.id} />
      </Suspense>

      <Suspense fallback={<div>Loading posts...</div>}>
        <UserPosts userId={params.id} />
      </Suspense>

      <Suspense fallback={<div>Loading activity...</div>}>
        <UserActivity userId={params.id} />
      </Suspense>
    </div>
  );
}
```

## Server Component Data Fetching

### Optimal Data Fetching Strategy

Use Server Components for data fetching to reduce client bundle size:

```typescript
// ✅ Correct - Server Component fetches data
// app/posts/page.tsx
async function PostsList() {
  // Fetches on server, no client bundle impact
  const posts = await fetch('https://api.example.com/posts').then(r => r.json());

  return (
    <div>
      {posts.map((post: Post) => (
        <article key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.excerpt}</p>
        </article>
      ))}
    </div>
  );
}

export default function PostsPage() {
  return (
    <Suspense fallback={<div>Loading posts...</div>}>
      <PostsList />
    </Suspense>
  );
}
```

```typescript
// ❌ Incorrect - Client Component fetches data
'use client';
import { useEffect, useState } from 'react';

export default function PostsPage() {
  const [posts, setPosts] = useState([]);

  useEffect(() => {
    // Adds fetch logic to client bundle, slower initial load
    fetch('https://api.example.com/posts')
      .then(r => r.json())
      .then(setPosts);
  }, []);

  return <div>...</div>;
}
```

### Parallel Data Fetching

Fetch multiple data sources in parallel for better performance:

```typescript
// ✅ Correct - Parallel fetching
async function DashboardData() {
  // Fetch all data in parallel
  const [user, stats, notifications] = await Promise.all([
    fetchUser(),
    fetchStats(),
    fetchNotifications(),
  ]);

  return (
    <div>
      <UserCard user={user} />
      <StatsPanel stats={stats} />
      <NotificationsList notifications={notifications} />
    </div>
  );
}

// ❌ Incorrect - Sequential fetching (slow)
async function DashboardDataSlow() {
  const user = await fetchUser();           // Wait
  const stats = await fetchStats();         // Then wait
  const notifications = await fetchNotifications(); // Then wait again
  // Total time = sum of all waits
  return <div>...</div>;
}
```

## Cache and Revalidation Patterns

### Static Data Caching

Cache data that doesn't change often:

```typescript
// app/posts/[id]/page.tsx

// Cache forever - perfect for static content
async function getPost(id: string) {
  const res = await fetch(`https://api.example.com/posts/${id}`, {
    cache: 'force-cache', // Default behavior
  });
  return res.json();
}

export default async function PostPage({ params }: { params: { id: string } }) {
  const post = await getPost(params.id);
  return <article>{post.content}</article>;
}
```

### Time-Based Revalidation

Revalidate data at specific intervals:

```typescript
// app/products/page.tsx

// Revalidate every 60 seconds
async function getProducts() {
  const res = await fetch('https://api.example.com/products', {
    next: { revalidate: 60 }, // Refresh every 60 seconds
  });
  return res.json();
}

export default async function ProductsPage() {
  const products = await getProducts();
  return (
    <div>
      {products.map((product: Product) => (
        <div key={product.id}>{product.name}</div>
      ))}
    </div>
  );
}
```

### On-Demand Revalidation

Invalidate cache on-demand (e.g., after updates):

```typescript
// app/api/revalidate/route.ts
import { revalidatePath, revalidateTag } from 'next/cache';
import { NextRequest } from 'next/server';

export async function POST(request: NextRequest) {
  const path = request.nextUrl.searchParams.get('path');

  if (path) {
    revalidatePath(path); // Revalidate specific path
    return Response.json({ revalidated: true, now: Date.now() });
  }

  return Response.json({ revalidated: false });
}
```

### Tag-Based Cache Invalidation

Use tags for fine-grained cache control:

```typescript
// Fetch with tags
async function getProduct(id: string) {
  const res = await fetch(`https://api.example.com/products/${id}`, {
    next: {
      revalidate: 3600, // 1 hour
      tags: [`product-${id}`, 'products'] // Cache tags
    },
  });
  return res.json();
}

// Invalidate by tag
import { revalidateTag } from 'next/cache';

export async function updateProduct(id: string, data: ProductData) {
  // Update product
  await fetch(`https://api.example.com/products/${id}`, {
    method: 'PUT',
    body: JSON.stringify(data),
  });

  // Invalidate specific product and all products
  revalidateTag(`product-${id}`);
  revalidateTag('products');
}
```

### No-Cache Pattern

Disable caching for dynamic data:

```typescript
// app/dashboard/page.tsx

// Always fetch fresh data
async function getUserActivity() {
  const res = await fetch('https://api.example.com/activity', {
    cache: 'no-store', // Never cache
  });
  return res.json();
}

export default async function Dashboard() {
  const activity = await getUserActivity();
  return <div>Last login: {activity.lastLogin}</div>;
}
```

## Verification Steps

- [ ] Images use next/image: `Get-Content **/*.tsx | Select-String "next/image"`
- [ ] Links use next/link: `Get-Content **/*.tsx | Select-String "next/link"`
- [ ] Bundle analyzer configured: `Test-Path "next.config.ts"`
- [ ] Dynamic imports used for heavy components
- [ ] Suspense boundaries implemented for async components
- [ ] Server Components used for data fetching (not client-side useEffect)
- [ ] Parallel data fetching with Promise.all where applicable
- [ ] Cache strategies configured (revalidate, tags, or no-store)

## Success Criteria

- [x] All images optimized with next/image
- [x] Navigation uses next/link with prefetching
- [x] Fonts optimized with next/font
- [x] Bundle analysis tools configured
- [x] Lazy loading implemented for non-critical components
- [x] Streaming with Suspense boundaries for progressive rendering
- [x] Server Components handle data fetching (not client components)
- [x] Parallel data fetching used to minimize wait times
- [x] Cache and revalidation strategies configured appropriately

---

**Note**: This rule ensures NextJS applications achieve optimal performance through built-in optimization features.

