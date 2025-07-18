To add and backfill the `user_id` column in the `master_product_variants` table using data from `master_products`, follow these steps:

### 1. Add the `user_id` Column
```sql
ALTER TABLE master_product_variants
ADD COLUMN user_id INTEGER; -- Use the correct data type (e.g., BIGINT, UUID) if different
```

### 2. Backfill Data via Join Update
```sql
UPDATE master_product_variants AS v
SET user_id = p.user_id
FROM master_products AS p
WHERE v.master_product_id = p.id;
```

### Key Notes:
1. **Data Type**: Ensure `user_id` in `master_product_variants` matches the data type in `master_products` (e.g., `BIGINT`, `UUID`).
2. **Indexes**: If the table is large, temporarily create an index to speed up the join:
   ```sql
   CREATE INDEX CONCURRENTLY idx_temp_master_product_id ON master_product_variants (master_product_id);
   -- Run the UPDATE statement here
   DROP INDEX CONCURRENTLY idx_temp_master_product_id;
   ```
3. **Locking**: For large tables, update in batches to avoid long locks (example below).

### Batch Update for Large Tables
```sql
DO $$
DECLARE
  batch_size INTEGER := 1000; -- Adjust based on table size
  min_id BIGINT; -- Use your table's actual ID type
  max_id BIGINT;
BEGIN
  SELECT MIN(id), MAX(id) INTO min_id, max_id FROM master_product_variants;
  
  FOR batch_start IN min_id..max_id BY batch_size LOOP
    UPDATE master_product_variants AS v
    SET user_id = p.user_id
    FROM master_products AS p
    WHERE v.id BETWEEN batch_start AND batch_start + batch_size - 1
      AND v.master_product_id = p.id;
    COMMIT; -- Release locks between batches
  END LOOP;
END $$;
```

### Verify Results
```sql
SELECT * FROM master_product_variants
WHERE user_id IS NULL; -- Should return 0 rows if all matched
```

### Optional: Add Constraints
After backfilling, add constraints if needed:
```sql
ALTER TABLE master_product_variants
ALTER COLUMN user_id SET NOT NULL;

-- If you need a foreign key (ensure existing data supports this!)
ALTER TABLE master_product_variants
ADD CONSTRAINT fk_user_id
FOREIGN KEY (user_id) REFERENCES users(id); -- Replace "users" with your user table
```

### Explanation:
- The `UPDATE` joins `master_product_variants` with `master_products` via `master_product_id` to populate `user_id`.
- **Batching** reduces lock contention by committing updates in small chunks.
- Always test in a non-production environment first!