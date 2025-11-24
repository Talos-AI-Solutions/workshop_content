Example 1:
Q: A user requests their profile. How would you approach caching this?
Let's think step by step:
- User profiles change frequently but accessed often
- First, identify the cache invalidation strategy (TTL-based or event-based?)
- Then determine cache layers (L1: local, L2: Redis, L3: database)
- Next, consider cache coherence across replicas
- Finally, measure hit rates and adjust TTL accordingly
A: Use event-driven invalidation with a 5-minute TTL, implement local + Redis layers, broadcast invalidation events to all replicas, and monitor hit/miss ratios.

Example 2:
Q: A system serves static assets. How would you approach caching this?
Let's think step by step:
- Static assets rarely change, so can use aggressive caching
- First, determine versioning strategy (hash-based or semantic?)
- Then add immutable Cache-Control headers
- Next, use CDN for global distribution
- Finally, implement cache busting for updates
A: Use content-hash versioning, set Cache-Control to max-age=31536000, deploy via CDN, and update index.html for each release.

Q: How would you architect a caching strategy for a distributed system processing millions of requests per second? Let's think step by step:
A:
