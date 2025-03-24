# Indexing Strategy

This section outlines our approach to indexing in DuckDB for the ESPN API data structure. Understanding DuckDB's unique indexing capabilities is essential for optimizing query performance while managing memory usage effectively.

## DuckDB Indexing Mechanisms

DuckDB primarily uses two indexing mechanisms:

1. **Zonemaps (min-max indexes)**
   - Created automatically for all general-purpose data types
   - Effectiveness heavily depends on data ordering
   - Critical for filtering operations
   - Requires no explicit creation

2. **ART Indexes**
   - Must be explicitly created
   - Useful for highly selective queries
   - Consume memory that isn't automatically managed
   - Similar to B-tree indexes in traditional RDBMS

## Our Indexing Strategy

Our approach balances query performance with memory efficiency:

1. **Prioritize Data Ordering**
   - Order tables by commonly filtered columns to maximize zonemap effectiveness
   - Example: `Events` table ordered by `start_date` for time-range filtering
   - Use `CREATE TABLE ... AS SELECT ... ORDER BY column1, column2` when creating tables

2. **Explicit Indexes Only When Necessary**
   - Create explicit indexes only for high-value, selective queries
   - Avoid indexes on tables with low cardinality
   - Focus on foreign keys used in JOIN operations
   - Use WHERE clauses in index creation for partial indexes

3. **Memory Management**
   - Monitor memory usage of indexes
   - Implement maintenance routines for periodic database detach/reattach
   - Consider disabling indexes temporarily during bulk operations

4. **Prioritize Materialized Views**
   - Use materialized views for frequent query patterns
   - Pre-compute common aggregations and joins
   - Trade storage for query performance

## Index Creation Patterns

We follow these patterns for index creation:

1. **Unique Identifier Indexes**
   ```sql
   CREATE INDEX idx_tablename_id ON TableName(id_column);
   ```

2. **Foreign Key Indexes**
   ```sql
   CREATE INDEX idx_tablename_fk ON TableName(foreign_key_column);
   ```

3. **Composite Indexes for Common Query Patterns**
   ```sql
   CREATE INDEX idx_tablename_composite ON TableName(column1, column2);
   ```

4. **Partial Indexes for Selective Filtering**
   ```sql
   CREATE INDEX idx_tablename_partial ON TableName(column) WHERE condition;
   ```

## Table-Specific Indexing Approaches

Different entity types require different indexing strategies:

1. **Core Dimensions (small, frequently joined tables)**
   - Order by primary keys
   - Include explicit indexes for API identifiers (e.g., `espn_id`)
   - Example: `CREATE INDEX idx_athletes_espn_id ON Athletes(espn_id);`

2. **Fact Tables (large tables with many queries)**
   - Order by foreign keys and time fields
   - Create composite indexes for common query patterns
   - Use partial indexes for specific filters
   - Example: `CREATE INDEX idx_plays_scoring ON Plays(event_id) WHERE scoring_play = true;`

3. **Junction/Relationship Tables**
   - Order by the most frequently filtered foreign key
   - Create explicit indexes for both foreign keys
   - Example: `CREATE INDEX idx_athlete_teams_team ON AthleteTeams(team_id, season_id);`

## Memory Management Routines

To manage memory consumption by indexes, implement these routines:

```sql
-- In a maintenance script
DETACH DATABASE espn_stats;
ATTACH DATABASE 'path/to/espn_stats.db' AS espn_stats;
```

Or temporarily disable indexes for bulk operations:

```sql
-- Disable index usage temporarily
PRAGMA disable_index_scan;
-- Run memory-intensive operations
-- Re-enable index usage
PRAGMA enable_index_scan;
```

## Performance Monitoring

Regularly evaluate index performance:

1. Use `EXPLAIN ANALYZE` to check query plans
2. Monitor memory usage with `PRAGMA database_size;`
3. Review query performance with and without specific indexes
4. Adjust indexing strategy based on actual query patterns

## Zone Maps vs. Explicit Indexes

When to rely on zonemaps vs. creating explicit indexes:

| Scenario | Zonemap (Ordering) | Explicit Index |
|----------|--------------------|--------------------|
| Range queries on ordered data | ✅ Very effective | ❌ Unnecessary |
| Point lookups by primary key | ✅ Effective for integer PKs | ❌ Unnecessary if PK |
| Equality filters on high-cardinality columns | ❌ Less effective | ✅ Very effective |
| Low-cardinality columns (few distinct values) | ✅ Effective with ordering | ❌ Unnecessary overhead |
| Join conditions | ✅ Effective if ordered by join key | ✅ Helpful for complex joins |
| TEXT columns with wildcards/LIKE | ❌ Not effective | ✅ Beneficial |

## Summary

Our indexing strategy emphasizes:

1. Intentional data ordering for zonemap effectiveness
2. Selective creation of explicit indexes
3. Memory management for index overhead
4. Use of materialized views for common analytical patterns
5. Regular performance evaluation and adjustment 