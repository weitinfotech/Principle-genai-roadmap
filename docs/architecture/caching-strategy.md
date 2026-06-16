# Recommended caching design for a read-heavy API

**Goal:** Serve hot reads quickly, reduce DB load, and keep stale data bounded.  
This matches the strengths of **distributed caching** and **cache-aside**, which official docs describe as good fits for cloud/server-farm apps and read-heavy scenarios.

---

## Layer 1 — Output caching for safe GET endpoints

Use **output caching** for public or non-user-specific GET endpoints where repeated identical responses are common.  
ASP.NET Core’s output caching middleware caches HTTP responses and is recommended when you want server-controlled caching behavior independent of client cache headers.

---

## Layer 2 — Redis distributed cache with cache-aside

Store API read models or query results in Redis using **cache-aside**:

1. Request checks Redis first  
2. On miss, read DB  
3. Serialize + cache result with TTL  
4. Return response  

Redis’s official guide describes this exact pattern and notes hot-set performance, bounded staleness via TTL, and invalidation on write.

---

## Layer 3 — Optional HybridCache / local hot cache

If you’re on ASP.NET Core and want very fast local reads plus shared durability, **HybridCache** is useful because Microsoft documents it as a unified API over in-memory + distributed cache with **stampede protection**.

---

## Suggested key strategy

Use versioned logical keys such as:

