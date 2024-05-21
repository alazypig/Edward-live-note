##### checksums) as it has already been locked by this process.
更新了 gradle 版本后，出现的问题。需要把 cache 给清除
```bash
find ~/.gradle -type f -name "*.lock" -delete
```