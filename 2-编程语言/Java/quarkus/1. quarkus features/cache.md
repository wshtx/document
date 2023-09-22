# cache

## quazrkus cache

### enable cache

相关注解:

- `@CacheResult(cacheName="")` : when a method annotated with the annotation is invoked, quarkus will enable compute a cache key and use it to check in the cache whether the method has been already invoked.
- `@CacheInvalidate`: remove an entry from the cache
- `@CacheInvalidateAll`: when a method annotated with the annotation is invoked, remove all entries from the cache.
- `@CacheKey`: it is identified as a part of the cache key during an invocation of a method annnotated with `@CacheResult` or `@CacheInvalidate`.

### Cache keys build logic

>[Cache keys build logic](https://quarkus.io/guides/cache#cache-keys-building-logic)

### Generate a cache key with `@CacheKeyGenerator`

>[Generate a cache key with `@CacheKeyGenerator`](https://quarkus.io/guides/cache#generating-a-cache-key-with-cachekeygenerator)