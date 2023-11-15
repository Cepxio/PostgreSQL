# Analyze and tweak Postgres Background writer and checkpointer


## The next output from the view help us to monitor the current databse load and needed tuning


```sql
postgres=# SELECT * FROM pg_stat_bgwriter\gx
-[ RECORD 1 ]---------+------------------------------
checkpoints_timed     | 38447
checkpoints_req       | 77
checkpoint_write_time | 278878329
checkpoint_sync_time  | 266106
buffers_checkpoint    | 3799216
buffers_clean         | 1603
maxwritten_clean      | 14
buffers_backend       | 1153320
buffers_backend_fsync | 0
buffers_alloc         | 1832554
stats_reset           | 2020-02-22 04:50:43.931931+01
```

Now back to pg_stat_bgwriter, using its stats we can understand some moments related to bgwriter.


- `maxwritten_clean` shows how many times bgwriter stopped because maxpages was exceeded. When you see high values there, you should increase bgwriter_lru_maxpages.
- `buffers_clean` and `buffers_backend` show number of buffers cleaned by bgwriter and postgres’ backends respectively – buffers_clean should be greater than buffers_backend. Otherwise, you should increase `bgwriter_lru_multiplier` and decrease `bgwriter_delay`. Note, it also may be a sign that you have insufficient *shared buffers* and hot part of your data don’t fit into shared buffers and forced to travel between RAM and disks.
- `buffers_backend_fsync` shows if backends are forced to make its own fsync requests to synchronize buffers with storage. Any values above zero point to problems with storage when fsync queue is completely filled. The newer versions of postgres addressed these issues and I haven’t seen non-zero values now for a long time.

> After changing settings, I recommend resetting pg_stat_bgwriter stats with pg_stat_reset_shared(‘bgwriter’) function and re-check stats the next day.
