# Performance Metrics to Rankings Relationships

This section documents the relationships between team performance metrics and ranking systems. These relationships enable analysis of how team performance metrics correlate with and potentially predict ranking movements, as well as how different ranking systems value different aspects of team performance.

## RankingPerformanceCorrelation

Tracks correlations between specific performance metrics and ranking changes.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| correlation_id | INTEGER | Primary key (auto-increment) | 1 |
| ranking_system_id | INTEGER | Foreign key to RankingSystems | 1 |
| season_id | INTEGER | Foreign key to seasons | 2024 |
| performance_metric | VARCHAR | Performance metric being correlated | offensive_rating |
| correlation_coefficient | DECIMAL | Pearson correlation coefficient | 0.78 |
| r_squared | DECIMAL | Coefficient of determination | 0.61 |
| confidence_level | DECIMAL | Statistical confidence (0.0-1.0) | 0.95 |
| sample_size | INTEGER | Number of data points in analysis | 357 |
| is_predictive | BOOLEAN | If metric predicts future ranking | true |
| prediction_window | INTEGER | Days ahead metric predicts (if predictive) | 7 |
| is_statistically_significant | BOOLEAN | If correlation is significant | true |
| created_at | TIMESTAMP | Record creation timestamp | 2024-03-12T18:23:01Z |
| updated_at | TIMESTAMP | Record update timestamp | 2024-03-12T18:23:01Z |

**Indexing:**
```sql
CREATE INDEX idx_ranking_correlation_system ON RankingPerformanceCorrelation(ranking_system_id, season_id);
CREATE INDEX idx_ranking_correlation_metric ON RankingPerformanceCorrelation(performance_metric, ranking_system_id);
```

## TeamRankingPerformance

Links specific team rankings with their performance metrics at the time of ranking.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| rank_perf_id | INTEGER | Primary key (auto-increment) | 1 |
| team_ranking_id | INTEGER | Foreign key to TeamRankings | 1254 |
| team_id | INTEGER | Foreign key to Teams | 2509 |
| poll_schedule_id | INTEGER | Foreign key to PollSchedule | 32 |
| offensive_rating | DECIMAL | Points scored per 100 possessions | 112.4 |
| defensive_rating | DECIMAL | Points allowed per 100 possessions | 98.6 |
| net_rating | DECIMAL | Net point differential per 100 possessions | 13.8 |
| effective_fg_pct | DECIMAL | Effective field goal percentage | 54.2 |
| strength_of_schedule | DECIMAL | Strength of schedule rating | 8.45 |
| win_pct | DECIMAL | Win percentage | 0.833 |
| last_5_win_pct | DECIMAL | Win percentage in last 5 games | 0.8 |
| road_win_pct | DECIMAL | Win percentage in road games | 0.769 |
| rank_movement | INTEGER | Change in rank from previous poll | 2 |
| created_at | TIMESTAMP | Record creation timestamp | 2024-03-12T18:23:01Z |
| updated_at | TIMESTAMP | Record update timestamp | 2024-03-12T18:23:01Z |

**Indexing:**
```sql
CREATE INDEX idx_team_ranking_perf_team ON TeamRankingPerformance(team_id, poll_schedule_id);
CREATE INDEX idx_team_ranking_perf_ranking ON TeamRankingPerformance(team_ranking_id);
```

## PollSystemMetricWeights

Estimates the implicit weights different ranking systems apply to performance metrics.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| weight_id | INTEGER | Primary key (auto-increment) | 1 |
| ranking_system_id | INTEGER | Foreign key to RankingSystems | 1 |
| season_id | INTEGER | Foreign key to seasons | 2024 |
| metric_name | VARCHAR | Performance metric name | offensive_rating |
| estimated_weight | DECIMAL | Estimated importance weight (0.0-1.0) | 0.35 |
| confidence_interval | VARCHAR | Statistical confidence interval | [0.28, 0.42] |
| methodology | VARCHAR | How weight was estimated | regression |
| explanation | VARCHAR | Human-readable explanation | Human voters value offensive rating more than efficiency metrics |
| created_at | TIMESTAMP | Record creation timestamp | 2024-03-12T18:23:01Z |
| updated_at | TIMESTAMP | Record update timestamp | 2024-03-12T18:23:01Z |

**Indexing:**
```sql
CREATE INDEX idx_poll_weights_system ON PollSystemMetricWeights(ranking_system_id, season_id);
CREATE INDEX idx_poll_weights_metric ON PollSystemMetricWeights(metric_name, ranking_system_id);
```

## RankingConsistencyMetrics

Analyzes how consistent rankings are with performance metrics.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| consistency_id | INTEGER | Primary key (auto-increment) | 1 |
| ranking_system_id | INTEGER | Foreign key to RankingSystems | 1 |
| season_id | INTEGER | Foreign key to seasons | 2024 |
| week_id | INTEGER | Foreign key to weeks | 15 |
| anomaly_count | INTEGER | Number of ranking anomalies | 3 |
| consistency_score | DECIMAL | Overall consistency score (0-100) | 87.5 |
| biggest_anomaly_team_id | INTEGER | Team with largest ranking anomaly | 2533 |
| biggest_anomaly_magnitude | INTEGER | Size of largest anomaly (ranks) | 8 |
| overrated_teams_count | INTEGER | Number of statistically overrated teams | 5 |
| underrated_teams_count | INTEGER | Number of statistically underrated teams | 4 |
| methodology_description | VARCHAR | Description of measurement methodology | Compared NET rankings with AP Poll |
| created_at | TIMESTAMP | Record creation timestamp | 2024-03-12T18:23:01Z |
| updated_at | TIMESTAMP | Record update timestamp | 2024-03-12T18:23:01Z |

**Indexing:**
```sql
CREATE INDEX idx_ranking_consistency_system ON RankingConsistencyMetrics(ranking_system_id, season_id);
CREATE INDEX idx_ranking_consistency_week ON RankingConsistencyMetrics(season_id, week_id, ranking_system_id);
```

## RankingAnomaly

Documents specific instances where rankings significantly deviate from performance metrics.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| anomaly_id | INTEGER | Primary key (auto-increment) | 1 |
| team_ranking_id | INTEGER | Foreign key to TeamRankings | 1254 |
| team_id | INTEGER | Foreign key to Teams | 2509 |
| poll_schedule_id | INTEGER | Foreign key to PollSchedule | 32 |
| expected_rank | INTEGER | Rank expected based on metrics | 12 |
| actual_rank | INTEGER | Actual rank in poll | 5 |
| deviation | INTEGER | Difference between expected and actual | -7 |
| primary_factor | VARCHAR | Primary factor for anomaly | recent_upset_win |
| recency_bias | BOOLEAN | If recency bias is factor | true |
| narrative_effect | VARCHAR | Narrative driving anomaly | Dramatic comeback in nationally televised game |
| media_coverage_score | DECIMAL | Media coverage intensity (0.0-10.0) | 8.5 |
| created_at | TIMESTAMP | Record creation timestamp | 2024-03-12T18:23:01Z |
| updated_at | TIMESTAMP | Record update timestamp | 2024-03-12T18:23:01Z |

**Indexing:**
```sql
CREATE INDEX idx_ranking_anomaly_team ON RankingAnomaly(team_id, poll_schedule_id);
CREATE INDEX idx_ranking_anomaly_ranking ON RankingAnomaly(team_ranking_id);
CREATE INDEX idx_ranking_anomaly_deviation ON RankingAnomaly(ABS(deviation) DESC);
```

## Implementation Guidelines

When implementing these performance-ranking relationships, follow these guidelines:

1. **Data Collection Synchronization**:
   - Ensure performance metrics are calculated and stored prior to analyzing their relationship with rankings
   - Capture performance metrics as close as possible to the ranking release date
   - Maintain timestamp information to enable time-series analysis

2. **Calculation Methodology**:
   - Document statistical methodologies used for correlation and weight calculations 
   - Use consistent statistical approaches across seasons for longitudinal analysis
   - Consider both raw and opponent-adjusted metrics for comprehensive analysis

3. **Interpretation Context**:
   - Provide contextual information for interpreting statistical relationships
   - Document limitations and assumptions in analysis
   - Consider external factors that may influence both rankings and metrics

4. **Visualization Support**:
   - Structure data to enable clear visualization of ranking-performance relationships
   - Include sufficient metadata for labeling and contextualizing visualizations
   - Support both point-in-time and trend analysis visualizations

## Example Queries

**Find metrics that best predict AP Poll rankings:**

```sql
SELECT performance_metric, correlation_coefficient, r_squared, confidence_level
FROM RankingPerformanceCorrelation
WHERE ranking_system_id = (SELECT ranking_system_id FROM RankingSystems WHERE short_name = 'AP')
AND season_id = ?
AND is_predictive = true
ORDER BY correlation_coefficient DESC
LIMIT 10;
```

**Identify teams whose ranking most diverges from their performance metrics:**

```sql
SELECT t.display_name as team, ra.actual_rank, ra.expected_rank, ra.deviation, 
       ra.primary_factor, ra.narrative_effect, rs.short_name as poll
FROM RankingAnomaly ra
JOIN TeamRankings tr ON ra.team_ranking_id = tr.team_ranking_id
JOIN PollSchedule ps ON ra.poll_schedule_id = ps.poll_schedule_id
JOIN RankingSystems rs ON ps.ranking_system_id = rs.ranking_system_id
JOIN Teams t ON ra.team_id = t.team_id
WHERE ps.season_id = ?
AND ps.week_id = ?
ORDER BY ABS(ra.deviation) DESC
LIMIT 15;
```

**Compare how different polls weight offensive vs. defensive metrics:**

```sql
SELECT rs.short_name as poll, 
       MAX(CASE WHEN metric_name = 'offensive_rating' THEN estimated_weight ELSE 0 END) as offense_weight,
       MAX(CASE WHEN metric_name = 'defensive_rating' THEN estimated_weight ELSE 0 END) as defense_weight,
       MAX(CASE WHEN metric_name = 'strength_of_schedule' THEN estimated_weight ELSE 0 END) as sos_weight,
       MAX(CASE WHEN metric_name = 'win_pct' THEN estimated_weight ELSE 0 END) as win_pct_weight
FROM PollSystemMetricWeights pw
JOIN RankingSystems rs ON pw.ranking_system_id = rs.ranking_system_id
WHERE pw.season_id = ?
GROUP BY rs.short_name
ORDER BY offense_weight DESC;
``` 