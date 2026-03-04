# TiDB DDL Reorg Phase Test Prompt

## Prompt for Claude Code

```
I need you to help me test TiDB's Online DDL behavior, specifically verifying DML operations during different phases of an INT → INT UNSIGNED type change.

## Test Objectives

Verify the following scenarios during WriteOnly/WriteReorganization DDL phases:
1. INSERT positive value - Expected: Success
2. INSERT negative value - Expected: Fail (cast to UNSIGNED fails)
3. UPDATE to negative value - Expected: Fail
4. INSERT large positive value exceeding INT range (e.g., 3000000000) - Expected: Fail (user still operates on INT column)

## Environment Setup

### 1. Start TiDB Cluster (Background)

Use tiup playground to start a single-node cluster. It MUST run in background:

```bash
# Start tiup playground in background
nohup tiup playground --tag ddl-test --db 1 --pd 1 --kv 1 > /tmp/tiup.log 2>&1 &

# Wait for cluster to start (about 30 seconds)
sleep 30

# Verify cluster status
mysql -h 127.0.0.1 -P 4000 -u root -e "SELECT VERSION();"
```

### 2. Adjust DDL Parameters to Slow It Down

```sql
-- Connect to TiDB
mysql -h 127.0.0.1 -P 4000 -u root

-- Reduce backfill speed to extend reorg phase duration
SET GLOBAL tidb_ddl_reorg_worker_cnt = 1;
SET GLOBAL tidb_ddl_reorg_batch_size = 32;
```

### 3. Create Test Table and Insert Large Amount of Data

```sql
CREATE DATABASE IF NOT EXISTS ddl_test;
USE ddl_test;

-- Create test table
DROP TABLE IF EXISTS t;
CREATE TABLE t (
  id INT PRIMARY KEY AUTO_INCREMENT,
  c1 INT NOT NULL DEFAULT 0,
  padding VARCHAR(255) DEFAULT 'padding_data_to_make_row_larger'
);

-- Insert 1 million rows to ensure reorg is slow enough
-- Method 1: Using stored procedure
DELIMITER //
CREATE PROCEDURE insert_data()
BEGIN
  DECLARE i INT DEFAULT 0;
  WHILE i < 1000000 DO
    INSERT INTO t (c1) VALUES (FLOOR(RAND() * 1000));
    SET i = i + 1;
    IF i % 10000 = 0 THEN
      SELECT CONCAT('Inserted ', i, ' rows') AS progress;
    END IF;
  END WHILE;
END //
DELIMITER ;

CALL insert_data();

-- Method 2: Faster approach using cross join
INSERT INTO t (c1) SELECT FLOOR(RAND() * 1000) FROM 
  (SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5) t1,
  (SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5) t2,
  (SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5) t3,
  (SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5) t4,
  (SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5) t5,
  (SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5) t6,
  (SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5) t7;
-- This inserts 5^7 = 78125 rows, run multiple times to reach target data volume
```

## Test Steps

### 1. Start DDL in Terminal 1 (will run for a while)

```sql
-- Terminal 1: Start DDL
USE ddl_test;
ALTER TABLE t MODIFY COLUMN c1 INT UNSIGNED;
```

### 2. Monitor DDL Status in Terminal 2

```sql
-- Terminal 2: Monitor DDL progress
USE ddl_test;

-- Execute every few seconds, observe SCHEMA_STATE column
ADMIN SHOW DDL JOBS WHERE TABLE_NAME = 't' LIMIT 5;

-- Start testing DML when SCHEMA_STATE shows "write reorganization"
```

### 3. Test DML During WriteReorganization State

**IMPORTANT**: Before each DML test, verify the DDL is still in `write reorganization` state.

```sql
-- Terminal 3: Test DML while DDL is running
USE ddl_test;

-- Helper: Check current DDL state (run before EACH test)
-- Must show SCHEMA_STATE = 'write reorganization'
SELECT JOB_ID, SCHEMA_STATE, ROW_COUNT FROM information_schema.ddl_jobs 
WHERE TABLE_NAME = 't' AND STATE = 'running' LIMIT 1;

-- ============================================
-- Test 1: INSERT positive value - Expected: Success
-- ============================================
-- Step 1: Verify state is 'write reorganization'
SELECT JOB_ID, SCHEMA_STATE FROM information_schema.ddl_jobs 
WHERE TABLE_NAME = 't' AND STATE = 'running' LIMIT 1;
-- Step 2: Execute DML
INSERT INTO t (c1) VALUES (100);
SELECT 'Test 1: INSERT positive - should succeed' AS test, @@last_insert_id AS inserted_id;

-- ============================================
-- Test 2: INSERT negative value - Expected: Fail
-- ============================================
-- Step 1: Verify state is 'write reorganization'
SELECT JOB_ID, SCHEMA_STATE FROM information_schema.ddl_jobs 
WHERE TABLE_NAME = 't' AND STATE = 'running' LIMIT 1;
-- Step 2: Execute DML
INSERT INTO t (c1) VALUES (-1);
-- Expected error: [types:1690] constant -1 overflows int unsigned

-- ============================================
-- Test 3: UPDATE to negative value - Expected: Fail
-- ============================================
-- Step 1: Verify state is 'write reorganization'
SELECT JOB_ID, SCHEMA_STATE FROM information_schema.ddl_jobs 
WHERE TABLE_NAME = 't' AND STATE = 'running' LIMIT 1;
-- Step 2: Execute DML
UPDATE t SET c1 = -100 WHERE id = 1;
-- Expected error: [types:1690] constant -100 overflows int unsigned

-- ============================================
-- Test 4: INSERT large positive (exceeds INT range) - Expected: Fail
-- ============================================
-- Step 1: Verify state is 'write reorganization'
SELECT JOB_ID, SCHEMA_STATE FROM information_schema.ddl_jobs 
WHERE TABLE_NAME = 't' AND STATE = 'running' LIMIT 1;
-- Step 2: Execute DML
INSERT INTO t (c1) VALUES (3000000000);
-- Expected error: Out of range value for column 'c1'
-- Because user still operates on INT column, not UNSIGNED

-- ============================================
-- Test 5: UPDATE to large positive - Expected: Fail
-- ============================================
-- Step 1: Verify state is 'write reorganization'
SELECT JOB_ID, SCHEMA_STATE FROM information_schema.ddl_jobs 
WHERE TABLE_NAME = 't' AND STATE = 'running' LIMIT 1;
-- Step 2: Execute DML
UPDATE t SET c1 = 4294967295 WHERE id = 1;
-- Expected error: Out of range value for column 'c1'
```

## Verify Results

After DDL completes, run these verification queries:

```sql
-- Verify column type has changed
DESC t;
-- c1 should show as 'int unsigned'

-- Verify large positive values now work
INSERT INTO t (c1) VALUES (3000000000);
SELECT * FROM t WHERE c1 = 3000000000;

-- Verify negative values are no longer allowed
INSERT INTO t (c1) VALUES (-1);
-- Expected error: Out of range value for column 'c1'
```

## Cleanup

```bash
# Stop tiup playground
tiup clean ddl-test
```

## Expected Results Summary

| Test | Before DDL | WriteOnly/Reorg Phase | After DDL |
|------|------------|----------------------|-----------|
| INSERT (100) | ✅ Success | ✅ Success | ✅ Success |
| INSERT (-1) | ✅ Success | ❌ Cast fails | ❌ Out of range |
| INSERT (3000000000) | ❌ Out of range | ❌ Out of range | ✅ Success |
| UPDATE c1=-100 | ✅ Success | ❌ Cast fails | ❌ Out of range |

Please execute the test steps above and record whether each test's actual result matches the expected result.
```
