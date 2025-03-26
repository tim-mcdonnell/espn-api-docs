# Materialized View Strategy

This document outlines our comprehensive approach to managing materialized views in the ESPN API data structure. Materialized views serve as a critical performance optimization technique that balances query efficiency with resource utilization.

## Purpose and Benefits

Materialized views provide several key advantages in our database architecture:

1. **Query Performance**: Pre-computed results for complex joins and aggregations significantly reduce execution time
2. **Resource Efficiency**: Decreased CPU and memory usage for frequently executed queries
3. **Consistency**: Single source of truth for commonly used aggregated metrics
4. **Simplified Access**: Complex data relationships presented through simplified view structures

## View Classification System

We categorize materialized views based on their refresh requirements and usage patterns:

### By Refresh Frequency

| Category | Refresh Frequency | Examples |
|----------|-------------------|----------|
| **Real-time** | Every 1-5 minutes | Live game statistics, current scores |
| **Near real-time** | Every 15-30 minutes | In-progress tournament standings |
| **Daily** | Once per day | Team standings, athlete season totals |
| **Weekly** | Once per week | Power rankings, trend analysis |
| **Monthly** | Once per month | Long-term statistical aggregations |
| **Seasonal** | End of season | Historical season-level aggregations |

### By Data Volatility

| Category | Characteristics | Examples |
|----------|-----------------|----------|
| **Highly volatile** | Changes with every game event | Play-by-play statistics, live odds |
| **Moderately volatile** | Changes after game completion | Team records, playoff seedings |
| **Low volatility** | Changes infrequently | Historical performance metrics, career stats |
| **Static** | Changes rarely or never | All-time records, completed season data |

## Core Materialized Views

| View Name | Purpose | Refresh Category | Volatility |
|-----------|---------|------------------|------------|
| `vw_mat_CurrentStandings` | Team standings by division/conference | Near real-time | Moderate |
| `vw_mat_AthleteSeasonTotals` | Season-to-date stats by athlete | Daily | Moderate |
| `vw_mat_TeamPerformanceMetrics` | Advanced team metrics | Daily | Moderate |
| `vw_mat_HeadToHeadRecords` | Historical matchup results | Weekly | Low |
| `vw_mat_TrendingAthletes` | Athletes with significant performance changes | Daily | High |
| `vw_mat_InjuryImpact` | Injury effects on team performance | Weekly | Moderate |
| `vw_mat_PowerRankings` | Computed team power rankings | Weekly | Moderate |
| `vw_mat_ShootingEfficiency` | Shot charts with efficiency metrics | Daily | Moderate |
| `vw_mat_GamePredictions` | Predictive modeling outputs | Daily | Moderate |
| `vw_mat_HistoricalComparisons` | Current vs. historical performance | Monthly | Low |

## Refresh Schedule Implementation

Our refresh strategy is designed to optimize resource usage while maintaining data freshness:

### Time-Based Scheduling

```sql
-- Example refresh job definition (pseudo-code)
CREATE REFRESH JOB refresh_standings AS (
  REFRESH MATERIALIZED VIEW vw_mat_CurrentStandings
) SCHEDULE EVERY 30 MINUTES;

CREATE REFRESH JOB refresh_daily_views AS (
  REFRESH MATERIALIZED VIEW vw_mat_AthleteSeasonTotals;
  REFRESH MATERIALIZED VIEW vw_mat_TeamPerformanceMetrics;
  REFRESH MATERIALIZED VIEW vw_mat_TrendingAthletes;
) SCHEDULE DAILY AT '04:00';
```

### Event-Based Triggering

For sports data, certain events should trigger immediate view refreshes:

1. **Game Completion**: Trigger standings and team statistics updates
2. **Injury Status Change**: Trigger injury impact view updates
3. **Significant Statistical Outlier**: Trigger trending athletes view updates

### Conditional Refresh Logic

```sql
-- Pseudo-code for conditional refresh
IF (SELECT COUNT(*) FROM Events 
    WHERE status = 'FINAL' 
    AND updated_at > last_refresh_time) > 0 THEN
  REFRESH MATERIALIZED VIEW vw_mat_CurrentStandings;
END IF;
```

## Automated View Maintenance Procedures

### Refresh Process Monitoring

1. **Refresh Logging**:
```sql
CREATE TABLE ViewRefreshLog (
  view_name VARCHAR,
  refresh_start TIMESTAMP,
  refresh_end TIMESTAMP,
  rows_affected INTEGER,
  status VARCHAR
);

-- Inside refresh procedure
INSERT INTO ViewRefreshLog VALUES (
  'vw_mat_CurrentStandings',
  CURRENT_TIMESTAMP,
  NULL,
  0,
  'STARTED'
);

-- After completion
UPDATE ViewRefreshLog 
SET refresh_end = CURRENT_TIMESTAMP,
    rows_affected = (SELECT COUNT(*) FROM vw_mat_CurrentStandings),
    status = 'COMPLETED'
WHERE view_name = 'vw_mat_CurrentStandings'
AND refresh_end IS NULL;
```

2. **Error Handling**:
```sql
-- Pseudo-code for error handling
BEGIN
  -- Refresh operation
  REFRESH MATERIALIZED VIEW vw_mat_CurrentStandings;
  
  -- Success logging
  INSERT INTO ViewRefreshLog VALUES (..., 'COMPLETED');
EXCEPTION
  -- Error handling
  INSERT INTO ViewRefreshLog VALUES (..., 'FAILED: ' || error_message);
  
  -- Alert mechanism
  CALL send_alert('View refresh failed: ' || error_message);
END;
```

### Health Check System

Implement automatic validation checks after each refresh:

```sql
-- Pseudo-code for view validation
CREATE PROCEDURE validate_standings_view() AS
BEGIN
  -- Check row count
  IF (SELECT COUNT(*) FROM vw_mat_CurrentStandings) < 
     (SELECT COUNT(*) FROM Teams WHERE active = TRUE) THEN
    RAISE EXCEPTION 'Standings view missing teams';
  END IF;
  
  -- Check for data inconsistencies
  IF EXISTS (
    SELECT team_id FROM vw_mat_CurrentStandings
    GROUP BY team_id HAVING COUNT(*) > 1
  ) THEN
    RAISE EXCEPTION 'Duplicate team entries in standings';
  END IF;
  
  -- Additional validation rules...
END;
```

### Dependency Management

Track view dependencies to handle cascading refreshes:

```sql
CREATE TABLE ViewDependencies (
  view_name VARCHAR,
  depends_on VARCHAR,
  PRIMARY KEY (view_name, depends_on)
);

-- Example dependencies
INSERT INTO ViewDependencies VALUES
  ('vw_mat_PowerRankings', 'vw_mat_TeamPerformanceMetrics'),
  ('vw_mat_PowerRankings', 'vw_mat_CurrentStandings');

-- Procedure to refresh views with dependencies
CREATE PROCEDURE refresh_with_dependencies(view_name VARCHAR) AS
BEGIN
  -- First refresh dependencies
  FOR dep IN (SELECT depends_on FROM ViewDependencies WHERE view_name = view_name)
  LOOP
    CALL refresh_with_dependencies(dep);
  END LOOP;
  
  -- Then refresh the target view
  EXECUTE 'REFRESH MATERIALIZED VIEW ' || view_name;
END;
```

## Performance Optimization and Resource Management

### Memory Usage Monitoring

```sql
-- Track memory usage of materialized views
CREATE VIEW vw_view_memory_usage AS
SELECT 
  view_name,
  estimated_size,
  last_refresh,
  query_count,
  avg_query_time
FROM system_views_info;  -- Hypothetical system view

-- Alert on excessive memory usage
CREATE PROCEDURE check_memory_usage() AS
BEGIN
  IF EXISTS (
    SELECT view_name FROM vw_view_memory_usage
    WHERE estimated_size > threshold_value  -- Define appropriate threshold
  ) THEN
    -- Generate alert or consider view optimization
  END IF;
END;
```

### Selective Column Materialization

For large views, consider materializing only the most frequently accessed columns:

```sql
-- Instead of materializing all athlete statistics
CREATE MATERIALIZED VIEW vw_mat_AthleteSeasonTotals AS
SELECT 
  athlete_id,
  team_id,
  season_id,
  games_played,
  points,
  rebounds,
  assists
  -- Only include most commonly queried metrics
FROM AthleteStatistics
WHERE season_id = current_season_id;
```

### Incremental Refresh Strategy

For large views, implement incremental refresh logic:

```sql
-- Pseudo-code for incremental refresh
CREATE PROCEDURE incremental_refresh_athlete_stats() AS
BEGIN
  -- Create temporary table with only new/changed data
  CREATE TEMPORARY TABLE temp_new_stats AS
  SELECT * FROM AthleteStatistics
  WHERE updated_at > (SELECT last_refresh FROM ViewRefreshLog 
                     WHERE view_name = 'vw_mat_AthleteSeasonTotals'
                     ORDER BY refresh_end DESC LIMIT 1);
  
  -- Delete changed records from materialized view
  DELETE FROM vw_mat_AthleteSeasonTotals
  WHERE athlete_id IN (SELECT athlete_id FROM temp_new_stats);
  
  -- Insert updated records
  INSERT INTO vw_mat_AthleteSeasonTotals
  SELECT athlete_id, team_id, season_id, games_played, points, rebounds, assists
  FROM temp_new_stats;
  
  -- Update refresh log
  UPDATE ViewRefreshLog SET 
    refresh_end = CURRENT_TIMESTAMP,
    rows_affected = (SELECT COUNT(*) FROM temp_new_stats),
    status = 'INCREMENTAL REFRESH COMPLETED'
  WHERE view_name = 'vw_mat_AthleteSeasonTotals'
  AND refresh_end IS NULL;
  
  DROP TABLE temp_new_stats;
END;
```

### Query Usage Tracking

Implement tracking to identify which views provide the most value:

```sql
CREATE TABLE ViewUsageStats (
  view_name VARCHAR,
  query_timestamp TIMESTAMP,
  execution_time_ms INTEGER,
  user_id VARCHAR
);

-- Trigger or logging mechanism to record view usage
CREATE PROCEDURE log_view_usage(view_name VARCHAR, exec_time INTEGER) AS
BEGIN
  INSERT INTO ViewUsageStats VALUES (
    view_name,
    CURRENT_TIMESTAMP,
    exec_time,
    current_user()
  );
END;
```

## Archiving and Cleanup Strategy

```sql
-- Pseudo-code for view archiving
CREATE PROCEDURE archive_old_materialized_data() AS
BEGIN
  -- For seasonal views, archive at end of season
  IF current_date > season_end_date THEN
    -- Create permanent table from materialized view
    EXECUTE 'CREATE TABLE archive_' || current_season || '_team_stats AS 
             SELECT * FROM vw_mat_TeamPerformanceMetrics';
    
    -- Refresh view for new season
    EXECUTE 'REFRESH MATERIALIZED VIEW vw_mat_TeamPerformanceMetrics';
  END IF;
END;
```

## Implementation Considerations for DuckDB

Since DuckDB doesn't natively support traditional materialized views with automatic refresh capabilities, we implement our strategy using these approaches:

1. **Emulated Materialized Views**: Create tables that function as materialized views

```sql
-- Create materialized view equivalent
CREATE TABLE vw_mat_CurrentStandings AS
SELECT 
  t.id AS team_id,
  t.name AS team_name,
  d.name AS division,
  COUNT(CASE WHEN e.home_score > e.away_score AND e.home_team_id = t.id THEN 1
            WHEN e.away_score > e.home_score AND e.away_team_id = t.id THEN 1
            ELSE NULL END) AS wins,
  COUNT(CASE WHEN e.home_score < e.away_score AND e.home_team_id = t.id THEN 1
            WHEN e.away_score < e.home_score AND e.away_team_id = t.id THEN 1
            ELSE NULL END) AS losses
FROM Teams t
JOIN Divisions d ON t.division_id = d.id
LEFT JOIN Events e ON (e.home_team_id = t.id OR e.away_team_id = t.id)
              AND e.status = 'FINAL'
GROUP BY t.id, t.name, d.name
ORDER BY d.name, wins DESC, losses ASC;

-- Create refresh procedure
CREATE PROCEDURE refresh_standings() AS
BEGIN
  -- Drop and recreate for full refresh
  DROP TABLE IF EXISTS vw_mat_CurrentStandings;
  
  CREATE TABLE vw_mat_CurrentStandings AS
  SELECT -- same query as above
  FROM -- same tables as above
  WHERE -- same conditions as above
  GROUP BY -- same grouping as above
  ORDER BY -- same ordering as above;
END;
```

2. **Scheduled Refresh Management**: Implement external scheduling system

```python
# Example Python script for scheduled refreshes
import duckdb
import schedule
import time
import logging

logging.basicConfig(level=logging.INFO, 
                    format='%(asctime)s - %(message)s',
                    filename='refresh_log.txt')

def refresh_view(view_name, sql_command):
    try:
        conn = duckdb.connect('espn_stats.db')
        start_time = time.time()
        logging.info(f"Starting refresh of {view_name}")
        
        conn.execute(sql_command)
        
        end_time = time.time()
        logging.info(f"Completed refresh of {view_name} in {end_time - start_time:.2f} seconds")
        conn.close()
    except Exception as e:
        logging.error(f"Error refreshing {view_name}: {str(e)}")

# Define refresh jobs
schedule.every(30).minutes.do(
    refresh_view, 
    "vw_mat_CurrentStandings",
    "CALL refresh_standings();"
)

schedule.every().day.at("04:00").do(
    refresh_view,
    "vw_mat_AthleteSeasonTotals",
    "CALL refresh_athlete_stats();"
)

# Run the scheduler
while True:
    schedule.run_pending()
    time.sleep(60)
```

## Conclusion

Our materialized view strategy balances performance optimization with resource management through:

1. **Strategic Classification**: Views are categorized by refresh needs and data volatility
2. **Customized Refresh Schedules**: Each view is refreshed based on its specific requirements
3. **Automated Maintenance**: Monitoring and validation ensure data quality and performance
4. **Resource Optimization**: Memory usage tracking and incremental updates maximize efficiency
5. **Adaptive Implementation**: Custom approach accommodates DuckDB's capabilities

By following this strategy, we ensure that our analytical queries benefit from pre-computed results while maintaining reasonable resource utilization and data freshness. 