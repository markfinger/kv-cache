Changelog
=========

### 1.1.0 (2015-03-03)

- Fixed an issue where malformed cache files would kill `get` calls. This situation is
  mostly caused by processes stopping mid-write. To fix the problem, we simply remove
  the file and then continue as though it didn't exist.