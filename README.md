# Usefull stuff for mysql

## Find how the buffer pool is spent

``` sql
select page_type as page_type, sum(data_size)/1024/1024 as MB
from information_schema.innodb_buffer_page
group by page_type
order by MB desc;
```

## Buffer pool usage by index

``` sql
select table_name as table_name, index_name as index_name, count(*) as page_nbr, sum(data_size)/1024/1024 as MB
from information_schema.innodb_buffer_page
group by table_name, index_name
order by MB desc;
```

## Innodb status ;)
show engine innodb status \G;

## Determine if the buffer pool is equal to working set size (if you can't have buffer pool = rows data size + indexes size (excl. clustered) + 20% )

``` sql
SHOW STATUS like 'Innodb_buffer_pool_wait_free';
```

**Innodb_buffer_pool_wait_free**: Normally, writes to the InnoDB buffer pool happen in the background. When InnoDB needs to read or create a page and no clean pages are available, InnoDB flushes some dirty pages first and waits for that operation to finish. This counter counts instances of these waits. If innodb_buffer_pool_size has been set properly, this value should be small.

``` sql
SHOW STATUS like 'Innodb_buffer_pool_read_requests';
```

**Innodb_buffer_pool_read_requests**:  The number of logical read requests (read from memory).

``` sql
SHOW STATUS like 'innodb_buffer_pool_reads';
```

**Innodb_buffer_pool_reads**: The number of logical reads that InnoDB could not satisfy from the buffer pool, and had to read directly from disk.

If the buffer pool is big enough (equal to the working set size), the Innodb_buffer_pool_reads will decrease (and tend to 0). From time to time, mysql will have to hit the disk to retrieve some missing data, because the working set size != rows data size + indexes size  (you're database do not fit in memory).
Your wss is often < database size, because eg: if you store transactions in a bank, your customers check 99% of the time the past month old transactions and not all transactions from the begining of your bank ;).

So you will say that "show engine innodb status" will give you "Buffer pool hit rate 1000 / 1000", which give you the number of time mysql hit the pool to satisfy queries. Which it's the same has customizing the buffer pool size while watching the innodb_buffer_pool_reads (aiming to 0). And you will be right ;).

>Innodb_buffer_pool_read_requests / (Innodb_buffer_pool_read_requests + Innodb_buffer_pool_reads) * 100 = InnoDB Buffer Pool hit ratio

## Usage of your indexes
	The count_star column show how many times each index was used since MySQL was started.	
```sql
	select INDEX_NAME, COUNT_STAR from performance_schema.table_io_waits_summary_by_index_usage where object_schema = 'DB_NAME' and object_name = 'TABLE_NAME';
```
