# MySQL Slow Query Log Analysis with Docker and ELK Stack


## Setup
1. `docker-compose up -d`
2.  Create table for testing
    ```
    CREATE TABLE employee (
        id INT AUTO_INCREMENT PRIMARY KEY,
        name VARCHAR(255) NOT NULL,
        age INT,
        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    );

    INSERT INTO employee (name, age) VALUES 
    ('Alice', 25),
    ('Bob', 30),
    ('Charlie', 35),
    ('David', 28),
    ('Eve', 22);
    ```

3. Measure performance with `long_query_time = 0`
    ```
    SET GLOBAL long_query_time = 0;
    SELECT * FROM employee;            -- Fast query
    SELECT SLEEP(1);                   -- 1 second
    SELECT SLEEP(2);                   -- 2 seconds
    SELECT SLEEP(5);                   -- 5 seconds
    ```

4. Measure performance with `long_query_time = 1`
    ```
    SET GLOBAL long_query_time = 1;
    SELECT * FROM employee;            -- Fast query
    SELECT SLEEP(1);                   -- 1 second (should NOT be logged)
    SELECT SLEEP(2);                   -- 2 seconds (should be logged)
    SELECT SLEEP(5);                   -- 5 seconds (should be logged)
    ```

4. Measure performance with `long_query_time = 10`
    ```
    SET GLOBAL long_query_time = 10;
    SELECT * FROM employee;            -- Fast query
    SELECT SLEEP(1);                   -- 1 second (should NOT be logged)
    SELECT SLEEP(2);                   -- 2 seconds (should NOT be logged)
    SELECT SLEEP(12);                  -- 12 seconds (should be logged)
    ```
5. Get MySQL query logs

```
docker exec -it mysql bash
cat /var/log/mysql/slow.log
```


## Comparison of results

### Example of MySql logs output:

```
bash-5.1# cat /var/log/mysql/slow.log
/usr/sbin/mysqld, Version: 8.0.39 (MySQL Community Server - GPL). started with:
Tcp port: 3306  Unix socket: /var/run/mysqld/mysqld.sock
Time                 Id Command    Argument
...
SET GLOBAL long_query_time = 0;
# Time: 2024-10-06T00:00:34.464777Z
# User@Host: root[root] @ localhost []  Id:     8
# Query_time: 0.001425  Lock_time: 0.000051 Rows_sent: 5  Rows_examined: 5
SET timestamp=1728172834;
SELECT * FROM employee;
# Time: 2024-10-06T00:00:35.470726Z
# User@Host: root[root] @ localhost []  Id:     8
# Query_time: 1.001454  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 1
SET timestamp=1728172834;
SELECT SLEEP(1);
# Time: 2024-10-06T00:00:37.472066Z
# User@Host: root[root] @ localhost []  Id:     8
# Query_time: 2.001034  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 1
SET timestamp=1728172835;
SELECT SLEEP(2);
# Time: 2024-10-06T00:00:47.308763Z
# User@Host: root[root] @ localhost []  Id:     8
# Query_time: 5.003156  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 1
SET timestamp=1728172842;
SELECT SLEEP(5);
# Time: 2024-10-06T00:00:50.758571Z
# User@Host: root[root] @ localhost []  Id:     8
# Query_time: 0.002409  Lock_time: 0.000000 Rows_sent: 0  Rows_examined: 0
SET timestamp=1728172850;
SET GLOBAL long_query_time = 1;
# Time: 2024-10-06T00:00:50.763172Z
# User@Host: root[root] @ localhost []  Id:     8
# Query_time: 0.002740  Lock_time: 0.000100 Rows_sent: 5  Rows_examined: 5
SET timestamp=1728172850;
SELECT * FROM employee;
# Time: 2024-10-06T00:00:51.780989Z
# User@Host: root[root] @ localhost []  Id:     8
# Query_time: 1.002734  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 1
SET timestamp=1728172850;
SELECT SLEEP(1);
# Time: 2024-10-06T00:00:53.782849Z
# User@Host: root[root] @ localhost []  Id:     8
# Query_time: 2.000960  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 1
SET timestamp=1728172851;
SELECT SLEEP(2);
# Time: 2024-10-06T00:00:58.785338Z
# User@Host: root[root] @ localhost []  Id:     8
# Query_time: 5.001314  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 1
SET timestamp=1728172853;
SELECT SLEEP(5);
# Time: 2024-10-06T00:01:03.346665Z
# User@Host: root[root] @ localhost []  Id:     8
# Query_time: 0.001098  Lock_time: 0.000000 Rows_sent: 0  Rows_examined: 0
SET timestamp=1728172863;
SET GLOBAL long_query_time = 10;
# Time: 2024-10-06T00:01:03.355657Z
# User@Host: root[root] @ localhost []  Id:     8
# Query_time: 0.001467  Lock_time: 0.000057 Rows_sent: 5  Rows_examined: 5
SET timestamp=1728172863;
SELECT * FROM employee;
# Time: 2024-10-06T00:01:04.362138Z
# User@Host: root[root] @ localhost []  Id:     8
# Query_time: 1.001730  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 1
SET timestamp=1728172863;
SELECT SLEEP(1);
# Time: 2024-10-06T00:01:06.363956Z
# User@Host: root[root] @ localhost []  Id:     8
# Query_time: 2.001167  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 1
SET timestamp=1728172864;
SELECT SLEEP(2);
# Time: 2024-10-06T00:01:18.367697Z
# User@Host: root[root] @ localhost []  Id:     8
# Query_time: 12.003139  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 1
SET timestamp=1728172866;
SELECT SLEEP(12);
# Time: 2024-10-06T00:01:38.342946Z
# User@Host: root[root] @ localhost []  Id:     8
# Query_time: 0.000118  Lock_time: 0.000000 Rows_sent: 0  Rows_examined: 1
SET timestamp=1728172898;
# administrator command: Quit;
```

### Comparison of results in table:

| `long_query_time`                                               | Query                    | Execution Time (seconds) | Logged (Yes/No) | Query Time in Log (seconds) |
|-----------------------------------------------------------------|--------------------------|--------------------------|-----------------|-----------------------------|
| **0** (All queries are logged, including fast ones)             | `SELECT * FROM employee`  | ~0.0026                  | Yes             | 0.002543                    |
|                                                                 | `SELECT SLEEP(1)`         | ~1.0015                  | Yes             | 1.001454                    |
|                                                                 | `SELECT SLEEP(2)`         | ~2.001                   | Yes             | 2.001034                    |
|                                                                 | `SELECT SLEEP(5)`         | ~5.003                   | Yes             | 5.003156                    |
|                                                                 | `SELECT SLEEP(12)`        | ~12.003                  | Yes             | 12.003139                   |
|-----------------------------------------------------------------|--------------------------|--------------------------|-----------------|-----------------------------|
| **1** (Only queries that take more than 1 second are logged)    | `SELECT * FROM employee`  | ~0.0026                  | No              | -                           |
|                                                                 | `SELECT SLEEP(1)`         | ~1.0015                  | Yes             | 1.001454                    |
|                                                                 | `SELECT SLEEP(2)`         | ~2.001                   | Yes             | 2.001034                    |
|                                                                 | `SELECT SLEEP(5)`         | ~5.003                   | Yes             | 5.003156                    |
|                                                                 | `SELECT SLEEP(12)`        | ~12.003                  | Yes             | 12.003139                   |
|-----------------------------------------------------------------|--------------------------|--------------------------|-----------------|-----------------------------|
| **10** (Only queries that take more than 10 seconds are logged) | `SELECT * FROM employee`  | ~0.0026                  | No              | -                           |
|                                                                 | `SELECT SLEEP(1)`         | ~1.0015                  | No              | -                           |
|                                                                 | `SELECT SLEEP(2)`         | ~2.001                   | No              | -                           |
|                                                                 | `SELECT SLEEP(5)`         | ~5.003                   | No              | -                           |
|                                                                 | `SELECT SLEEP(12)`        | ~12.003                  | Yes             | 12.003139                   |
