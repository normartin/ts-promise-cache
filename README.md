# ts-promise-cache #

[![Build Status](https://travis-ci.org/normartin/ts-promise-cache.svg?branch=master)](https://travis-ci.org/normartin/ts-promise-cache)
[![Coverage Status](https://coveralls.io/repos/github/normartin/ts-promise-cache/badge.svg?branch=master)](https://coveralls.io/github/normartin/ts-promise-cache?branch=master)
[![Dependencies](https://david-dm.org/normartin/ts-promise-cache.svg)](https://david-dm.org/normartin/ts-promise-cache)

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
The second constructor argument is an optional config (Partial config is ok).
```typescript
interface CacheConfig<T> {
    // how often to check for expired entries (default: "NEVER")
    checkInterval: number | "NEVER";
    // time to live after last access (default: "FOREVER")
    ttl: number | "FOREVER";
    // fallback for rejected promises (default: (error) => Promise.reject(error))
    onReject: (error: Error, key: string, loader: (key: string) => Promise<T>) => Promise<T>;
    // called before entries are removed because of ttl
    onRemove: (key: string, p: Promise<T>) => void;
    // remove rejected promises? (default: true)
    removeRejected: boolean;
}
```

## Single promise cache ##
In case you have a single value to cache, use _singlePromiseCache_ factory.
```typescript
// singlePromiseCache<T>(loader: () => Promise<T>, config?: Partial<CacheConfig<T>>): () => Promise<T>

const cache = singlePromiseCache(() => Promise.resolve("value"));

const value = await cache();
expect(value).to.eq("value");
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

## Retry ##
Retry can by implemented by using [ts-retry-promise](https://www.npmjs.com/package/ts-retry-promise)
```typescript
const loader = failsOneTime("value");
const cache = new PromiseCache<string>(() => retry(loader));
expect(await cache.get("key")).to.eq("value");
```

## Previous versions ##
Source: https://bitbucket.org/martinmo/ts-tools
