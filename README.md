# ts-promise-cache #

[![Build Status](https://travis-ci.org/normartin/ts-promise-cache.svg?branch=master)](https://travis-ci.org/normartin/ts-promise-cache)
[![Coverage Status](https://coveralls.io/repos/github/normartin/ts-promise-cache/badge.svg?branch=master)](https://coveralls.io/github/normartin/ts-promise-cache?branch=master)
[![Dependencies](https://david-dm.org/normartin/ts-promise-cache.svg)](https://david-dm.org/normartin/ts-promise-cache)
[![NPM](https://img.shields.io/npm/v/ts-promise-cache.svg?color=#555)](https://www.npmjs.com/package/ts-promise-cache)

Loading cache for promises. 
Does not suffer from thundering herds problem (aka [cache stampede](https://en.wikipedia.org/wiki/Cache_stampede)).

## Usage ##
The constructor takes a loader that loads missing entries.
By default rejected Promises are not kept in the cache. 

_loader: (key: string) => Promise<T>_

```typescript
const cache = new PromiseCache<string>((key: string) => Promise.resolve("value"));

const value = await cache.get("key");

expect(value).to.eq("value");
```

## Config ##
The second constructor argument is an optional config (Partial&lt;CacheConfig> config is ok).
```typescript
export class CacheConfig<T> {
    // how often to check for expired entries
    public readonly checkInterval: number | "NEVER" = "NEVER";
    // time to live (milliseconds)
    public readonly ttl: number | "FOREVER" = "FOREVER";
    // specifies that entries should be removed 'ttl' milliseconds from either when the cache was accessed (read) or when the cache value was created or replaced
    public readonly ttlAfter: "ACCESS" | "WRITE" = "ACCESS";
    // remove rejected promises?
    public readonly removeRejected: boolean = true;
    // fallback for rejected promises
    public readonly onReject: (error: Error, key: string, loader: (key: string) => Promise<T>) => Promise<T>
        = (error) => Promise.reject(error)
    // called before entries are removed because of ttl
    public readonly onRemove: (key: string, p: Promise<T>) => void
        = () => undefined
}
```
Using [Partial](https://www.typescriptlang.org/docs/handbook/advanced-types.html)&lt;CacheConfig> it looks like:
```typescript
new PromiseCache<string>(loader, {ttl: 1000, onRemove: (key) => console.log("removing", key)});
```

## Single promise cache ##
In case you have a single value to cache, use _singlePromiseCache_ factory.
```typescript
// singlePromiseCache<T>(loader: () => Promise<T>, config?: Partial<CacheConfig<T>>): () => Promise<T>

const cache = singlePromiseCache(() => Promise.resolve("value"));

const value = await cache();
expect(value).to.eq("value");
```

## TTL Expiry ##
The default behavior is for values to remain in the cache for 'ttl' milliseconds since the last access.  It
is sometimes useful for cache expiry to be relative to when the cached value was first created (or last changed).

For this situation, set ttlAfter:"WRITE":
```typescript
new PromiseCache<string>(loader, {ttl: 1000, ttlAfter:"WRITE"});

```


## Statistics ##
_stats()_ returns statistics:
 ```typescript
interface Stats {
    readonly misses: number;
    readonly hits: number;
    readonly entries: number;
    readonly failedLoads: number;
}
 ```

## Set value ##
You can set a value directly
```typescript
cache.set("key", "value");
expect(await cache.get("key")).to.eq("value");
```

## Retry ##
Retry can by implemented by using [ts-retry-promise](https://www.npmjs.com/package/ts-retry-promise)
```typescript
const loader = failsOneTime("value");
const cache = new PromiseCache<string>(() => retry(loader));
expect(await cache.get("key")).to.eq("value");
```
