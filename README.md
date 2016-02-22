# kv-cache

A persistent key/value cache.


## Install

`npm install --save kv-cache`


## Example

```js
import {createFileCache} from 'kv-cache';

// Specify a path to a directory where the values will be stored
const cacheDir = '/path/to/some/directory';

const cache = createFileCache(cacheDir);

cache.events.on('error' => console.error('Cache error: ' + err));

cache.get('foo').then(data => {
  console.log(data); // null

  cache.set('foo', 'bar');

  cache.get('foo').then(data => {
    console.log(data); // 'bar'

    cache.invalidate('foo');

    cache.get('foo').then(data => {
      console.log(data); // null
    });
  });
});
```

`key` should be a string or an array of strings. Keys are hashed with murmur to produce
consistent and performant mappings.

`value` should be a JSON-serializable value.

`get` calls will resolve when the file has been read either from disk or memory.

`set` calls will update the memory store immediately, but will only resolve once the
value has been persisted to disk.

`invalidate` calls will update the memory store immediately, but will only resolve once
the associated value has been removed from disk.

`events` is an EventEmitter instance that emits `'error'` events. If you dont want to
wait for `set` or `invalidate` calls to be resolved or rejected, you can use it to
handle error conditions.


## File cache

```js
import {createFileCache} from 'kv-cache';

const cache = createFileCache('/path/to/directory');
```

Persists data to a directory where each key is mapped to distinct file. Spreading keys
across files avoids IO overhead associated with stale data. To reduce filesystem reads
on repeated gets, it maintains an in-memory map from a key to the serialized object.

When `get` is called, it looks for JSON data in either memory or the filesystem, then
parses the stored JSON and provides the object. If no associated data exists, `null`
is provided.

When `set` is called, the value is immediately serialized to JSON, preserved in memory,
then asynchronously written to disk.

When `invalidate` is called, it removes any related data in memory and then asynchronously
remove the related file. Note: invalidating a missing key will not produce an error.


## Mock cache

```js
import {createMockCache} from 'kv-cache';

const cache = createMockCache();
```

Presents a similar API to the file cache, however it will immediately resolve all promises
with `null`.

This is useful as a drop-in replacement, if you want to debug or profile without the
serialization or IO overheads.
