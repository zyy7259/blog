# Safari

When start a transaction on more than one object stores, Safari throw an error:
- NotFoundError: DOM IDBDatabase Exception 8: An operation failed because the requested database object could not be found.

# limit

- global limit: 50% of free disk space
- group limit: 20% of the global limit (group of origins)

# LRU policy

When exceed global limit, the least recently used origin will be deleted first, then the next one, until no longer over limit.

# See also

- Source: [limit](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API/Browser_storage_limits_and_eviction_criteria)
- Concepts: [Basic Concepts Behind IndexedDB](basic.md)
- Tutorial: [Using IndexedDB](tutorial.md)
- Reference: [IndexedDB API reference](api.md)
- Specification: [IndexedDB Database API Specification](http://www.w3.org/TR/IndexedDB/)
